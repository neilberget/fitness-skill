---
name: fitness-coach
description: Personal fitness coach that designs workouts tailored to the user's bio, goals, equipment, preferences, and recent training history. Use whenever the user asks for a workout, workout idea, exercise suggestion, training session, daily lift/cardio/mobility plan, modifications to a proposed workout, or wants to log/report what they did. Also use when the user asks to update their fitness profile, change their goals, add equipment, or report new injuries. On first use, this skill onboards the user by interviewing them and saving a structured profile; on subsequent uses, it loads the profile and recent workout log to inform suggestions.
---

# Fitness Coach

Acts as the user's personal trainer. Maintains a long-lived profile (bio, goals, equipment, preferences) and an append-only workout log under `~/.claude/fitness-coach/`. Uses both to design workouts that fit the user's history, recovery state, and goals.

## Files this skill owns

All state lives outside the skill bundle, in the user's home directory.

**Storage location (in priority order):**
1. `$FITNESS_COACH_HOME` if the env var is set
2. `~/.fitness-coach/` if it exists
3. `~/.claude/fitness-coach/` otherwise (and create it if missing)

This lets the same skill run cleanly under Claude Code, Codex CLI, or anything else without fighting over directory namespaces. Pick the path once at the start of the session and use it consistently.

Files in that directory:

- `profile.md` — bio, goals, equipment, preferences. Long-lived. Edit in place when facts change.
- `workout-log.md` — append-only reverse-chronological log of completed sessions (most recent at top).

Create the directory if missing: `mkdir -p <chosen-path>`.

## Workflow decision tree

Read files first, then branch:

