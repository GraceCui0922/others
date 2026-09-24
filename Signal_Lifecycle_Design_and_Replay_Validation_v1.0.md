# Signal Lifecycle Design & Lifecycle Replay Validation

**Version:** v1.0  
**Scope:** Standard Chartered Individual Decision Agent — Signal Lifecycle  
**Status:** Working design for Signal episode construction, validation and production governance  
**Primary source basis:** `Signal_Event_Change_Detection_Validation_Research_Paper_v1.0.md` and agreed project design

---

## 1. Purpose

The detector produces a time series of numeric detection statistics.

The lifecycle layer converts those repeated detector outputs into **governed Signal Episodes**.

The core distinction is:

> **Detector:** Is the factual measure materially unusual / changed at this point in time?  
> **Lifecycle:** Do these repeated detections belong to one Signal episode or multiple episodes, and when does the episode begin, remain active, resolve, expire or reopen?

Without lifecycle logic, the system only produces daily anomaly scores or repeated alerts; it does not produce stable, auditable Signal objects suitable for State / Need consumption.

---

## 2. Position in the Signal architecture

```text
Semantic Evidence
        ↓
Indicator
        ↓
Baseline
        ↓
Detector
        ↓
Numeric signal_value(t)
        ↓
Activation / Quality Gate
        ↓
LIFECYCLE ENGINE
        ↓
Signal Episode
        ↓
State / Need consumers
```

Lifecycle operates **after numeric detection** and before downstream inference.

---

## 3. Detector firing vs Signal episode

Example numeric stream:

```text
Date      signal_value
D1        0.4
D2        0.7
D3        2.1
D4        2.8
D5        3.4
D6        3.0
D7        2.5
D8        1.7
D9        0.9
```

The detector may flag multiple consecutive days.

The lifecycle engine should usually create:

```text
ONE EPISODE
onset_at      = D4
peak_value    = 3.4
duration      = defined by lifecycle rule
resolved_at   = D9 (illustrative only)
```

rather than generating a new Signal for every day above threshold.

---

# PART A — LIFECYCLE SPECIFICATION

## 4. LifecycleSpec — minimum required components

Each Signal family should have its own `LifecycleSpec`.

At minimum it must define:

1. Onset
2. Persistence
3. Active / Maintenance rule
4. Resolution
5. Expiry
6. Cooldown
7. Reopen
8. Deduplication / episode merge
9. Missing-data behaviour
10. Source correction behaviour
11. Parameter provenance

Different Signal families may have different lifecycle semantics.

---

## 5. Onset rule

Onset answers:

> When does a new candidate Signal episode begin?

Generic form:

```text
onset when:
  signal_value crosses activation boundary
  AND quality gate passes
  AND minimum persistence condition passes
```

Possible designs:

```text
A. immediate onset
B. N consecutive observations above boundary
C. K of last N observations above boundary
D. event-authoritative onset
E. change-point confirmed onset
```

The exact rule is Signal-family specific.

---

## 6. Persistence rule

Persistence prevents one-off noise from becoming an episode.

Examples:

```text
3 consecutive valid observations
5 of last 7 valid observations
signal_value above boundary for ≥ X calendar days
```

Persistence parameters must be calibrated / validated, not invented by the runtime agent.

---

## 7. Activation and maintenance thresholds

For continuous Signals, use separate concepts where justified:

```text
activation_threshold
maintenance_threshold
exit_threshold
```

This allows hysteresis.

Illustrative example only:

```text
activate above 3.0
remain active above 1.5
resolve below 1.5 for N observations
```

Why hysteresis matters:

```text
2.9 → 3.1 → 2.9 → 3.2 → 2.8 → 3.1
```

Without hysteresis the episode may repeatedly switch ON/OFF.

Threshold values must be empirically calibrated or otherwise governed.

---

## 8. Active status

Once an episode is active, the engine should update rather than recreate the Signal.

Update fields may include:

```text
last_seen_at
current_value
peak_value
peak_at
episode_age
persistence_count
quality_status
supporting_evidence_refs
contradictory_evidence_refs
```

Repeated detector firing should update the current episode unless a defined split / reopen condition applies.

---

## 9. Resolution rule

Resolution answers:

> When has the detected material condition/change ended sufficiently to close the episode?

