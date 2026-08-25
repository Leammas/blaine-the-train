# Virtual Trainer/Coach Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a virtual ultra-trail coach as Claude Code skills in this repo that reads COROS data via MCP, plans training weekly, adapts daily, and keeps all state in git-versioned markdown files.

**Architecture:** Three project skills (`coach-init`, `coach-plan-week`, `coach-today`) share one knowledge file (`.claude/skills/shared/coach-rules.md`) holding all coaching logic, file formats, and safety invariants. State lives in `athlete.md`, `plan/`, `log/`, `reference/`. COROS comes in live via the `coros` MCP server; weather via a curl to Open-Meteo. No application code — the deliverables are skill instructions and reference/state documents.

**Tech Stack:** Claude Code project skills (SKILL.md), markdown + YAML headers, COROS MCP (read-only), Open-Meteo REST API (no key), git.

**Spec:** `docs/superpowers/specs/2026-08-22-virtual-coach-design.md`

## Global Constraints

- Weekly training time ≤ 9 hours **including strength** (spec: hard constraint).
- Weekday morning sessions ≤ 1 h (home by 07:00); doubles allowed.
- Thursday social ride is fixed and counts as quality bike work.
- Long run may sit on Friday, Saturday, Sunday, or Monday; never adjacent to another hard run day.
- ITB invariants: run volume ramp ≤ 10 %/week; downhill volume tracked separately and ramps slower; no two hard run days back-to-back; strength 2×/week non-negotiable; symptom in last 7 days → no downhill progression that week.
- Strength stays home-based (bodyweight + improvised weight); gym only ever as an explicit recommendation the user may decline.
- COROS MCP is read-only. No push-to-watch, no COROS-calendar conflict checking (explicit non-goals).
- COROS fitness assessment (VO2max/threshold/predictions) is distrusted — never used as an input.
- Riverbank has **no shade**; shade exists only in Fruška Gora forest.
- Every Fruška Gora trail session budgets the access ride: ~10 km / ~350 m up one way, easy return.
- Data precedence: user's in-chat statements > COROS actuals > plan files.
- Runs > 20 km or > 200 m elevation gain count as trail runs regardless of recorded sport type.
- Every skill run ends with a git commit.
- All coaching output in English; no placeholder text in any produced file.

---

### Task 1: Repo scaffolding, athlete profile, and reference files

**Files:**
- Create: `athlete.md`
- Create: `reference/strength.md`
- Create: `reference/routes.md`
- Create: `reference/stretching.md`
- Create: `plan/.gitkeep`, `log/weeks/.gitkeep`
- Create: `log/itb.md`

**Interfaces:**
- Consumes: nothing (first task).
- Produces: the state files every skill reads. Later tasks refer to these exact paths and to the YAML header fields defined here.

- [ ] **Step 1: Create directory skeleton**

```bash
mkdir -p plan log/weeks reference .claude/skills/shared
touch plan/.gitkeep log/weeks/.gitkeep
```

- [ ] **Step 2: Write `athlete.md`**

```markdown
---
updated: 2026-08-22
# Body metrics, age, sex and location live in `athlete.private.md` (gitignored).
---
# Athlete Profile

## Who

10+ years of running/trail training, UTMB-index runner. Lives in the city
named in `athlete.private.md`. The COROS fitness assessment (VO2max, threshold, race predictions) is
currently DISTRUSTED and must not be used as an input; judge fitness from
actual recent sessions in COROS history.

## Goals

1. **Primary: 100-mile ultra trail, 9,000 m+ gain, summer 2027.** Race not yet
   chosen — plan targets the profile, not a date. Firm up once picked.
2. **Divca Trail, 2026-09-28, 28 km / 1,453 m+.** Goal: not worse than the 2025
   result (in COROS history — find it around late Sept 2025).
3. **Fruška Gora Ultra Trail, 2027-04-28, 130 km / 6,000 m+.**

## Hard constraints

- ≤ 9 training hours/week INCLUDING strength.
- Weekday morning sessions ≤ 1 h (home by 07:00). Doubles allowed.
- Thursday social ride is fixed (see reference/routes.md); counts as bike quality.
- Long run slot is flexible: Fri/Sat/Sun/Mon all valid; user may slide it at
  daily check-in. Never adjacent to another hard run day.
- Weekends (and long-run Fri/Mon) are uncapped in duration.

## ITB status

- IT band injury reduced the 2026 season. No pain on easy runs now; symptoms
  still appear after long sitting sessions.
- Last 3 months of strength were ITB-focused. Current routine = stage 0 in
  reference/strength.md.
- Symptom log: log/itb.md (append-only). Escalation = pain returning on easy
  runs → freeze progression, rehab mode.

## Equipment & terrain

- Bike: gravel.
- Home-city proximity: paved flat roads; ~10 km riverbank path with soft gummy
  surface (NO SHADE); Petrovaradin fortress: ~40 m stairway gain + cobblestone
  roads.
- Fruška Gora access: ~10 km ride, ~350 m up one way; easy ride back. Budget
  this into every FG session's hours and load.
- Interpretation rule: any recorded run > 20 km or > 200 m gain was a trail run.

## Nutrition

- Goal: maintain the target weight in `athlete.private.md`. No food logging. Daily kcal targets per
  coach-rules.md formula; carb emphasis on/before long or hard days.
- Weight updated occasionally by hand in `athlete.private.md`; comment on
  trend only when an update actually happened.
```

