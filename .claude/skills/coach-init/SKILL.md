---
name: coach-init
description: One-time (or major-reset) setup of the training macrocycle to the summer-2027 100-miler. Use when plan/ is empty, when the user asks to rebuild the plan from scratch, or after a season-changing event (injury escalation, race change). Reads 12+ months of COROS history, interviews the user, writes macro/meso/week plans.
---

# Coach Init

Read `.claude/skills/shared/coach-rules.md` in full, then `athlete.md`,
`athlete.private.md`, `reference/strength.md`, `reference/routes.md`,
`log/itb.md`. If plan/ already
has a macro.md, warn that this is a reset and get explicit confirmation
before overwriting anything.

## 1. Gather history (COROS MCP)

- `querySportRecords` for the last 12 months (batches: use startDate/endDate
  windows, sportTypeCodes [65535], limit 100 per window).
- `queryTrainingLoadAssessment` for the last 28 days.
- Find the 2025 Divca result (late September 2025, ~28 km trail effort) and
  record its date, time, distance as the benchmark in macro.md.
- Do NOT call queryFitnessAssessmentOverview (distrusted).
- Compute from history: typical weekly hours, run km, ride km, long-run size,
  descent exposure (getActivityDetail on the biggest recent trail runs), and
  the "big day" pattern (trip weekends).

## 2. Interview the user (one question at a time)

Only what data cannot show: ITB history details worth logging as baseline;
confirmation that reference/strength.md stage 0 is still exactly current;
any upcoming trips/absences in the next 8 weeks; preferred long-run day
by default; gut tolerance — what was actually consumed per hour in past
ultras (cross-check the answer against the fuel inventory in coach-rules
and record personal intake reality, not generic targets).

If `athlete.private.md` is missing, also ask for weight, height, birth year,
sex and home city, then write it from `athlete.private.example.md` (it is
gitignored — never put these values in any tracked file).

## 3. Draft the macrocycle

Backwards from anchors (see coach-rules.md periodization). Each phase's
description must state its role in the fueling-progression arc (when intake
builds toward race rates) and respect the recovery→specific symptom-free
gate. Present the phase plan IN CHAT for approval before writing any file.
Iterate until approved.

## 4. Write files

After approval: write `plan/macro.md`, the first `plan/meso-current.md`, and
the first `plan/week-current.md` in the exact formats from coach-rules.md.
Run the safety-invariant checklist on the week before presenting it. Present
the week in chat for final edits.

## 5. Validate against ground truth

Before finishing, check and state explicitly:
- Week 1 resembles a plausible next week for THIS athlete's recent COROS
  weeks (not a generic template).
- planned_hours ≤ 9; race dates in macro match athlete.md.
- No tracked file contains body metrics, age, sex or the city — those live
  only in `athlete.private.md`.
- The 2025 Divca benchmark is recorded in macro.md.

## 6. Commit

`git add -A && git commit -m "coach: init macrocycle to summer 2027"`

## Degraded mode

COROS unavailable → interview the user for a typical-week summary and last
month's pattern; mark macro.md header with `data: manual`; continue.
