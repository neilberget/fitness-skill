# fitness-coach

A [Claude Code](https://claude.com/claude-code) skill that turns Claude into your personal fitness coach. It onboards you once, then designs tailored workouts on demand using your profile, goals, equipment, and recent training history.

## What it does

- **One-time onboarding.** First time you ask for a workout, Claude interviews you in 5 short rounds: bio, long-term goals, preferences, equipment & location, and current life context.
- **Persistent profile.** Saves your goals, equipment, preferences, safety notes, flexible weekly targets, and derived heart-rate zones.
- **Tailored workouts on demand.** Ask "give me a workout" any time and Claude designs a session that fits your goals, available time today, current equipment, recovery state, and what you've done recently.
- **Safety boundaries.** Captures injuries, clinician restrictions, and red flags during onboarding, then avoids prescribing workouts when medical guidance is the right next step.
- **Modifications mid-conversation.** "Make it 30 minutes." "Swap out the deadlifts." "No jumping today." It adjusts.
- **Workout log.** Report back how it went and Claude appends a structured entry to `workout-log.md` in the chosen storage directory (newest first). Future sessions read recent entries to inform programming.
- **Flexible weekly compass.** Instead of forcing a rigid schedule, it keeps broad weekly targets and "next good options" so training still works around busy weeks. If you want exact day-by-day planning, it can do that too.
- **Weekly check-in.** After ~a week of training (7+ days, 3+ sessions), Claude offers a 5-minute retrospective interview — what worked, what to drop, goal progress, life context shifts — then updates your profile, flexible weekly compass, and `check-ins.md`. The suggestion is always optional; if you just want today's workout, say so and it skips.
- **Profile updates.** Tell it about a new injury, new equipment, or a goal change and it edits the profile in place.

Every workout includes specific loads, rep schemes, time blocks, and BPM ranges — no "moderate weight, a few rounds" hand-waving.

## Example session

```
You: give me a workout for today, I have 45 minutes

Claude: How are you feeling — any soreness or sleep issues today?

You: feeling good, slept 8 hours

Claude:
## Today's workout — 2026-04-24, ~45 min

**Focus:** Lower body strength + zone 2 finisher
**Why this:** Your last two sessions were upper-pull and a Z2 ride;
lower hasn't been hit in 4 days. VO2max goal still wants regular Z2 dose.

### Warm-up (~6 min)
- 3 min easy bike or rower
- World's greatest stretch × 5/side
- Goblet squat × 8 @ 35 lb
- Single-leg RDL × 5/side bodyweight

### Main work (~28 min)
- Trap bar deadlift — 4×5 @ 285 lb (last session 275×5×4 RPE 7), rest 2:30
- Bulgarian split squat — 3×8/leg @ 35 lb DB each hand, rest 90s
- Hanging knee raise — 3×10, rest 60s

### Finisher (~8 min)
- Zone 2 row: 8 min @ 118–132 bpm

**Notes:** If trap bar feels heavy on warm-up, stay at 275.
```

## Installation

The skill is built as a Claude Code skill but works in OpenAI Codex CLI natively (same `SKILL.md` + `references/` format) and can be adapted for ChatGPT with caveats.

### Claude Code

Clone into your user skills directory:

```bash
mkdir -p ~/.claude/skills
git clone https://github.com/neilberget/fitness-skill.git /tmp/fitness-skill
mv /tmp/fitness-skill/fitness-coach ~/.claude/skills/
rm -rf /tmp/fitness-skill
```

Or install the packaged `.skill` file directly:

```bash
# In Claude Code:
/skill install /path/to/fitness-coach.skill
```

State is saved to `~/.claude/fitness-coach/` by default in Claude Code, unless `$FITNESS_COACH_HOME` is set or `~/.fitness-coach/` already exists.

### OpenAI Codex CLI

Codex uses the same skill format (`SKILL.md` with YAML frontmatter, `references/` subdirectory) and loads skills from `~/.agents/skills/`:

```bash
mkdir -p ~/.agents/skills
git clone https://github.com/neilberget/fitness-skill.git /tmp/fitness-skill
mv /tmp/fitness-skill/fitness-coach ~/.agents/skills/
rm -rf /tmp/fitness-skill
```

Set the storage path so it doesn't live under Claude's namespace:

```bash
export FITNESS_COACH_HOME="$HOME/.fitness-coach"
```

(Add that to your shell rc file to make it permanent.) The skill checks `$FITNESS_COACH_HOME` first, then `~/.fitness-coach/` if it already exists, then `~/.claude/fitness-coach/` in Claude Code or `~/.fitness-coach/` elsewhere.

If Codex's frontmatter validator complains about the SKILL.md, the [`claude-to-codex`](https://community.openai.com/t/claude-to-codex-bring-claude-skills-to-codex-automatically/1378574) community tool handles the rewrite automatically.

### ChatGPT (Custom GPT) — works with caveats

ChatGPT has no filesystem, so the workout log can't auto-persist. You can still use the skill's logic by creating a Custom GPT:

1. Create a new Custom GPT (chatgpt.com → Explore GPTs → Create).
2. Paste the contents of [`fitness-coach/SKILL.md`](fitness-coach/SKILL.md) (everything below the YAML frontmatter) into the **Instructions** field.
3. Upload the four files in [`fitness-coach/references/`](fitness-coach/references/) as **Knowledge** files.
4. Add this to the end of the Instructions: *"There is no filesystem in this environment. When the user asks to save the profile or log a workout, output the updated file contents in a code block and ask the user to copy it into their own `profile.md` / `workout-log.md`. When the user wants to load context, ask them to paste those files."*

**What this gets you:** the same coaching logic and onboarding flow.
**What it doesn't:** automatic profile/log persistence — you'll be copy-pasting markdown blocks each session. If you want true persistence, the ChatGPT Atlas desktop app with a filesystem MCP server can read/write local files; setup is involved but the skill itself works unchanged once the MCP layer is in place.

### Other agents

The same `SKILL.md` + `references/` structure works (sometimes with light frontmatter tweaks) in Cursor, Aider, Windsurf, Gemini CLI, and other tools that have adopted Anthropic's Skill format. Point them at the `fitness-coach/` directory.

## Verify it's installed

Open Claude Code and ask:

> Suggest a workout for today

If installed, Claude will recognize it has no profile yet and start the onboarding interview. After that, it will go straight to designing workouts.

## What gets saved on your machine

The skill writes only a small set of files to a single directory. Location is chosen in this order:

1. `$FITNESS_COACH_HOME` if set
2. `~/.fitness-coach/` if it already exists
3. `~/.claude/fitness-coach/` when running under Claude Code
4. `~/.fitness-coach/` otherwise

Files:

- `profile.md` — your bio, goals, equipment, preferences. Edited in place when facts change.
- `workout-log.md` — append-only, newest first. One entry per session.
- `check-ins.md` — append-only, newest first. One entry per weekly check-in (created on the first one).

Both are plain markdown. You can edit them by hand any time. Nothing is sent anywhere except as part of your normal agent conversations.

## Customization

The skill's behavior is defined in `fitness-coach/SKILL.md` and the files under `fitness-coach/references/`:

- `references/profile-template.md` — structure of the saved profile
- `references/log-format.md` — structure of workout log entries
- `references/check-in-format.md` — structure of weekly check-in entries
- `references/programming-principles.md` — heuristics for designing sessions (goal→dose mapping, recovery rules, loading progression, time budgeting)

Edit these to taste. If you change the programming principles, Claude's workout design changes accordingly.

## Repository layout

```
fitness-coach/
├── SKILL.md                    # Skill entry point: workflow + decision tree
└── references/
    ├── profile-template.md     # Saved-profile structure
    ├── log-format.md           # Workout log entry format
    ├── check-in-format.md      # Weekly check-in entry format
    └── programming-principles.md  # Workout design heuristics
fitness-coach.skill             # Packaged distributable
```

## Contributing

Issues and PRs welcome. If you've found heuristics that consistently produce better workouts for you, the most useful place to send them is `references/programming-principles.md`.

## License

MIT