- [ ] **Step 3: Write `reference/strength.md`**

```markdown
---
current_stage: 0
updated: 2026-08-22
---
# Strength Routine — Versioned Stages

Constraint: home-based, bodyweight + improvised weight (1.5 L bottles,
backpack, band). Gym may only ever be RECOMMENDED by the coach if progression
is genuinely impossible without it; user accepts or declines. Never scheduled
by default. Frequency: 2×/week, non-negotiable.

## Stage 0 (current — the athlete's own routine, recorded verbatim)

Two sessions/week: session A at 15–16 reps/set, session B at 12 reps/set.
Warm-up: same exercises without added weight, 10 reps.

Then 2 sets of:
1. Lying hip abduction, band at knees, then 30 s hold
2. Single-leg glute bridge, then 20 s hold
3. Single-leg squats with 1.5 L bottle
4. Single-leg jump-ups, rear foot on sofa, with 1.5 L bottle
5. Single-leg straight-knee calf jumps, 20 reps, with 1.5 L bottle
6. Single-leg bent-knee forward-back jumps ~50 cm, 20 reps, with 1.5 L bottle
7. Archer push-ups, 10 reps

Plus 1 set × 20 reps each:
1. Medium step ahead, swinging bottle left–right, arms straight ahead
2. Medium step ahead, swinging bottle left–right, arms ~60° raised
3. Large step ahead, swinging bottle up–down, arms straight ahead

## Progression principles (for the coach when authoring stage 1+)

- Progress ONE variable at a time: load (heavier backpack/bottles), then range,
  then instability, then fatigue-state placement (strength after easy runs).
- Direction: heavier single-leg loading and hip stability under fatigue.
- A new stage is proposed in a weekly plan, appended here as "Stage N" with
  full exercise list, and only becomes current when the user confirms.
- Any ITB symptom in the last 7 days → stay on current stage.
```

- [ ] **Step 4: Write `reference/routes.md`**

```markdown
# Routes & Venues

## Riverbank path
~10 km, flat, soft gummy surface. NO SHADE — do not treat as heat mitigation.
Use: easy runs, strides, surface-friendly volume, easy bail-out points.

## Petrovaradin fortress
~40 m gain stairway + cobblestone road options. Use: stair repeats for
elevation work, downhill-technique doses in small controlled amounts.

## Thursday social ride (fixed)
~15 km flat → 100 m + 400 m climb at ~10 % avg → ~20 km back with small hills
and a downhill section. Counts as bike quality. Plan around it, never over it.

## Fruška Gora (main trail terrain)
Access: ~10 km ride with ~350 m gain one way; return is easy. Budget the
commute into the day's hours and load (legitimate warm-up + aerobic work).
Forest = the only real shade option in summer. Trails suitable for long runs,
vert accumulation, hiking economy work.

## Weather venue logic
- Heat ≥ 30 °C at session time: shift earlier/to evening, or run by HR not pace.
- Long sessions above ~28 °C: run-hike, or move to Fruška Gora shade
  (with access ride budgeted).
```

- [ ] **Step 5: Write `reference/stretching.md`**

