# fitness-coach

A [Claude Code](https://claude.com/claude-code) skill that turns Claude into your personal fitness coach. It onboards you once, then designs tailored workouts on demand using your profile, goals, equipment, and recent training history.

## What it does

- **One-time onboarding.** First time you ask for a workout, Claude interviews you in 5 short rounds: bio, long-term goals, preferences, equipment & location, and current life context.
- **Persistent profile.** Saves everything to `~/.claude/fitness-coach/profile.md`, including derived heart-rate zones.
- **Tailored workouts on demand.** Ask "give me a workout" any time and Claude designs a session that fits your goals, available time today, current equipment, recovery state, and what you've done recently.
- **Modifications mid-conversation.** "Make it 30 minutes." "Swap out the deadlifts." "No jumping today." It adjusts.
- **Workout log.** Report back how it went and Claude appends a structured entry to `~/.claude/fitness-coach/workout-log.md` (newest first). Future sessions read recent entries to inform programming.
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

This skill works with Claude Code (CLI, desktop app, or IDE extensions).

### Option 1: Clone into your skills directory

```bash
mkdir -p ~/.claude/skills
cd ~/.claude/skills
git clone https://github.com/neilberget/fitness-skill.git
mv fitness-skill/fitness-coach .
rm -rf fitness-skill
```

### Option 2: Install from the packaged `.skill` file

Download `fitness-coach.skill` from the [latest release](https://github.com/neilberget/fitness-skill/releases) (or from the repo root) and install it via Claude Code's skill installation flow:

```bash
# In Claude Code:
/skill install /path/to/fitness-coach.skill
```

### Option 3: Use as a plugin

If you manage Claude skills via a plugin system, point it at this repo's `fitness-coach/` directory.

## Verify it's installed

Open Claude Code and ask:

> Suggest a workout for today

If installed, Claude will recognize it has no profile yet and start the onboarding interview. After that, it will go straight to designing workouts.

## What gets saved on your machine

The skill writes only to `~/.claude/fitness-coach/`:

- `profile.md` — your bio, goals, equipment, preferences. Edited in place when facts change.
- `workout-log.md` — append-only, newest first. One entry per session.

Both are plain markdown. You can edit them by hand any time. Nothing is sent anywhere except as part of your normal Claude conversations.

## Customization

The skill's behavior is defined in `fitness-coach/SKILL.md` and the files under `fitness-coach/references/`:

- `references/profile-template.md` — structure of the saved profile
- `references/log-format.md` — structure of workout log entries
- `references/programming-principles.md` — heuristics for designing sessions (goal→dose mapping, recovery rules, loading progression, time budgeting)

Edit these to taste. If you change the programming principles, Claude's workout design changes accordingly.

## Repository layout

```
fitness-coach/
├── SKILL.md                    # Skill entry point: workflow + decision tree
└── references/
    ├── profile-template.md     # Saved-profile structure
    ├── log-format.md           # Workout log entry format
    └── programming-principles.md  # Workout design heuristics
fitness-coach.skill             # Packaged distributable
```

## Contributing

Issues and PRs welcome. If you've found heuristics that consistently produce better workouts for you, the most useful place to send them is `references/programming-principles.md`.

## License

MIT