Candidate formulations:

```text
value below exit threshold for N observations
return to baseline band for N days
authoritative event resolved
change-point regime ends
supporting factual event reversed / completed
```

Resolution should not occur merely because one day is missing or one observation falls slightly below threshold unless explicitly designed that way.

---

## 10. Expiry rule

Expiry prevents stale Signals from surviving indefinitely.

Possible designs:

```text
hard maximum episode age
known event date expiry
validity horizon from detection
expiry after evidence freshness breach
```

Example use cases:

- maturity Signal expires at / after factual maturity event
- anomaly Signal expires after a maximum validity horizon if not reconfirmed

Expiry is different from resolution:

- **Resolution:** evidence suggests the condition ended
- **Expiry:** the Signal is no longer valid to carry forward even if a clean resolution cannot be observed

---

## 11. Cooldown rule

Cooldown answers:

> After an episode closes, how long should the system suppress minor re-triggering from the same underlying episode?

Cooldown prevents alert fragmentation.

Possible forms:

```text
fixed N days
N valid observations
until baseline is re-established
until factual event resets
```

Cooldown must not suppress a genuinely new material episode; therefore cooldown and reopen logic must be designed together.

---

## 12. Reopen rule

Reopen answers:

> Under what conditions can a previously resolved Signal family create a new episode?

Possible rule components:

```text
cooldown completed
AND new material departure occurs
AND new peak / new event exceeds reopen criterion
AND quality gate passes
```

Alternative design:

```text
same episode is reopened
```

vs.

```text
new episode_id is created
```

The design must explicitly define whether the event is a continuation or a genuinely new episode.

---

## 13. Deduplication / episode merge rule

Common problem:

```text
same underlying event
→ multiple related detector firings
→ duplicate Signals
```

LifecycleSpec should define:

- episode key
- temporal merge window
- same-family merge criteria
- cross-indicator supporting evidence rules
- whether multiple detectors map to one Signal family

Typical principle:

> repeated detector firings inside the same factual episode update one Signal Episode rather than create multiple independent Signals.

---

## 14. Missing-data rule

Lifecycle must explicitly distinguish:

```text
TRUE_ZERO / NORMAL
SOURCE_MISSING
NOT_OBSERVED
INSUFFICIENT_HISTORY
STALE_SOURCE
```

Examples:

```text
missing data during active episode
≠ automatic resolution
```

Possible handling:

```text
hold status for grace period
mark QUALITY_DEGRADED
abstain from resolution
expire if source stale beyond allowed horizon
```

No generic hidden fallback should be permitted.

---

## 15. Source correction / backfill rule

If upstream Evidence can be corrected retrospectively, define whether lifecycle is:

```text
A. historically reconstructed / re-stated
B. append-only with correction events
C. production history frozen but analytical history replayable
```

For governance, the preferred design is usually:

- preserve what production knew at the time
- allow research / audit reconstruction using corrected PIT data
- never silently overwrite production decision history

---

## 16. Parameter provenance

Every lifecycle parameter must carry provenance.

Examples:

```text
persistence_days: CALIBRATION_REQUIRED
exit_threshold: EMPIRICALLY_ESTIMATED
cooldown_days: DOMAIN_EXPERT_ELICITED + HISTORICAL_VALIDATION
max_episode_age: POLICY_DEFINED
```

The implementation engine must not invent these values.

---

# PART B — HOW TO DESIGN LIFECYCLE RULES

## 17. Source 1 — Detector firing behaviour

Historical detector output should be profiled for:

- isolated spikes
- continuous runs
- frequent threshold crossing
- temporary dips
- long tails
- repeated peaks
- time between episodes

Purpose:

> understand how the detector behaves before defining episode rules.

---

## 18. Source 2 — Business / event semantics

Lifecycle is family-specific.

### Example: deterministic maturity Signal

Possible structure:

```text
onset = enters maturity horizon
active = until maturity / factual completion
resolution = maturity event completed
expiry = event date + governed grace period
```

### Example: behavioural / anomaly Signal

Possible structure:

```text
onset = sustained material departure
active = departure persists
resolution = stable return to expected band
cooldown = suppress short-term oscillation
reopen = new independent material departure
```

Therefore no universal lifecycle template should be blindly shared across all Signal families.

---

