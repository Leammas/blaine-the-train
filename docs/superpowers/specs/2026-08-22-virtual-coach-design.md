# Virtual Trainer/Coach — Design Spec

Date: 2026-08-22
Status: approved design, pending implementation plan

## What this is

A virtual running/ultra-trail coach that lives in this repo as Claude Code skills.
It reads live training data from the COROS MCP server, plans and adapts training
week by week, and keeps all state in human-editable files under git.

## The athlete

- Body metrics, age and sex live in `athlete.private.md` (gitignored). COROS profile
  vs. user-stated values differ slightly — watch data wins
  unless corrected in `athlete.md`).
- 10+ years of training; UTMB-index runner. The COROS fitness assessment
  (VO2max, threshold pace, race predictions) is considered unreliable right now
  and is not used as an input — fitness is judged from actual recent sessions.
- Current volume ~7–10 h/week: easy road runs 5–14 km around the home city, trail long
  runs on Fruška Gora and trips (Kolašin, Durmitor), regular gravel-bike rides,
  hikes. One 61 km / 7:48 mountain effort on 2026-07-04.
- This season was reduced by an IT band (ITB) injury. No pain on easy runs now,
  but symptoms still appear after long sitting sessions. Last 3 months of strength
  work were ITB-focused.
- All training except strength is recorded in COROS. Interpretation rule: any run
  over 20 km or over 200 m elevation gain was a trail run even if recorded as
  "Outdoor Run".
- Bike is a gravel bike.

## Goals

1. **Primary target**: a 100-mile ultra trail with 9,000 m+ elevation gain,
   summer 2027. The specific race is not yet chosen — the plan targets the
   profile (distance, vert, season), not a fixed date, and firms up once the
   race is picked.
2. **Divca Trail, 2026-09-28**: 28 km / 1,453 m+. Goal: perform not worse than
   last year (2025 result is in COROS history and serves as the benchmark).
3. **Fruška Gora Ultra Trail, 2027-04-28**: 130 km / 6,000 m+.

## Hard constraints

- **≤ 9 training hours per week**, including strength training.
- **Weekday morning sessions ≤ 1 h** (must be home by 07:00). Two sessions a day
  are allowed when a day needs more volume.
- **Thursday social ride is fixed**: ~15 km flat, 100 m + 400 m climb at ~10 %
  average, ~20 km back with small hills and a downhill section. It counts as
  quality bike work; the plan schedules around it.
- **Long-session flexibility**: the weekly long run has a preferred day but
  Friday, Saturday, Sunday, and Monday are all valid slots. The daily check-in
  may slide it on request, rebalancing adjacent days (never adjacent to another
  hard run day).
- **Weekends are uncapped**: the 1 h limit is weekday-mornings only; Sat/Sun can
  host 2–4 h+ sessions.

## Terrain and environment

- Home city: mainly paved flat roads; a ~10 km riverbank path with a soft gummy
  surface (**no shade**); Petrovaradin fortress with a ~40 m stairway climb and
  cobblestone road options.
- **Fruška Gora access cost**: reaching the main trail terrain requires a ~10 km
  ride with ~350 m of climbing one way (the return is easy). Every Fruška Gora
  trail session must budget this commute into the day's hours and training load —
  it is treated as warm-up plus aerobic work, not as free time.
- Summer heat: mornings can exceed 30 °C; every plan must be weather-adjustable.

## Product decisions (from brainstorming)

| Question | Decision |
|---|---|
| Product form | Claude Code skills in this repo; state in files; no app/UI |
| Cadence | Weekly planning session + optional short daily check-in |
| Weather | Coach fetches the home-city forecast automatically from Open-Meteo (free, no API key), coordinates from `athlete.private.md` |
| ITB & strength authority | Coach evolves the strength routine from the current one (stage 0), with ITB-protective principles baked in |
| Nutrition | Maintain target weight from `athlete.private.md`; simple daily kcal targets sized to the day's training; no food logging |
| COROS write access | **None exists** — the MCP is read-only. The week file is the delivery format, written so sessions are easy to execute or manually enter into the watch |

### Explicit non-goals

