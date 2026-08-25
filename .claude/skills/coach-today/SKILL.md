---
name: coach-today
description: Short daily check-in (any morning). Confirms or adjusts ONLY today's session for weather, recovery, and yesterday's actuals; can slide the long run on request. Use when the user asks what's today, says how they slept/feel, or wants to move today's session.
---

# Coach Today

Read `.claude/skills/shared/coach-rules.md` (at minimum: invariants, heat
protocol, ITB section), then `athlete.private.md` (weather coordinates,
weight for kcal targets) and `plan/week-current.md`. No week file →
"run /coach-plan-week first". Missing athlete.private.md → stop: "copy
athlete.private.example.md to athlete.private.md and fill it in". Keep the
whole interaction to ONE screen of output — no ceremony.

## 1. Quick inputs

- Yesterday's actual: `querySportRecords` (yesterday only).
- Recovery: `queryRecoveryStatus` and/or `querySleepData` (last night).
- Today's hourly forecast (coach-rules curl, hourly view).
- One question max to the user, and only if needed ("any ITB signals /
  schedule changes today?"). Log any symptom to log/itb.md.

## 2. Decide today

- Weather: apply heat protocol (start-time shift, by-HR conversion,
  venue change with FG access budgeted).
- Poor recovery or yesterday heavier than planned → downgrade intensity,
  never add.
- User slides the long run (Fri/Sat/Sun/Mon) → move it, rebalance adjacent
  days so no hard-day adjacency, re-check invariants.
- COROS down → proceed from the plan file + what the user says.

## 3. Output

One compact block: today's session (structure, venue, start time), kcal
target, stretching variant, one-line reason for any change. For sessions
> 75 min or hot days: one carry line — fluid volume (≥2 L / named refill on
FG mornings) + fuel items and count from the coach-rules inventory.

## 4. Persist

Write the adjustment into plan/week-current.md (today's section; adjacent
days only if the long run moved), bump `updated:` in the header, then
`git add -A && git commit -m "coach: adjust YYYY-MM-DD"`.
If nothing changed, say so and skip the commit.
