# Workout log format

`~/.claude/fitness-coach/workout-log.md` is append-only and **reverse chronological** (newest entry directly under the `# Workout log` header). This lets future sessions read just the top of the file to get recent context.

## File header (only on first creation)

```markdown
# Workout log

_Newest first. Append new entries directly below this header._
```

## Entry format

```markdown
## YYYY-MM-DD — [short title, e.g. "Lower body strength + Z2"]

**Duration:** ~X min
**Followed plan:** yes / partially / no — [one line on what changed if not "yes"]
**RPE / felt:** [overall RPE 1–10 + one-line subjective note]

### What was done
- Exercise — actual sets × reps @ load (e.g., "Trap bar deadlift — 4×5 @ 275 lb")
- Cardio — actual time / distance / avg HR / avg pace
- ...

### Notes
- [Pain, energy, PRs, surprises, what to carry into next session]

---
```

The trailing `---` separates entries visually.

## What to capture

Pull from what the user reported. Don't pad. If they only said "did the workout, felt good, 45 min," that's the entry.

Always capture **load and reps** for strength work, and **time + intensity (HR or pace)** for cardio, when reported. These are what future workout design needs to progress the user.

If the user reports pain or a new limitation, ALSO update `profile.md` injuries section — don't only bury it in the log.

## What NOT to do

- Don't fabricate numbers the user didn't give.
- Don't overwrite or edit prior entries — only append new ones at the top.
- Don't rewrite the whole file when adding an entry. Read the top, prepend the new entry below the header, leave the rest untouched.

## Reading the log for context

When designing a new workout, read the top ~10 entries (or last 200 lines, whichever is shorter). Look for:

- **Movement balance** — what muscle groups / patterns have been hit recently?
- **Cardio dose** — how much zone 2 vs intervals vs none in the last 7–10 days?
- **Progression** — are loads going up on the main lifts? If stuck, plan to deload or vary.
- **Recovery flags** — repeated high RPE, poor sleep notes, new pain → bias today's session lighter.
