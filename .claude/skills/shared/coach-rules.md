# Coach Rules (shared by coach-init, coach-plan-week, coach-today)

Read `athlete.md` first, always, then `athlete.private.md` (gitignored) for
body metrics, age, sex and location. Data precedence: user's in-chat statements >
COROS actuals > plan files. COROS fitness assessment is distrusted — never use
queryFitnessAssessmentOverview as an input.

## Safety invariants — check EVERY generated week before presenting

1. Total planned hours ≤ 9 including strength (and including Fruška Gora
   access rides). Exception (user ruling 2026-08-22): optional easy mobility
   runs (≤8 km) on strength days sit OUTSIDE the core plan — mark them
   OPTIONAL in the week file; they may push the total over 9 h. The cap binds
   the core plan; the ≤10% run-km ramp still counts them when done.
2. Planned run km ≤ 1.10 × last week's ACTUAL run km (from COROS). Down weeks
   go lower; never higher.
3. Downhill: track weekly descent meters (sum of elevation loss across runs,
   via getActivityDetail). Planned descent ≤ 1.10 × the 3-week average of
   actual descent, and any ITB symptom logged in the last 7 days → planned
   descent ≤ last week's actual (no progression). Cold start: when fewer
   than 3 weeks of actual descent exist, use last week's actual as the
   baseline instead of the average.
4. No two hard run days back-to-back. Hard run day = intensity work
   (threshold/intervals/hills/stairs), any long run, or any trail run with
   meaningful descent. Soft adjacency: a QUALITY bike ride scheduled the day
   before the long run must either be downgraded to easy spin or the long
   run's descent front-loaded into its first half — never fresh quality ride
   legs straight into controlled descents on an ITB-rebuilding athlete.
5. Strength 2×/week, home-based (see reference/strength.md). Gym is only ever
   a recommendation the user may decline.
6. Long run sits on Fri/Sat/Sun/Mon only.
7. Load gate: if the weekly review's load ratio exceeds 1.3, the next build
   week is automatically converted to a down week (or hard-capped at ~7 h)
   regardless of momentum.

A week violating any invariant is FIXED before presenting — never presented
with a caveat.

## Periodization

- Macro anchors: Divca 28 km (2026-09-28) → winter base → Fruška Gora 130 km
  (2027-04-28) → 2–3 wk recovery → 100-mile specific block → race (summer
  2027, profile: 100 mi / 9,000 m+; exact race TBD by the user, not the coach).
- Mesocycles 3–4 weeks: 2–3 build + 1 down week (down = roughly 70 % of build
  volume, intensity mostly out); a race taper block may substitute for the
  down week.
- Taper: 2-week pre-race block, final ~10 days true taper, for Divca; 3 weeks
  for the 130k; race-specific block for the 100-miler once the race is
  chosen. Every taper meso for a goal race ≥ 28 km must contain a written
  RACE-EXECUTION RULE: pacing cap, descent strategy (when to walk, cadence/
  stride cues), and the fueling/hydration plan with actual numbers.
- Phase-transition gate: entry into any specific block directly after a
  recovery phase requires ≥ 2 consecutive ITB-symptom-free weeks — state the
  check explicitly when authoring that meso; if not met, extend recovery.
- Under the 9 h cap, ultra prep = back-to-back weekend long runs, fortress
  stair repeats, hiking economy (poles), bike aerobic volume (Thursday ride
  fixed). Occasional planned "big days" (trip weekends) are allowed;
  the following down week absorbs them.
- Interpretation rule: recorded runs > 20 km or > 200 m gain were trails.

## ITB management

- Read log/itb.md before planning anything. Append every reported symptom:
  `YYYY-MM-DD | context | severity 1-5 | note`.
- Symptom in last 7 days → downgrade next downhill/long session, no descent
  progression this week, stay on current strength stage.
- Descent technique cues on every planned descent work: cadence +~5 %,
  shorter and slightly wider step on downhills (both reduce iliotibial
  strain); stop-and-walk at the first lateral tightness stays in force.
- Plyometric sequencing: when authoring a strength stage that ADDS plyos
  while descent progression is active, introduce them in a descent-stable
  week — never ramp both eccentric stimuli the same week.
- Escalation (pain on easy runs returns) → freeze all progression, switch the
  meso to rehab mode: run volume −30 %, no descent work, strength per current
  stage, flag to user prominently.
- Long-sitting days are a known trigger: assign stretching Variant B and
  suggest movement breaks when the user mentions desk-heavy days.

## Heat protocol

Fetch forecast when planning (weekly: daily view; daily: hourly view):

Substitute `$LAT`, `$LON` and `$TZ` from `weather_lat` / `weather_lon` /
`weather_tz` in `athlete.private.md`.

```bash
curl -s "https://api.open-meteo.com/v1/forecast?latitude=$LAT&longitude=$LON&hourly=temperature_2m,precipitation_probability,wind_speed_10m&daily=temperature_2m_max,temperature_2m_min,precipitation_probability_max&timezone=$TZ&forecast_days=7"
```

- ≥ 30 °C at session time → shift earlier/to evening, or convert to
  intensity-by-HR (ignore pace).