## 19. Source 3 — Historical episode distribution

Historical replay should estimate:

- run-length distribution above threshold
- run-length distribution below exit threshold
- gap distribution between detector firings
- episode duration distribution
- time to re-trigger
- time to baseline recovery

These empirical distributions help calibrate:

- persistence
- merge window
- resolution grace period
- cooldown
- reopen criteria

---

# PART C — LIFECYCLE REPLAY VALIDATION

## 20. Core idea

Lifecycle Replay Validation replays historical numeric detector output under competing lifecycle specifications.

Input:

```text
client_id × date × signal_value
```

Candidate lifecycle configurations:

```text
Lifecycle A
Lifecycle B
Lifecycle C
...
```

Output:

```text
episode_id
onset_at
detected_at
peak_value
peak_at
duration
resolved_at
expiry_at
reopen_count
status
```

Then compare which lifecycle specification best represents coherent Signal episodes.

---

## 21. Candidate lifecycle grid

Example structure only:

```text
Persistence:
1 / 3 / 5 / K-of-N

Exit confirmation:
1 / 3 / 5 valid observations

Cooldown:
short / medium / long candidate windows

Reopen:
new threshold exceedance
new material peak
new authoritative event
```

Candidate values must be declared as sensitivity / calibration parameters before confirmatory evaluation.

---

## 22. Replay validation metrics

### 22.1 Fragmentation rate

Question:

> Is one obvious factual episode being split into multiple Signal episodes?

High fragmentation may indicate:

- exit rule too aggressive
- cooldown too short
- merge rule too strict

---

### 22.2 Over-merging rate

Question:

> Are two distinct events being incorrectly merged into one episode?

High over-merging may indicate:

- cooldown too long
- reopen rule too strict
- resolution too sticky

---

### 22.3 Onset / detection delay

Measure:

```text
factual change start
→ episode onset / detected_at
```

Too much delay may mean persistence is too long or detector confirmation is too conservative.

---

### 22.4 Resolution delay

Measure:

```text
condition returns to normal
→ episode resolution
```

Too much delay creates stale Signals.

---

### 22.5 Episode duration distribution

Monitor:

- median duration
- long-tail duration
- extreme active duration
- very short episode share

Unexpected distributions indicate lifecycle mis-specification.

---

### 22.6 Reopen quality

Measure whether reopened episodes represent genuine new material departures.

Review:

- reopen shortly after close
- reopen after minor oscillation
- reopen after true new event

---

### 22.7 Duplicate burden

Measure:

```text
multiple episode_ids apparently referring to same underlying factual episode
```

---

### 22.8 Operational burden

Report:

```text
new episodes / day
active episodes / day
resolved episodes / day
reopened episodes / day
```

Operational burden is a governance input, not the sole basis for tuning lifecycle rules.

---

# PART D — HUMAN LIFECYCLE REVIEW

## 23. Human review objective

Human review does not answer:

> Does the client have a Need?

It answers:

> Are the Signal episode boundaries semantically reasonable given the factual time series?

---

## 24. Review sample design

Prioritise:

- very short episodes
- very long episodes
- frequent reopen
- repeated threshold crossing
- close-then-immediate-reopen
- near-boundary episodes
- high-magnitude but short-lived events
- missing-data periods
- source corrections

---

## 25. Human review pack

Show:

```text
factual Evidence time series
numeric signal_value
baseline
activation threshold
maintenance / exit threshold
quality status
proposed episode boundaries
relevant factual events
```

Reviewer verdicts:

```text
CORRECT_EPISODE
SHOULD_MERGE
SHOULD_SPLIT
ONSET_TOO_EARLY
ONSET_TOO_LATE
RESOLUTION_TOO_EARLY
RESOLUTION_TOO_LATE
INVALID_REOPEN
DATA_ISSUE
AMBIGUOUS
```

---

# PART E — SYNTHETIC LIFECYCLE TESTS

## 26. Purpose

Synthetic sequences are highly useful for deterministic lifecycle regression tests.

They test whether the implementation follows `LifecycleSpec`.

They do not prove that the lifecycle is valid for the real bank population.

---

## 27. Minimum synthetic fixture set

### Case 1 — one-day spike

```text
normal → spike 1 day → normal
```