```markdown
# Stretching — 10-minute routines (hard cap)

## Variant A — post-run (default)
1. Standing TFL/ITB cross-over stretch — 45 s/side
2. Figure-4 glute stretch (standing or lying) — 45 s/side
3. Half-kneeling hip-flexor lunge, tall posture — 45 s/side
4. Standing hamstring hinge, flat back — 45 s/side
5. Wall calf stretch, straight knee then bent knee — 30 s + 30 s per side
6. Quad pull with glute squeeze — 30 s/side

## Variant B — after long sitting days (ITB trigger days)
1. Half-kneeling hip-flexor lunge with side reach — 60 s/side
2. Standing TFL/ITB cross-over stretch — 60 s/side
3. Figure-4 glute stretch — 45 s/side
4. Thoracic open-book on floor — 45 s/side
5. Hamstring hinge — 30 s/side
6. Calf stretch straight-knee — 30 s/side

Rules: never stretch into pain; assign Variant B on days the athlete reports
long sitting or any ITB tightness; both variants fit in 10 minutes.
```

- [ ] **Step 6: Write `log/itb.md`**

```markdown
# ITB Symptom Log (append-only)

Format, one line per entry:
`YYYY-MM-DD | context (run/sitting/other) | severity 1-5 | note`

Baseline 2026-08-22: no pain on easy runs; symptoms still appear after long
sitting sessions. Last 3 months of strength were ITB-focused.
```

- [ ] **Step 7: Verify structure**

```bash
ls athlete.md reference/strength.md reference/routes.md reference/stretching.md log/itb.md plan/.gitkeep log/weeks/.gitkeep
grep -c "DISTRUSTED" athlete.md   # expect 1
grep -c "NO SHADE" reference/routes.md   # expect 1
```
Expected: all files listed, both greps return 1.

- [ ] **Step 8: Commit**

```bash
git add -A && git commit -m "feat: athlete profile, reference files, state skeleton"
```

---

### Task 2: Shared coach rules and file-format templates

**Files:**
- Create: `.claude/skills/shared/coach-rules.md`

**Interfaces:**
- Consumes: paths from Task 1.
- Produces: `.claude/skills/shared/coach-rules.md` — every SKILL.md in Tasks 3–5 begins with "Read `.claude/skills/shared/coach-rules.md` in full before doing anything." It defines: the safety invariants, periodization rules, heat protocol, calorie formula, the exact YAML headers and day-entry format for `plan/macro.md`, `plan/meso-current.md`, `plan/week-current.md`, `log/weeks/YYYY-Www.md`, and the exact Open-Meteo command.

- [ ] **Step 1: Write `.claude/skills/shared/coach-rules.md`**

````markdown
# Coach Rules (shared by coach-init, coach-plan-week, coach-today)

Read `athlete.md` first, always. Data precedence: user's in-chat statements >
COROS actuals > plan files. COROS fitness assessment is distrusted — never use
queryFitnessAssessmentOverview as an input.

## Safety invariants — check EVERY generated week before presenting

1. Total planned hours ≤ 9 including strength (and including Fruška Gora
   access rides).
2. Planned run km ≤ 1.10 × last week's ACTUAL run km (from COROS). Down weeks
   go lower; never higher.
3. Downhill: track weekly descent meters (sum of elevation loss across runs,
   via getActivityDetail). Planned descent ≤ 1.10 × the 3-week average of
   actual descent, and any ITB symptom logged in the last 7 days → planned
   descent ≤ last week's actual (no progression).
4. No two hard run days back-to-back. Hard run day = intensity work
   (threshold/intervals/hills/stairs), any long run, or any trail run with
   meaningful descent.
5. Strength 2×/week, home-based (see reference/strength.md). Gym is only ever
   a recommendation the user may decline.
6. Long run sits on Fri/Sat/Sun/Mon only.

A week violating any invariant is FIXED before presenting — never presented
with a caveat.

## Periodization

- Macro anchors: Divca 28 km (2026-09-28) → winter base → Fruška Gora 130 km
  (2027-04-28) → 2–3 wk recovery → 100-mile specific block → race (summer
  2027, profile: 100 mi / 9,000 m+; exact race TBD by the user, not the coach).
- Mesocycles 3–4 weeks: 2–3 build + 1 down week (down = roughly 70 % of build
  volume, intensity mostly out).