- No pushing workouts to the watch and no future-extension design for it
  (dropped by user).
- No checking of the COROS training-schedule/calendar for conflicts (dropped by
  user).
- No food logging or macro reconciliation.
- No web UI, dashboard, or notifications.

## Repo layout & state files

```
blaine-train/
├── athlete.md              # Single source of truth: profile, constraints,
│                           # ITB status, terrain, nutrition goal
├── .claude/skills/coach/   # The coach skills (init, plan-week, today)
├── plan/
│   ├── macro.md            # Macrocycle: now → Divca 2026 → Fruška Gora 130k → 100-miler 2027
│   ├── meso-current.md     # Current 3–4 week block: focus, target loads,
│   │                       # strength progression stage
│   └── week-current.md     # This week, day by day: sessions, kcal, stretching
├── log/
│   ├── weeks/2026-W35.md   # One review per completed week (planned vs actual)
│   └── itb.md              # Append-only ITB symptom log
└── reference/
    ├── strength.md         # Strength routine as versioned stages (stage 0 = current routine)
    ├── routes.md           # Riverbank, fortress, Thursday ride, Fruška Gora access
    └── stretching.md       # 10-minute routines (post-run + after-long-sitting variant)
```

- Plan files are markdown with a small YAML header (dates, planned hours, load
  targets) so both the user and the coach can read, write, and diff them.
- A day in the week file is self-contained: session(s) with purpose and
  structure, heat alternative where relevant, daily kcal target, stretching
  variant, strength details on strength days.
- Everything is git-versioned; every skill run ends with a commit.

### Strength stage 0 (current routine, recorded verbatim in `reference/strength.md`)

Twice a week (one session at 15–16 reps/set, the other at 12 reps/set).
Warm-up: the same exercises without added weight, 10 reps. Then 2 sets of:

1. Lying hip abduction with band at knees, then 30 s hold
2. Single-leg glute bridge, then 20 s hold
3. Single-leg squats with 1.5 L bottle
4. Bulgarian-style single-leg jump-ups (rear foot on sofa) with 1.5 L bottle
5. Single-leg straight-knee calf jumps, 20 reps, with 1.5 L bottle
6. Single-leg bent-knee forward-back jumps ~50 cm, 20 reps, with 1.5 L bottle
7. Archer push-ups, 10 reps

Plus 1 set × 20 reps of three standing bottle-swing exercises (medium step
ahead / arms straight; medium step / arms ~60° raised; large step / vertical
swing).

## Coaching logic

### Periodization

- One macrocycle from now to summer 2027, built backwards from the targets:
  Divca 28 km (2026-09-28) → winter base → Fruška Gora 130 km (2027-04-28) →
  recovery → 100-mile specific block → race.
- Mesocycles of 3–4 weeks (2–3 build + 1 down week).
- Weekly skeleton respects all hard constraints above. Bike volume (including
  the Thursday ride) stays as ITB-friendly aerobic load.
- Under the 9 h cap, ultra preparation leans on back-to-back weekend long runs,
  fortress-stairs elevation repeats, and hiking economy work rather than huge
  single days. A few planned "big days" (trip weekends) are allowed as
  exceptions that the following down week absorbs.

### ITB guardrails (invariants, not suggestions)

- Weekly run volume grows ≤ 10 %.
- Downhill volume is tracked separately and grows slower than total volume.
- No two hard run days back-to-back. A "hard run day" is any run with
  intensity work (threshold/intervals/hills), any long run, or any trail run
  with meaningful descent.
- Strength 2×/week is non-negotiable; it evolves in stages from stage 0 toward
  heavier single-leg loading and hip stability under fatigue.
- Strength stays home-based: bodyweight plus improvised added weight (bottles,
  backpack, band). The coach may propose gym work only if it judges progression
  is genuinely impossible without it, and that is a recommendation for the user
  to accept or decline — never a scheduled default.
- Any reported symptom is appended to `log/itb.md` and automatically downgrades
  the next downhill/long session.
- Escalation (pain returning on easy runs) freezes all progression and switches
  the plan to rehab mode.

### Heat protocol