1. Check whether `~/.claude/fitness-coach/profile.md` exists.
   - **Missing or empty** → run **Onboarding** (below). Do not skip.
   - **Exists** → read it. Also read `workout-log.md` if it exists (load the most recent ~10 sessions; if the file is large, read the last 200 lines). If `Skill settings → permission_offer` is `unprompted`, briefly offer the persistent allow rule (see [Offer to skip future permission prompts](#offer-to-skip-future-permission-prompts-claude-code-only)) at the end of your response, then update the setting to `accepted` or `declined` based on the answer. Don't re-ask once it's `declined`.

2. Then route by what the user asked for:
   - "Suggest a workout" / "what should I do today" / "give me an idea" → **Design a workout**
   - "Make it shorter" / "swap X" / "no jumping today" → **Modify the proposed workout**
   - "Here's how it went" / "I did X" / "log this" → **Log the session**
   - "Update my profile" / new injury / new equipment / goal change → **Update the profile**

If the user's intent is ambiguous (e.g. "let's train"), default to Design a workout but confirm time available and any constraints today before locking the plan.

## Onboarding (first use only)

Goal: produce `profile.md` from a short interview. Do **not** dump all questions at once — ask in 5 grouped rounds, waiting for the user's answer before moving to the next round. This keeps the conversation natural and lets earlier answers inform later questions.

Before starting, say briefly: "I'll ask a few questions to build your fitness profile. This is a one-time setup — after this, I'll just suggest workouts."

### Round 1 — Bio
Ask for, in one message:
- Name (if you already know it from context, confirm rather than re-ask)
- Age
- Gender
- Height
- Weight
- General fitness level (sedentary / recreationally active / regularly trains / competitive athlete — or their own words)
- Injuries, surgeries, or movement limitations to work around
- Estimated max heart rate, if known (otherwise note "use 220 - age as default")

### Round 2 — Long-term goals
Ask the user to list their long-term goals, **with the explicit instruction to be specific** — measurable targets, baselines, and timeframes where possible. Show this example so they understand the depth wanted:

```
• Reach 97.5th percentile VO2max for age/gender.
  - Last professional test (Aug 2023): ~50 (about 90th percentile)
  - Last Cooper Test estimate (May 2025): ~50–51 (likely lower right now)
• Reach 75th percentile muscle mass (ALMI).
  - Last measured (Aug 2023): ~50th percentile
• Maintain mobility, agility, and balance
• Build broad capabilities: endurance, strength, mobility, balance, stability, high energy, power
• Gain weight/lean muscle mass toward ideal body weight of 180 lbs
• Appearance is secondary, but goal is to look strong, capable, and well cared for
• Build and maintain a healthy heart, body, and mind for a long, healthy, capable life
```

### Round 3 — Workout preferences
Ask:
- Typical time available per session (and any variation by day of week)
- Workout styles enjoyed (e.g. heavy lifting, zone 2, intervals, kettlebell flows, hiking, yoga)
- Styles disliked or to avoid
- Favorite exercises or movements

### Round 4 — Equipment & location
Ask for:
- Primary workout location(s) (basement, garage gym, commercial gym, outdoors, hotel room, etc.)
- Detailed equipment list — encourage specificity (e.g. "adjustable dumbbells 5–50 lb", "Concept2 rower", "pull-up bar", "trap bar 45 lb + plates to 315 lb"). Include what's available at each location if multiple.

### Round 5 — Current context
Open-ended. Ask for any other relevant info:
- Recent training context (what they've been doing the last few weeks)
- Life context (sleep, stress, travel, kids, work load)
- Time realities (specific recurring conflicts)
- Anything else worth knowing

### Save the profile

After the interview, write `~/.claude/fitness-coach/profile.md` using the structure in [references/profile-template.md](references/profile-template.md). Then show the user the saved profile and ask "Anything to fix before we move on?" — apply edits before suggesting a workout.

### Offer to skip future permission prompts (Claude Code only)

If running under Claude Code, every session will otherwise prompt the user to approve each Read/Write/Edit against the fitness directory. After the profile is saved, offer to fix this once:

> "Claude Code will prompt you to approve every read and write to your fitness profile. Want me to add a permanent allow rule to `~/.claude/settings.json` so it stops asking? You can revoke it any time with `/permissions`."

If the user agrees, edit `~/.claude/settings.json` to add the following entries to `permissions.allow` (create the file or the `permissions.allow` array if missing; do not overwrite existing entries). Use the absolute path that matches the storage directory chosen at the start of the session — the matcher does not expand `~` reliably across versions, and it does not expand env vars at all, so write the resolved path:

```json
{
  "permissions": {
    "allow": [
      "Read(/Users/<you>/.claude/fitness-coach/**)",
      "Write(/Users/<you>/.claude/fitness-coach/**)",
      "Edit(/Users/<you>/.claude/fitness-coach/**)"
    ]
  }
}
```

Resolve `/Users/<you>` from `$HOME` before writing. If the storage directory is `~/.fitness-coach/` or a custom `$FITNESS_COACH_HOME`, use that resolved path instead. Confirm in one line what you added ("Added persistent allow rules for `<path>` to `~/.claude/settings.json`."), then set `permission_offer: accepted` in the profile's Skill settings.

If the user declines, set `permission_offer: declined` in the profile and don't ask again. If the harness is not Claude Code (e.g. Codex CLI, which has no equivalent persistent-allow mechanism), skip this step silently and leave the setting as `unprompted`.

## Design a workout

Before designing, you should already have read `profile.md` and recent `workout-log.md` entries. If you haven't, do so now.

Ask one targeted question if needed (skip if obvious from context):
- How much time today?
- How are you feeling / any soreness or sleep issues?
- Any constraint today (location change, equipment unavailable)?

Then design the session. Use the heuristics in [references/programming-principles.md](references/programming-principles.md) — read it before designing if you haven't this session. Key things to weight:

- **Goals**: what energy systems / capacities does the user need to push? Tilt toward whatever the long-term goals demand.
- **Recent log**: don't repeat heavy lower-body two days in a row. Rotate movement patterns. If the last few sessions skewed one way, balance it.
- **Recovery signals**: high RPE / poor sleep / soreness reported recently → bias toward zone 2, mobility, or technique work.
- **Equipment & location** today.
- **Time available** today.

Output the workout in this shape (concise — no fluff):

```
## Today's workout — [date], ~[duration]

**Focus:** [one line — e.g., "lower body strength + zone 2 finisher"]
**Why this:** [1–2 sentences tying it to goals + recent log]

### Warm-up (~X min)
- ...

### Main work (~X min)
- [Exercise](https://www.youtube.com/results?search_query=Exercise+form) — sets × reps @ load/intensity, rest
- ...

### Finisher / cooldown (~X min)
- ...

**Target HR zones:** [if cardio is involved]
**Notes:** [form cues, alternatives if something feels off]
```

**Exercise name links (default on):** By default, wrap every exercise name in a YouTube search link so the user can click through if they don't recognize the movement. Format: `[Exercise name](https://www.youtube.com/results?search_query=Exercise+name+form)` — URL-encode spaces as `+` and append `+form` to bias results toward demonstrations. Apply this in warm-up, main work, and finisher sections. Skip it for plain rest/recovery items (e.g. "rest 90s") and section labels.

If the user asks to turn the links off ("stop linking exercises", "no YouTube links", etc.), record it in `profile.md` under `## Skill settings` as `youtube_links: off` and stop adding links in this and future sessions. If they later ask to turn it back on, set it to `on`. Always check the profile for this setting before generating a workout — default to on if missing.

Then ask: "Want any modifications, or ready to go?"

## Modify the proposed workout

Apply the user's requested changes, keeping the same output shape. Don't re-explain the parts that didn't change — just present the updated workout. If a change conflicts with the user's stated goals (e.g. cutting all the strength work on a strength-focused day), call it out in one sentence, then comply.

## Log the session

When the user reports back, append a new entry to the **top** of `~/.claude/fitness-coach/workout-log.md` (most recent first). Use the entry format in [references/log-format.md](references/log-format.md). Capture:

- Date
- What was actually done (modifications from the proposed workout, if any)
- Loads / times / distances / HR if reported
- RPE or how it felt
- Anything notable (pain, energy, PRs, surprises)

If the file doesn't exist, create it with a top-of-file header `# Workout log` and add the entry below.

After logging, briefly acknowledge ("Logged.") and either stop or — if the user signaled they want to keep going — propose what to do next.

## Update the profile

When the user shares info that changes the profile (new injury, lost/gained equipment, shifted goals, changed weight, hit a goal), edit `profile.md` in place. Keep the structure intact. Confirm what you changed in one sentence. Don't rewrite sections that didn't change.

## Conventions

- Use the user's name once at the start of a session, not in every message.
- Don't moralize or push. The user is in charge of their training; you're the planner.
- Be specific with loads, reps, times, distances. Avoid "moderate weight" or "a few rounds."
- When suggesting HR zones, use the user's stored max HR (or 220 - age) and give the actual BPM range, not just "Zone 2."
- Never invent log entries. Only log what the user reports.