- Taper: ~10 days for Divca; 3 weeks for the 130k; race-specific block for the
  100-miler once the race is chosen.
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
- Escalation (pain on easy runs returns) → freeze all progression, switch the
  meso to rehab mode: run volume −30 %, no descent work, strength per current
  stage, flag to user prominently.
- Long-sitting days are a known trigger: assign stretching Variant B and
  suggest movement breaks when the user mentions desk-heavy days.

## Heat protocol

Fetch forecast when planning (weekly: daily view; daily: hourly view):

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

## Calories

- BMR (Mifflin-St Jeor, male): 10×weight_kg + 6.25×height_cm − 5×age + 5.
  Values from `athlete.private.md` (age = current year − birth_year).
- Daily target = BMR × 1.4 (non-training daily life) + planned session kcal
  (estimate from similar past COROS sessions), rounded to nearest 100.
- Long/hard days and the day before: add note "carb emphasis".
- Goal: maintenance at the target weight. Weight comes only from the
  athlete.private.md header;
  comment on trend only if the user actually updated it.

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
`Calories: ~NNNN kcal (note)`, `Stretching: A|B`, strength details on strength
days. Every day self-contained.

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

## Session hygiene

Every skill run ends with: `git add -A && git commit` with a message naming
the skill and week (e.g. `coach: plan week 2026-W36`).
````

- [ ] **Step 2: Verify**

```bash
grep -c "Safety invariants" .claude/skills/shared/coach-rules.md   # expect 1
grep -c "open-meteo.com" .claude/skills/shared/coach-rules.md      # expect 1
grep -c "queryFitnessAssessmentOverview" .claude/skills/shared/coach-rules.md  # expect 1 (as a prohibition)
```
Expected: 1 / 1 / 1.

- [ ] **Step 3: Commit**

```bash
git add -A && git commit -m "feat: shared coach rules, formats, and invariants"
```

---

### Task 3: `coach-init` skill

**Files:**
- Create: `.claude/skills/coach-init/SKILL.md`

**Interfaces:**
- Consumes: `athlete.md`, `reference/*`, `.claude/skills/shared/coach-rules.md`, COROS MCP tools (`querySportRecords`, `queryTrainingLoadAssessment`, `getActivityDetail`).
- Produces: on execution, `plan/macro.md`, `plan/meso-current.md`, `plan/week-current.md` in the exact formats from coach-rules.md. Later skills read those files.

- [ ] **Step 1: Write `.claude/skills/coach-init/SKILL.md`**

````markdown
---
name: coach-init
description: One-time (or major-reset) setup of the training macrocycle to the summer-2027 100-miler. Use when plan/ is empty, when the user asks to rebuild the plan from scratch, or after a season-changing event (injury escalation, race change). Reads 12+ months of COROS history, interviews the user, writes macro/meso/week plans.
---

# Coach Init

Read `.claude/skills/shared/coach-rules.md` in full, then `athlete.md`,
`reference/strength.md`, `reference/routes.md`, `log/itb.md`. If plan/ already
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
by default.

## 3. Draft the macrocycle

Backwards from anchors (see coach-rules.md periodization). Present the phase
plan IN CHAT for approval before writing any file. Iterate until approved.

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
- The 2025 Divca benchmark is recorded in macro.md.

## 6. Commit

`git add -A && git commit -m "coach: init macrocycle to summer 2027"`

## Degraded mode

COROS unavailable → interview the user for a typical-week summary and last
month's pattern; mark macro.md header with `data: manual`; continue.
````

- [ ] **Step 2: Verify skill registers**

```bash
ls .claude/skills/coach-init/SKILL.md
head -5 .claude/skills/coach-init/SKILL.md   # frontmatter with name: coach-init
```
Then in a Claude Code session: confirm `/coach-init` appears in the skills list (or `Skill` tool lists it). Expected: present.

- [ ] **Step 3: Commit**

```bash
git add -A && git commit -m "feat: coach-init skill"
```

---

### Task 4: `coach-plan-week` skill

**Files:**
- Create: `.claude/skills/coach-plan-week/SKILL.md`

