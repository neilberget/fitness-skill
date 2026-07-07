# Onboarding (first use only)

Run this when `profile.md` is missing from the storage directory. Goal: produce `profile.md` from a short interview. Do **not** dump all questions at once — ask in 5 grouped rounds, waiting for the user's answer before moving to the next round. This keeps the conversation natural and lets earlier answers inform later questions.

Before starting, say briefly: "I'll ask a few questions to build your fitness profile. This is a one-time setup — after this, I'll just suggest workouts."

## Round 1 — Bio and safety

Ask for, in one message:
- Name (if you already know it from context, confirm rather than re-ask)
- Age
- Gender
- Height
- Weight
- General fitness level (sedentary / recreationally active / regularly trains / competitive athlete — or their own words)
- Injuries, surgeries, or movement limitations to work around
- Any clinician restrictions or medical considerations that should affect training (e.g. cardiovascular/metabolic conditions, blood-pressure issues, pregnancy/postpartum, medications that affect heart rate, dizziness/fainting history, chest pain with exertion)
- Estimated max heart rate, if known (otherwise note "use 220 - age as default")

## Round 2 — Long-term goals

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

## Round 3 — Workout preferences

Ask:
- Typical time available per session (and any variation by day of week)
- Workout styles enjoyed (e.g. heavy lifting, zone 2, intervals, kettlebell flows, hiking, yoga)
- Styles disliked or to avoid
- Favorite exercises or movements

## Round 4 — Equipment & location

Ask for:
- Primary workout location(s) (basement, garage gym, commercial gym, outdoors, hotel room, etc.)
- Detailed equipment list — encourage specificity (e.g. "adjustable dumbbells 5–50 lb", "Concept2 rower", "pull-up bar", "trap bar 45 lb + plates to 315 lb"). Include what's available at each location if multiple.

## Round 5 — Current context

Open-ended. Ask for any other relevant info:
- Recent training context (what they've been doing the last few weeks)
- Life context (sleep, stress, travel, kids, work load)
- Time realities (specific recurring conflicts)
- Anything else worth knowing

## Round 6 — Daily scheduled check-in (only if scheduling is available)

Skip this round entirely unless the runtime can schedule a recurring prompt (see SKILL.md § Detecting scheduling capability). If it can't, leave `daily_check_in: off` in the profile and move on — do not promise something the environment can't deliver.

If scheduling is available, offer it once during onboarding:

> "I noticed this environment can run me on a schedule (cron / scheduled agents). Want me to ping you each morning? Three options:
> - **Off** — only when you ask.
> - **Morning ping** — 1–2 lines: today's suggested focus and why, based on your recent log. You can ask for the full workout when ready.
> - **Accountability check-in** — short interview each morning (sleep, water, soreness, energy, time available today), then a tailored 1–2 line suggestion. Best if you want the program to react to recovery state.
>
> If yes, what local time? (Default: 6:30am.)"

Save their answer. If they pick `morning_ping` or `accountability`, schedule it (see SKILL.md § Manage the daily scheduled check-in) and store the resulting schedule ID, the time, and the mode in the profile under `## Skill settings`. If they pick off — or scheduling can't be set up cleanly — set `daily_check_in: off` and move on without scheduling anything.

## Save the profile

After the interview, Write `profile.md` to the storage directory using the structure in [profile-template.md](profile-template.md). Set `profile_created` to today's date and initialize `## Flexible weekly compass` from the user's goals, preferences, equipment, and time realities. Use the **absolute path** resolved during bootstrap so future Read/Write calls match any persistent allow rule. Then show the user the saved profile and ask "Anything to fix before we move on?" — apply edits before suggesting a workout.

## Offer to skip future permission prompts (Claude Code only)

If running under Claude Code, every session will otherwise prompt the user to approve each Read/Write/Edit against the fitness directory. After the profile is saved, offer to fix this once. (This same offer is also made at the start of any later session while `permission_offer` is still `unprompted` — see SKILL.md's bootstrap steps.)

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