- Hard sessions move to the coolest suitable day in the week when possible.
- Long sessions ≥ ~28 °C → run-hike, or Fruška Gora shade (budget access ride).
- Riverbank = soft surface + bail-outs, NOT shade.
- Heat is load: acknowledge HR drift, relax paces.
- Fetch fails → write the day with its hot-weather alternative inline plus
  note "forecast unavailable — pick by feel".

## Calories, fueling & hydration

- BMR (Mifflin-St Jeor, male): 10×weight_kg + 6.25×height_cm − 5×age + 5,
  using `weight_kg` / `height_cm` / `birth_year` from `athlete.private.md`
  (age = current year − birth_year).
- Daily target = BMR × 1.4 (non-training daily life) + planned session kcal
  (estimate from similar past COROS sessions), rounded to nearest 100.
- Long/hard days and the day before: add note "carb emphasis".
- Goal: maintenance at the athlete's target weight. Weight comes only from
  the `athlete.private.md` header; comment on trend only if the user actually
  updated it.

### Intra-session fueling (athlete's actual kit — prescribe items, not abstractions)

Fuel inventory (recorded from the athlete):
- Alesto fruit bar (dried dates/grapes) — 30 g bar, **19.2 g carbs**
- Balans cereal bar — **29.5 g carbs**
- SiS isotonic gel — ~**25 g carbs**
- Haribo Gold-Bears — 77 g carbs per 100 g, bags are 200 g (~15.4 g carbs
  per 30 g portion; portion into a zip bag or pour directly from the bag)

Rules:
- Sessions ≤ 75 min → nothing needed. Sessions > 75–90 min → 1 item per
  30 min (~38–59 g/h depending on mix). Fruit bars need sips of water
  alongside; the isotonic gel does not.
- Big-day/rehearsal blocks (macro Phase 4–5 onward): add a second item to
  the back half of long runs, building toward ~60 g/h race intake. The
  Haribo bag is the densest option — a handful every ~30 min scales past
  60 g/h cheaply and is easy to eat while hiking; keep it as the volume
  backbone once intake targets rise.
- Never debut an untested item on a key session or race day.
- Sodium: electrolyte tabs in bottles for hot sessions > 90 min.
- Hydration default: ~500–750 ml/h in heat; Fruška Gora morning sessions
  need ≥ 2 L carried or an explicit refill point named in the plan.
- Protein: 25–40 g within ~1 h after strength sessions and long runs.

## File formats

### plan/macro.md
```yaml
---
horizon: 2026-08 .. 2027-08
anchors:
  - {date: 2026-09-28, event: "Divca 28k/1453m", goal: "not worse than 2025"}
  - {date: 2027-04-28, event: "Fruska Gora 130k/6000m"}
  - {date: 2027-07-15, event: "100mi/9000m+ (placeholder date, race TBD)"}
updated: YYYY-MM-DD
---
```
Body: one section per phase (name, date range, intent, weekly-hours envelope,
key sessions, strength stage expectation).

### plan/meso-current.md
```yaml
---
meso: N
phase: "name matching a macro phase"
weeks: {start: YYYY-MM-DD, end: YYYY-MM-DD, count: 3-4}
week_types: [build, build, down]        # one entry per week
run_km_targets: [n, n, n]
descent_m_targets: [n, n, n]
strength_stage: 0
updated: YYYY-MM-DD
---
```
Body: block focus, what progresses, what holds, exit criteria to next meso.

### plan/week-current.md
```yaml
---
week: YYYY-Www
dates: {start: YYYY-MM-DD, end: YYYY-MM-DD}
meso_week: "k of n (type)"
planned_hours: n.n          # must be ≤ 9
run_km: n
descent_m: n
long_run_day: fri|sat|sun|mon
updated: YYYY-MM-DD
---
```
Body: one `## Day — date` section per day containing: session(s) with purpose
and simple structure (warm-up / N × work+recovery / cool-down — easy to enter
into a watch manually), hard/easy tag, heat alternative when relevant,
`Calories: ~NNNN kcal (note)`, a `Fueling:` line on sessions > 75 min
(items + per-30-min cadence + fluid volume), `Stretching: A|B`, strength
details on strength days. Every day self-contained.

### log/weeks/YYYY-Www.md
```yaml
---
week: YYYY-Www
data: coros | manual
planned_hours: n.n
actual_hours: n.n
planned_run_km: n
actual_run_km: n
actual_descent_m: n
itb_symptoms: yes|no
---
```
Body: planned vs actual per day (short), load-ratio note from
queryTrainingLoadAssessment, flags for next week, what the user said.

## Degraded modes

- COROS MCP down: plan-week asks the user for a 3-line week summary, marks the
  review `data: manual`; coach-today proceeds from the plan file alone. Never
  refuse to coach.
- Broken/missing week file: regenerate from meso-current.md, say so.
- Missing athlete.md: stop with "run /coach-init first".
- Missing athlete.private.md: stop with "copy athlete.private.example.md to
  athlete.private.md and fill it in".

## Session hygiene

Every skill run ends with: `git add -A && git commit` with a message naming
the skill and week (e.g. `coach: plan week 2026-W36`).