**Interfaces:**
- Consumes: all plan/ files from Task 3's execution, `log/itb.md`, coach-rules formats, COROS MCP, Open-Meteo command from coach-rules.md.
- Produces: on execution, `log/weeks/YYYY-Www.md` (review of the completed week) and a fresh `plan/week-current.md`. Updates `plan/meso-current.md` when a block ends or adjusts.

- [ ] **Step 1: Write `.claude/skills/coach-plan-week/SKILL.md`**

````markdown
---
name: coach-plan-week
description: Weekly planning session (typically Sunday evening). Reviews the completed week against COROS actuals, advances the mesocycle, writes next week's plan with sessions, calories, and stretching. Use when the user asks to plan the week, review the week, or says the week is over.
---

# Coach Plan Week

Read `.claude/skills/shared/coach-rules.md` in full, then `athlete.md`,
`plan/macro.md`, `plan/meso-current.md`, `plan/week-current.md`,
`log/itb.md`. Missing athlete.md → stop: "run /coach-init first".
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

## 3. Advance the mesocycle

- Meso finished → author the next one from macro.md (present block focus in
  chat first).
- Symptom escalation (per coach-rules ITB section) → rehab-mode meso, flag
  prominently.
- Down-week due → schedule it regardless of momentum.
- Update meso-current.md header/body if anything changed.

## 4. Draft next week

- Fetch the 7-day forecast (coach-rules curl, daily view).
- Build the week within the skeleton: Thursday ride fixed; strength 2×;
  long run on the user's preferred day (Fri/Sat/Sun/Mon — ask if unclear);
  weekday sessions ≤ 1 h; FG sessions budget the access ride; heat rules
  applied (hard sessions on cooler days, ≥30 °C → time shift or by-HR).
- Calories per coach-rules formula, per day. Stretching A default, B on
  known long-sitting days.
- Run ALL safety invariants; fix violations before presenting.

## 5. Present and finalize