Expected outcome depends on persistence rule; often no episode for persistence-based families.

### Case 2 — sustained change

```text
normal → sustained high → normal
```

Expected:

```text
ONE EPISODE
```

### Case 3 — temporary dip

```text
high → 1-day dip → high
```

Expected:

```text
SAME EPISODE
```

if the lifecycle design includes hysteresis / resolution grace.

### Case 4 — clearly separated events

```text
high episode → long normal period → high episode
```

Expected:

```text
TWO EPISODES
```

### Case 5 — missing data during active episode

```text
active → missing 3 days → active
```

Expected behaviour must follow explicit missingness rule and should not silently resolve.

### Case 6 — source correction

Historical evidence changes after correction.

Test both:

- audit reconstruction
- production-history preservation policy

### Case 7 — cooldown boundary

New firing occurs:

```text
before cooldown ends
exactly at cooldown end
after cooldown ends
```

### Case 8 — reopen threshold

Test:

- small post-close noise
- material new peak
- new authoritative event

---

# PART F — LIFECYCLE OUTPUT CONTRACT

## 28. SignalEpisode

Recommended fields:

```text
episode_id
signal_definition_id
client_id

onset_at
detected_at
last_seen_at
resolved_at
expiry_at
cooldown_until

status
current_value
peak_value
peak_at
persistence_count
episode_age

quality_status
missingness_status
reopen_count
parent_episode_id

supporting_evidence_refs
contradictory_evidence_refs

lifecycle_version
detector_version
parameter_version
validation_release_id
reason_codes
```

Possible statuses:

```text
CANDIDATE
ACTIVE
MONITORING
DECAYING
RESOLVED
EXPIRED
REJECTED
QUALITY_HOLD
```

Exact status taxonomy should be frozen at implementation-contract stage.

---

# PART G — LIFECYCLE VALIDATION ARTIFACT

## 29. LifecycleValidationResult

Recommended fields:

```text
signal_definition_id
lifecycle_candidate_id
historical_period
population_scope

clients_tested
episodes_created
median_episode_duration
short_episode_rate
long_episode_rate
fragmentation_rate
overmerge_rate
reopen_rate
stale_active_rate
missing_data_hold_rate

detection_delay_distribution
resolution_delay_distribution

human_review_sample_size
human_review_agreement
human_review_error_types

synthetic_test_pass_rate
negative_control_result

subgroup_results
temporal_results

verdict
reviewer
review_date
```

Allowed verdicts:

```text
APPROVE
REVISE
DEFER
REJECT
```

---

# PART H — VERSIONING AND RELEASE

## 30. Approved lifecycle release

Lifecycle becomes production-eligible only after:

```text
LifecycleSpec
        +
Lifecycle Replay Validation
        +
Human Review
        +
Synthetic Regression Pass
        ↓
Approved LifecycleSpec v1
```

It is then referenced by the top-level SignalDefinition.

---

## 31. Change management

Lifecycle parameter changes require new versioning.

Example:

```text
LIFE_014_v1
→ proposed LIFE_014_v2
→ historical replay
→ added / removed / merged / split episode analysis
→ human review
→ shadow run
→ approval
→ release
```

Never silently overwrite v1.

---

# PART I — DESIGN PRINCIPLES

## 32. Key rules

1. **Detector firing is not an episode.**
2. **Repeated firing usually updates one episode.**
3. **Lifecycle is Signal-family specific.**
4. **Activation and exit rules may differ.**
5. **Missing data must not silently mean resolution.**
6. **Cooldown must not hide genuinely new events.**
7. **Replay must reconstruct episode history from factual observations.**
8. **Lifecycle parameters require provenance.**
9. **Synthetic fixtures validate mechanics, not real-world truth.**
10. **Lifecycle cannot infer Need / Opportunity.**

---

## 33. Final summary

Lifecycle sits between numeric detection and governed Signal consumption:

```text
NUMERIC DETECTION STREAM
        ↓
LifecycleSpec
        ↓
Episode Construction
        ↓
Historical Replay Validation
        ↓
Human Review
        ↓
Approved LifecycleSpec
        ↓
Production Signal Episodes
```

The lifecycle layer exists to ensure that Signal outputs are stable, deduplicated, time-bounded, explainable and replayable — not merely a collection of daily anomaly scores.

