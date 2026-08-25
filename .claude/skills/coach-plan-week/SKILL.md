---
name: coach-plan-week
description: Weekly planning session (typically Sunday evening). Reviews the completed week against COROS actuals, advances the mesocycle, writes next week's plan with sessions, calories, and stretching. Use when the user asks to plan the week, review the week, or says the week is over.
---

# Coach Plan Week

Read `.claude/skills/shared/coach-rules.md` in full, then `athlete.md`,
`athlete.private.md`, `plan/macro.md`, `plan/meso-current.md`, `plan/week-current.md`,
`log/itb.md`. Missing athlete.md → stop: "run /coach-init first". Missing
athlete.private.md → stop: "copy athlete.private.example.md to
athlete.private.md and fill it in".
Missing/corrupt week file → regenerate context from meso-current.md and say so.

## 1. Collect actuals

- `querySportRecords` for the completed week (all sports).
- `queryTrainingLoadAssessment` (7 days) for load-ratio commentary.
- `getActivityDetail` on runs that were long or trail-flagged (>20 km or
  >200 m gain) to sum descent meters.
- Ask the user: anything unrecorded (strength sessions done? ITB symptoms?
  long-sitting days? how did the long run feel?). Append symptoms to
  log/itb.md.
- COROS down → ask for a 3-line summary, mark review `data: manual`.

## 2. Write the review

`log/weeks/YYYY-Www.md` in the coach-rules format: planned vs actual per day,
load note, flags (missed sessions, symptom events, heat disruptions).
Evaluate the load ratio against the 1.3 action threshold explicitly and say
which side of it the week landed on.

## 3. Advance the mesocycle

- Meso finished → author the next one from macro.md (present block focus in
  chat first).
- Symptom escalation (per coach-rules ITB section) → rehab-mode meso, flag
  prominently.
- Down-week due → schedule it regardless of momentum. Load ratio > 1.3 →
  next build auto-converts to a down week (invariant 7).
- Authoring a taper for a goal race ≥ 28 km → include the written
  race-execution rule (pacing cap, descent strategy, numbered fueling plan).
- Entering a specific block straight after a recovery phase → check the
  ≥ 2 consecutive symptom-free weeks gate; if unmet, extend recovery.
- Update meso-current.md header/body if anything changed.

## 4. Draft next week

- Fetch the 7-day forecast (coach-rules curl, daily view).
- Build the week within the skeleton: Thursday ride fixed; strength 2×;
  long run on the user's preferred day (Fri/Sat/Sun/Mon — ask if unclear);
  weekday sessions ≤ 1 h; FG sessions budget the access ride; heat rules
  applied (hard sessions on cooler days, ≥30 °C → time shift or by-HR).
- Apply the soft-adjacency rule if a quality ride precedes the long run.
- Fueling: `Fueling:` lines per coach-rules (items from the inventory +
  cadence + fluid) on every session > 75 min; protein note on strength and
  long-run days.
- Calories per coach-rules formula, per day. Stretching A default, B on
  known long-sitting days.
- Run ALL safety invariants; fix violations before presenting.

## 5. Present and finalize

Show the week in chat (compact table + notes). Apply user edits (e.g. "long
run Friday"), re-check invariants after edits, then write
plan/week-current.md.

## 6. Commit

`git add -A && git commit -m "coach: plan week YYYY-Www"`