Show the week in chat (compact table + notes). Apply user edits (e.g. "long
run Friday"), re-check invariants after edits, then write
plan/week-current.md.

## 6. Commit

`git add -A && git commit -m "coach: plan week YYYY-Www"`
````

- [ ] **Step 2: Verify**

```bash
ls .claude/skills/coach-plan-week/SKILL.md
grep -c "safety invariants" .claude/skills/coach-plan-week/SKILL.md  # expect ≥1 (case-insensitive: grep -ci)
```
Expected: file present, grep ≥ 1.

- [ ] **Step 3: Commit**

```bash
git add -A && git commit -m "feat: coach-plan-week skill"
```

---

### Task 5: `coach-today` skill

**Files:**
- Create: `.claude/skills/coach-today/SKILL.md`

**Interfaces:**
- Consumes: `plan/week-current.md`, COROS MCP (`querySportRecords` for yesterday, `querySleepData`/`queryRecoveryStatus`), Open-Meteo hourly view.
- Produces: on execution, an edited `plan/week-current.md` (today's day section updated in place, other days rebalanced only when the long run slides).

- [ ] **Step 1: Write `.claude/skills/coach-today/SKILL.md`**

````markdown
---
name: coach-today
description: Short daily check-in (any morning). Confirms or adjusts ONLY today's session for weather, recovery, and yesterday's actuals; can slide the long run on request. Use when the user asks what's today, says how they slept/feel, or wants to move today's session.
---

# Coach Today

Read `.claude/skills/shared/coach-rules.md` (at minimum: invariants, heat
protocol, ITB section), then `plan/week-current.md`. No week file →
"run /coach-plan-week first". Keep the whole interaction to ONE screen of
output — no ceremony.

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
target, stretching variant, one-line reason for any change.

## 4. Persist

Write the adjustment into plan/week-current.md (today's section; adjacent
days only if the long run moved), bump `updated:` in the header, then
`git add -A && git commit -m "coach: adjust YYYY-MM-DD"`.
If nothing changed, say so and skip the commit.
````

- [ ] **Step 2: Verify**

```bash
ls .claude/skills/coach-today/SKILL.md
grep -ci "one screen" .claude/skills/coach-today/SKILL.md   # expect ≥1
```
Expected: file present, grep ≥ 1.

- [ ] **Step 3: Commit**

```bash
git add -A && git commit -m "feat: coach-today skill"
```

---

### Task 6: Live run of `/coach-init` (produces the actual plan)

**Files:**
- Create (by the skill, not by hand): `plan/macro.md`, `plan/meso-current.md`, `plan/week-current.md`

**Interfaces:**
- Consumes: everything from Tasks 1–3, live COROS MCP, the user (interview + approvals).
- Produces: the real macrocycle to summer 2027 and the first planned week — the state Tasks 7 depends on.

**Note:** this task REQUIRES the user present (interview + macro approval). It cannot be delegated to an unattended subagent.

- [ ] **Step 1: Run the skill**

In this repo's Claude Code session, invoke `/coach-init`. Follow it end to end: history pull, interview, macro approval in chat, file writing.

- [ ] **Step 2: Validate outputs mechanically**

```bash
ls plan/macro.md plan/meso-current.md plan/week-current.md
grep -c "2026-09-28" plan/macro.md    # Divca anchor present, expect ≥1
grep -c "2027-04-28" plan/macro.md    # FG 130k anchor present, expect ≥1
awk '/^planned_hours:/{print $2}' plan/week-current.md   # value must be ≤ 9
```
Expected: three files exist, both anchors found, planned hours ≤ 9.

- [ ] **Step 3: Validate against ground truth (judgment check, with the user)**

Confirm in chat: week 1 looks like a plausible next week versus the athlete's recent COROS weeks (volumes within ~10 % of recent actuals, long run consistent with recent long runs, Thursday ride present, strength 2×, no back-to-back hard run days); the 2025 Divca benchmark is recorded in macro.md.

- [ ] **Step 4: Commit** (the skill commits; verify)

```bash
git log --oneline -1    # expect a "coach: init" commit
```

---

### Task 7: Backtest validation of `coach-plan-week`

**Files:**
- Create: `docs/superpowers/validation/2026-08-22-backtest.md`

**Interfaces:**
- Consumes: Tasks 1–6 outputs, COROS history for the 4-weeks-ago window.
- Produces: a written verdict on whether the coach is too aggressive, too timid, or calibrated — plus any rule adjustments applied to `coach-rules.md`.

**Note:** requires the user for the "how did it actually feel" comparison.

- [ ] **Step 1: Run the backtest**

Invoke `/coach-plan-week` with the explicit instruction: "Backtest mode: plan as if today were 2026-07-26 (so 'last week' = 2026-07-20..26). Do NOT overwrite plan/week-current.md — write the generated week to docs/superpowers/validation/2026-08-22-backtest.md instead, alongside the review it would have written."

- [ ] **Step 2: Compare against reality**

In the same file, add a comparison section: the backtest week vs what the athlete ACTUALLY did 2026-07-27..08-02 (from COROS: which included a 118 km ride and a load spike COROS called "Excessive" on Aug 2–3). Ask the user how that week actually felt. Judge: would the coach's week have been safer/better? Any invariant the real week violated that the coach correctly avoids (the Excessive spike is the expected catch)?

- [ ] **Step 3: Apply lessons**

If the backtest shows miscalibration (e.g. volume steps too timid for this athlete's demonstrated tolerance, or big-day handling too rigid), adjust the relevant rule in `.claude/skills/shared/coach-rules.md` and note the change in the backtest file. If calibrated, state that explicitly.

- [ ] **Step 4: Verify and commit**

```bash
ls docs/superpowers/validation/2026-08-22-backtest.md
grep -ci "verdict" docs/superpowers/validation/2026-08-22-backtest.md  # expect ≥1
git add -A && git commit -m "test: backtest coach-plan-week against July actuals"
```

---

## Self-Review (completed)

- **Spec coverage:** athlete/goals/constraints → Task 1; coaching logic, formats, heat, calories, invariants, degraded modes → Task 2; three workflows → Tasks 3–5; init validation → Task 6; backtest → Task 7; ongoing weekly reviews are produced by Task 4's skill by design. Non-goals (push-to-watch, calendar checks, food logging, UI) appear nowhere. No gaps found.
- **Placeholder scan:** all file contents are written in full; no TBD/TODO. The 100-miler date in macro format is labeled "placeholder date, race TBD" — that is spec-intended (race not chosen), not a plan placeholder.
- **Type consistency:** file paths, YAML field names (`planned_hours`, `long_run_day`, `data: manual`, `strength_stage`), skill names (`coach-init`, `coach-plan-week`, `coach-today`), and MCP tool names are identical across tasks.
