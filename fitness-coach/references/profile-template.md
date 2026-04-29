# Profile template

Use this exact structure when writing `profile.md` (in the storage directory chosen per SKILL.md). Fill every section from the onboarding interview. If the user didn't answer something, write `_not provided_` rather than omitting the section — it makes future updates easier.

```markdown
# Fitness profile — [Name]

_Last updated: YYYY-MM-DD_

## Bio
- **Age:** ...
- **Gender:** ...
- **Height:** ...
- **Weight:** ... (as of YYYY-MM-DD)
- **Fitness level:** ...
- **Estimated max HR:** ... bpm (source: measured / 220-age / Tanaka / etc.)

### HR zones (derived)
- Z1 (50–60%): ... – ... bpm
- Z2 (60–70%): ... – ... bpm
- Z3 (70–80%): ... – ... bpm
- Z4 (80–90%): ... – ... bpm
- Z5 (90–100%): ... – ... bpm

### Injuries / limitations
- ...
- ...

### Medical / safety notes
- **Clinician restrictions:** ...
- **Cardiovascular/metabolic considerations:** ...
- **Medications affecting HR or exercise tolerance:** ...
- **Pregnancy/postpartum considerations:** ...
- **Red flags reported:** ...

## Long-term goals
[Verbatim from user — preserve their bullet structure, baselines, and dates.]

## Workout preferences
- **Typical session length:** ...
- **Variation by day:** ...
- **Enjoys:** ...
- **Dislikes / avoids:** ...
- **Favorite exercises:** ...

## Equipment & locations

### [Location 1, e.g. Basement]
- ...

### [Location 2, e.g. Commercial gym]
- ...

## Current context
- **Recent training:** ...
- **Life context:** ...
- **Time realities:** ...
- **Other:** ...

## Flexible weekly compass
- **Mode:** loose  <!-- "loose" by default; use "fixed plan" only if the user asks for exact weekly scheduling -->
- **Current bias:** ...
- **Weekly targets:** ...
- **Next good options:** ...
- **Avoid for now:** ...

## Skill settings
- **youtube_links:** on  <!-- set to "off" if user asks to stop linking exercise names to YouTube searches -->
- **permission_offer:** unprompted  <!-- "accepted" once persistent allow rules are added, "declined" if user said no, "unprompted" otherwise -->
- **daily_check_in:** off  <!-- off | morning_ping | accountability — only set non-off when scheduling capability exists -->
- **daily_check_in_time:** _none_  <!-- HH:MM local, e.g. "06:27"; "_none_" if daily_check_in is off -->
- **daily_check_in_id:** _none_  <!-- schedule ID returned by the scheduling primitive (CronCreate, OpenClaw, etc.); "_none_" if off -->
- **profile_created:** YYYY-MM-DD
- **last_check_in:** _none_  <!-- YYYY-MM-DD of last completed weekly check-in; "_none_" until the first one runs -->

## Change log
- YYYY-MM-DD — Profile created.
```

## Computing HR zones

If max HR was provided (or estimated from 220 - age), pre-compute the zone ranges in the table above. Round to whole bpm. The user shouldn't have to do the math themselves to use a workout.

If the user later provides a measured max HR, update the bio line AND the zones AND add a change log entry.

## Safety rules

If the user reports chest pain, fainting, severe or unusual shortness of breath, acute injury, neurological symptoms, uncontrolled dizziness, or worsening pain, do not prescribe a workout. Recommend stopping training for now and seeking appropriate medical care or clinician guidance.

Do not diagnose medical issues, prescribe rehab for unresolved injuries, or override clinician restrictions. For non-emergency limitations, work around them and update the profile if they should persist beyond today.

## Updating the profile

- Edit in place — do not rewrite the whole file.
- Bump `_Last updated:_` to today.
- Append a one-line entry to the change log explaining what changed and why.
- Keep the section headings intact even if a section becomes empty.