- Forecast from Open-Meteo for the home city at planning time and at daily check-in.
- ≥ 30 °C at session time → shift the session earlier or to the evening, or run
  by heart rate instead of pace. The riverbank offers a soft surface and easy
  bail-outs but **no shade** — shade exists only in Fruška Gora forest.
- Hard sessions get swapped to a cooler day within the week when possible.
- Long sessions above ~28 °C become run-hike or move to Fruška Gora shade
  (budgeting the access ride).
- Heat counts as load: HR drift acknowledged, paces relaxed.

### Calories

- BMR by Mifflin-St Jeor over the `athlete.private.md` metrics × activity
  factor + session calories from COROS, rounded to 100 kcal steps.
- Carb-emphasis note on long/hard days and the day before.
- Goal is maintenance. Weight is updated only occasionally and manually in
  `athlete.private.md`; the coach never assumes fresh weight data and comments on trend
  only when an update actually happened.

### Stretching

- Fixed 10-minute post-run routine (hip flexors, glutes, TFL, calves) plus a
  variant for after-long-sitting days, assigned per day in the week file.

## Skill workflows

### `/coach:init` — run once (or for major resets)

Reads 12+ months of COROS history (records, load — not the fitness assessment,
which is currently distrusted) including
the 2025 Divca benchmark result; interviews the user for what data can't show
(ITB history detail, strength stage 0 confirmation, route corrections). Writes
`athlete.md`, `reference/*`, `plan/macro.md`, the first `meso-current.md`, and
the first week file. The macrocycle is presented for approval before writing.

### `/coach:plan-week` — weekly (e.g. Sunday evening)

Reads: athlete profile, macro + meso plans, last week's plan, actual COROS
activities and load for the week, ITB log, 7-day forecast.
Does: writes last week's review to `log/weeks/` (planned vs actual, load ratio,
flags); advances or adjusts the mesocycle (down-week trigger, symptom freeze);
writes the new `week-current.md`. Ends by presenting the week in chat for quick
edits (e.g. "long run on Friday this week").

### `/coach:today` — optional, any morning

Reads: week file, yesterday's actual activity, last night's sleep/recovery from
COROS, today's hourly forecast.
Does: confirms or adjusts today only — start time for heat, intensity if
recovery is poor, slides the long run on request. Writes adjustments back into
`week-current.md`. Output is one screen, no ceremony.

All three workflows end with a git commit.

## Error handling

- **COROS MCP unavailable**: `plan-week` asks the user for a short summary of
  the week and marks the review `data: manual`; `today` proceeds from the plan
  file alone. The coach never refuses to coach because the watch won't talk.
- **Weather fetch fails**: sessions are written with the hot-weather alternative
  inline and a "forecast unavailable — pick by feel" note.
- **Missing/corrupt state**: YAML headers are validated on read; a broken week
  file is regenerated from the meso plan; a missing `athlete.md` produces a
  plain "run /coach:init first".
- **Conflicting truth**: COROS actuals beat the plan file; the user's in-chat
  statements beat both.

## Safety invariants (checked before any week is presented)

1. Weekly hours ≤ 9 including strength.
2. Run volume ramp ≤ 10 %.
3. No back-to-back hard run days.
4. ITB symptom within the last 7 days → no downhill progression that week.

A generated week that violates any invariant is fixed before presenting, never
presented with a caveat.

## Testing / validation

1. **Init validation**: after `/coach:init`, check the macrocycle and week 1
   against ground truth — recent actual weeks in COROS (plausible next week for
   this athlete, not a generic template), the 9 h cap, and the race dates.
2. **Dry-run backtest**: run `plan-week` as if it were 4 weeks ago; compare its
   prescription to what the athlete actually did and how that felt. Catches a
   coach that is too aggressive or too timid before it is trusted.
3. **Ongoing**: every weekly review is a regression test — planned-vs-actual
   divergence and ITB log trends are the signals.

## Implementation order (for the plan)

1. Reference files + `athlete.md` skeleton (content largely known already).
2. `/coach:init` skill (produces macro/meso/week from COROS + interview).
3. `/coach:plan-week` skill.
4. `/coach:today` skill.
5. Backtest validation pass.
