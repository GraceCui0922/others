# Signal Node Research & Production Design

**Version:** v1.0  
**Scope:** Standard Chartered Individual Decision Agent — Signal Node  
**Status:** Working design for research, validation and implementation alignment  
**Primary source basis:** `Signal_Event_Change_Detection_Validation_Research_Paper_v1.0.md` and the agreed project architecture / discussion context

---

## 1. Purpose

The Signal Node converts governed factual Evidence into validated, time-stamped, numeric detections of material events, changes, deviations or imminent conditions.

A Signal is **not** a Need, Opportunity, action recommendation, response propensity or client Trait/State.

The core research direction is:

```text
FACTUAL / SEMANTIC EVIDENCE
        ↓
Candidate Signal Family Inventory
        ↓
Candidate Indicator Discovery
        ↓
Exact Indicator Formulation
        ↓
Historical Profiling / Screening
        ↓
Baseline Candidates
        ↓
Detector Candidates
        ↓
Parameter Search / Historical Replay
        ↓
Detection Validation
        ↓
Lifecycle Design + Replay Validation
        ↓
Signal-vs-State / Need Boundary Test
        ↓
Human Approval
        ↓
Approved SignalDefinition
        ↓
Production Signal Engine
```

The Signal Node contains two clearly separated branches:

1. **Research / Discovery Branch** — discovers, formulates, validates and governs Signals.
2. **Production Branch** — executes only approved Signal definitions against current point-in-time Evidence.

Production must not invent thresholds, modify definitions, self-tune lifecycle rules or infer new Signal semantics at runtime.

---

## 2. Signal maturity model

| Level | Meaning | Permitted claim |
|---|---|---|
| G0 | Factual observation | A factual event / measure exists |
| G1 | Validated change/anomaly indicator | The indicator reliably measures a defined pattern/change |
| G2 | Calibrated detector candidate | The detector can identify the candidate change under a defined specification |
| G3 | Validated Signal family | Detection validity has been established within defined boundaries |
| G4 | Operational candidate | Lifecycle, deduplication, expiry and operating rules are complete |
| G5 | Approved operational Signal | Approved for production use |

A candidate Signal must move through the evidence ladder rather than jump directly from a business idea to G5.

---

# PART A — RESEARCH / DISCOVERY BRANCH

## 3. P0 objective

### P0 is NOT “build one predefined Signal”

P0 should establish the **Candidate Signal & Indicator Discovery Factory**.

The objective is:

> Systematically traverse the full governed Semantic Evidence universe, discover candidate measurable indicators by Signal family, validate whether those indicators are technically and semantically fit for Signal research, and create a governed research queue for detector design.

P0 must not begin from names such as `Persistent Cash Accumulation` and search for supporting variables.

The direction is:

```text
Evidence → Indicator → Signal Family → Detector Research → Validated Signal
```

not:

```text
Named Signal → find convenient features
```

---

## 4. P0 input — Semantic Evidence Universe

The primitive input is the governed Semantic Evidence Layer, not raw transaction detail.

Example domains include:

- Liquidity / Fund Flow
- Deposit Balance
- Deposit Lifecycle / Maturity
- Wealth Holdings
- Wealth Transactions
- Portfolio / AUM
- Realised / Unrealised P&L
- Digital Behaviour
- RM / Relationship
- Demographics / Customer Tags
- Campaign / Bank-treatment history

Campaign targeting / bank treatment should generally be treated as contextual or confounding evidence unless explicitly justified; historical targeting is not evidence of client Need.

Each evidence item should ideally carry:

```text
semantic_measure_id
business_definition
grain
as_of_date
observed_at
known_at
value
unit
source
source_version
quality_status
currency_method
missingness_status
```

---

## 5. Candidate Signal Family Inventory

Each evidence domain should be traversed using a fixed Signal-family lens.

### Family F1 — Deterministic Event

Examples:
- maturity / expiry event
- account / product status change
- large authoritative transfer event

Question:
> Is the factual event itself intrinsically material enough to be a Signal candidate?

### Family F2 — Threshold Crossing

Examples:
- cash concentration above governed contextual boundary
- inactivity exceeding expected cadence

Question:
> Has a factual measure crossed a meaningful reference boundary?

### Family F3 — Change

Examples:
- abrupt balance change
- inflow acceleration
- engagement decline
- sudden digital behaviour shift

Question:
> Has the level, slope or rate of change materially shifted?

### Family F4 — Anomaly / Deviation

Examples:
- client-relative deviation
- peer-conditioned deviation
- unusual portfolio behaviour

Question:
> Is the current observation materially different from expected behaviour?

### Family F5 — Forecast / Imminent Event

Examples:
- calibrated near-term event risk

Question:
> Is there sufficient calibrated evidence that an important event is likely imminent?

No Signal family is assumed to be valid before empirical validation.

---

## 6. Candidate Indicator Discovery

For each Semantic Evidence measure, generate candidate indicator formulations in a structured way.

Typical formulation classes:

```text
LEVEL
CHANGE
RELATIVE_CHANGE
ROLLING_MEAN / MEDIAN
SLOPE
ACCELERATION
VOLATILITY
RATIO
SHARE
CLIENT-RELATIVE_DEVIATION
PEER-RELATIVE_DEVIATION
EVENT-RESPONSE
RETENTION / RECOVERY
```

Example from CASA Evidence:

```text
CASA level
CASA 7D / 30D absolute change
CASA relative change
CASA rolling median
CASA robust slope
CASA / AUM
CASA deviation vs own baseline
CASA deviation vs peer baseline
External inflow acceleration
Cash retention after large inflow
```

At this stage these are **candidate measurements**, not approved Signals.

---

## 7. Candidate Indicator Specification

Each candidate indicator must be represented explicitly.

Recommended fields:

```yaml
indicator_id:
name:
source_evidence:
formula:
grain:
as_of_rule:
lookback:
eligibility:
normalization:
missingness_rule:
expected_temporal_character:
candidate_signal_families:
rival_explanations:
confounders:
prohibited_interpretations:
parameter_provenance:
research_status:
```

The implementation agent must not invent unexplained windows, thresholds or transforms.

---

## 8. Historical Profiling / Indicator Screening

Historical profiling is used to determine whether a candidate indicator is suitable for Signal detector research.

It is **not** used simply to pick a threshold.

Required analysis includes:

- coverage / observability
- history depth
- missingness
- distribution shape / heavy tails
- temporal volatility
- seasonality
- client heterogeneity
- subgroup heterogeneity
- data-source changes
- duplicate / impossible observations
- wealth / segment / tenure confounding
- bank-treatment contamination
- currency / FX effects
- redundancy with other indicators
- robustness to formulation/window choice

Possible indicator verdicts:

```text
RETAIN_FOR_SIGNAL_RESEARCH
RETAIN_AS_CONTEXT
RETAIN_AS_CONTROL
STATE_EVIDENCE_CANDIDATE
DESCRIPTIVE_ONLY
REVISE_FORMULATION
RESTRICTED_POPULATION
DEFER
REJECT
```

Output artifact:

`IndicatorProfilingReport`

---

## 9. Exact Signal Formulation

Only after indicator screening should research ask:

> What change in this indicator is sufficiently material to qualify as a Signal candidate?

Each Signal formulation must eventually define:

- factual source
- event/statistic formula
- reference baseline
- detection horizon
- minimum evidence
- threshold / decision rule
- magnitude definition
- uncertainty semantics
- persistence requirement
- first-seen / detected-at / effective-at
- validity / expiry
- cooldown / re-alert rule
- deduplication / episode rule
- parameter provenance

---

## 10. Baseline Research

Baseline candidates should be explicit and challenged.

Typical families:

```text
B0 fixed / policy threshold
B1 client historical distribution
B2 seasonal client baseline
B3 matched peer baseline
B4 model-predicted expected value
B5 State-conditioned baseline (only after explicit justification)
```

Baseline selection should consider:

- leakage
- subgroup bias
- temporal stability
- data sufficiency
- interpretability
- sensitivity to temporary events

The first implementation should generally prefer transparent client-history or simple reference baselines before more complex expected-value models.

---

## 11. Detector Candidate Universe

Candidate detector families may include:

```text
D0 authoritative deterministic rule
D1 static threshold benchmark
D2 robust deviation detector
D3 sequential detector (EWMA / CUSUM)
D4 change-point detector
D5 forecast-residual detector
D6 multivariate anomaly detector
```

No universal detector is assumed to be best.

---

## 12. Parameter Registry

All unresolved parameters should be explicitly registered.

Example:

```yaml
baseline_lookback:
  value: null
  provenance: CALIBRATION_REQUIRED

magnitude_threshold:
  value: null
  provenance: CALIBRATION_REQUIRED

persistence:
  value: null
  provenance: CALIBRATION_REQUIRED

minimum_history:
  value: null
  provenance: CALIBRATION_REQUIRED

exit_threshold:
  value: null
  provenance: CALIBRATION_REQUIRED

cooldown:
  value: null
  provenance: CALIBRATION_REQUIRED
```

Allowed provenance classes should include at least:

```text
EMPIRICALLY_ESTIMATED
EXTERNAL_RESEARCH
DOMAIN_EXPERT_ELICITED
POLICY_DEFINED
METHOD_DEFAULT
NULL_BASELINE
SENSITIVITY_PARAMETER
CALIBRATION_REQUIRED
UNKNOWN
```

---

## 13. Historical Replay

Replay runs the complete candidate formulation over historical point-in-time Evidence.

The candidate search space may include:

- alternative indicators
- alternative windows
- alternative baselines
- alternative detector families
- alternative threshold values
- alternative persistence values
- minimum-history rules

Candidate ranges and selection logic should be frozen before confirmatory validation.

Primary artifact:

`SignalFitRun`

---

## 14. Numeric Signal Output

The core Signal output should be numeric, not only binary.

Generic form:

```text
G(i,t) = detector(X(i,≤t), Baseline(i,t); parameters)
```

Recommended output fields:

```text
client_id
as_of_date
signal_family
signal_value
signal_value_type
magnitude
uncertainty
persistence
baseline_value
threshold_distance
detector_version
indicator_version
```

Examples of `signal_value_type`:

```text
ROBUST_Z
RATIO
DISTANCE
CHANGE_SCORE
PROBABILITY  # only if calibrated probability semantics are validated
```

Do not force all Signals onto one arbitrary 0–100 scale.

Binary / categorical states such as `ACTIVE` are lifecycle statuses layered on top of the numeric detector output.

---

# PART B — VALIDATION & GOVERNANCE

## 15. Detection Validation

Validation must assess whether the detector can reliably identify the intended factual change.

Minimum dimensions:

- detection delay
- false-alert / false-discovery burden
- sensitivity / recall where observable
- perturbation stability
- temporal robustness
- subgroup robustness
- source-system robustness
- interpretability
- evidence traceability
- missingness robustness

Where labels are weak, use:

- structured expert review
- negative controls
- synthetic perturbation tests
- future factual confirmation

Historical campaign action / acceptance must not be treated as ground truth.

---

## 16. Negative Controls

Each Signal candidate should define events/patterns that should **not** trigger it.

Examples:

- FX-only valuation movement
- internal transfer with no true relationship inflow
- known source-system change
- stable large balance with no new material change
- missing feed interpreted as zero

Failure on negative controls indicates formulation or baseline problems.

---

## 17. Synthetic Perturbation Test Factory

Synthetic data may be used to test detector mechanics and lifecycle implementation.

Permitted use:

- positive/negative mechanical scenarios
- boundary conditions
- missingness
- duplicate observations
- source corrections
- cooldown / reopen mechanics
- parameter sensitivity

Prohibited use:

- generator-created “true Signal” labels used as evidence of bank-real validity
- synthetic Opportunity / Need logic used to validate Signal semantics

Synthetic fixtures test **implementation correctness**, not real-world business validity.

---

## 18. Lifecycle Design + Lifecycle Replay Validation

Numeric detector firings must be converted into replayable Signal episodes.

Lifecycle must explicitly define:

- onset
- persistence
- active status
- resolution
- expiry
- cooldown
- reopen
- deduplication
- missing-data behaviour

Detailed design is specified in the separate document:

`Signal_Lifecycle_Design_and_Replay_Validation_v1.0.md`

---

## 19. Signal-vs-State Boundary Test

A Signal is a material event/change detection.

A State is a temporally coherent current condition.

A persistent elevated series should not remain an endlessly firing Signal if it is better represented as a sustained State condition.

Example logic:

```text
Material change detected
        ↓
Signal episode
        ↓
Persistent multivariate condition continues
        ↓
State layer independently assesses regime / condition
```

One Signal firing must not mechanically define State membership.

---

## 20. Downstream Need Evidence Test

A validated Signal may support one or more Need hypotheses, but must not mechanically create a Need.

The downstream test should ask:

> Does this Signal provide incremental, traceable evidence for Need inference beyond current facts / State alone?

Do NOT use:

- conversion rate alone
- historical targeting alone
- historical product purchase alone

as proof that the Signal is valid.

---

## 21. Human Approval

Human approval is primarily a **methodology / definition approval**, not approval of every client Signal instance.

The reviewer should assess:

1. Signal semantic meaning
2. Evidence factuality / PIT correctness
3. indicator formula
4. baseline choice
5. detector choice
6. parameter provenance
7. false-alert / sensitivity behaviour
8. subgroup / temporal robustness
9. missing / quality handling
10. lifecycle / expiry / cooldown / reopen
11. numeric output semantics
12. Signal-vs-State / Need boundary
13. expected production volume / operational burden

Possible decisions:

```text
APPROVE
REVISE
SHADOW_ONLY
DEFER
REJECT
```

---

## 22. Approved SignalDefinition

The approved definition should be modular rather than one large opaque object.

Recommended structure:

```text
SignalDefinition
    ├── IndicatorSpec
    ├── BaselineSpec
    ├── DetectorSpec
    ├── ParameterSet
    ├── LifecycleSpec
    ├── QualitySpec
    └── ValidationRelease
```

Example:

```yaml
signal_definition_id: SIGNAL_xxx
signal_name: ...
version: 1.0
status: APPROVED

indicator_spec: IND_xxx_v2
baseline_spec: BASE_xxx_v1
detector_spec: DET_xxx_v3
parameter_set: PARAM_xxx_v1
lifecycle_spec: LIFE_xxx_v1
quality_spec: QUAL_xxx_v1
validation_release: VAL_xxx_2026Q4
```

Version changes must never overwrite history.

---

# PART C — PRODUCTION BRANCH

## 23. Production Principle

Production Signal Engine should be deliberately constrained.

It executes only approved specifications.

Production must not:

- invent new Signal semantics
- change threshold dynamically without governance
- self-tune lifecycle rules
- allow an LLM to decide whether a client “has a Signal” from raw JSON
- infer Need or Opportunity inside the Signal runtime

---

## 24. Production Flow

```text
Current PIT Evidence
        +
Approved SignalDefinition
        ↓
Indicator Calculation
        ↓
Baseline Calculation
        ↓
Detector Execution
        ↓
Numeric Detection Statistic
        ↓
Quality / Activation Gate
        ↓
Episode / Lifecycle Engine
        ↓
Governed Signal Episode
        ↓
Publish to State / Need consumers
```

---

## 25. Production Signal Output Contract

Recommended fields:

```text
client_id
as_of_date

signal_definition_id
signal_family
signal_name
signal_value
signal_value_type
magnitude
uncertainty
quality_status

baseline_value
threshold_distance
persistence

episode_id
episode_status
onset_at
detected_at
last_seen_at
peak_value
episode_age
resolved_at
expiry_at
cooldown_until

evidence_refs
indicator_version
baseline_version
detector_version
parameter_version
lifecycle_version
signal_definition_version
validation_release_id
reason_codes
```

The numeric statistic is the analytical core. `ACTIVE`, `DECAYING`, `RESOLVED` etc. are lifecycle states.

---

## 26. Production Monitoring

Monitor at minimum:

- Signal volume drift
- signal-value distribution drift
- activation rate
- active episode count
- new episode count
- episode duration
- reopen rate
- duplicate rate
- stale-active rate
- data-quality failure rate
- subgroup concentration
- source-system drift
- sampled false-alert rate
- downstream rejection / contradiction patterns

Monitoring must not directly auto-modify production parameters.

---

## 27. Change-Control Loop

When monitoring identifies a potential issue:

```text
Production Monitoring
        ↓
Change Proposal
        ↓
Research Challenger v2
        ↓
Historical Replay
        ↓
Validation
        ↓
Human Approval
        ↓
Shadow Run
        ↓
New Approved Version
        ↓
Production Release
```

Example prohibited pattern:

```text
Production volume rises
→ system automatically raises threshold
```

Threshold / detector / lifecycle changes require governed versioning and replay validation.

---

# PART D — AGENT / ENGINE RESPONSIBILITY SPLIT

## 28. LLM responsibilities

Appropriate uses:

- evidence / source interpretation support
- candidate indicator proposal
- candidate Signal-family proposal
- rival explanation generation
- confounder identification
- detector / baseline challenger proposal
- validation-summary generation
- false-alert challenge
- human-review dossier preparation

LLM outputs are proposals / summaries unless separately governed.

---

## 29. Statistical / deterministic engine responsibilities

Must own:

- indicator calculation
- historical profiling
- baseline calculation
- detector calculation
- parameter replay
- stability metrics
- subgroup metrics
- false-alert statistics
- lifecycle replay
- synthetic regression execution
- numeric Signal production

All calculations must be reproducible and versioned.

---

## 30. Human responsibilities

Humans approve:

- semantic definition
- permitted interpretation
- detector / baseline release
- parameter provenance acceptance
- lifecycle semantics
- validation adequacy
- operational use
- version release

Human review should focus on methodology, sampled difficult cases, near-boundary cases, novel cohorts and drift — not every production Signal instance.

---

# PART E — REQUIRED RESEARCH ARTIFACTS

## 31. Minimum artifact set

```text
SignalResearchQuestion
EvidenceInputManifest
CandidateIndicatorRegister
IndicatorSpecification
IndicatorProfilingReport
CandidateSignalFamilyRecord
BaselineSpecCandidate
DetectorSpecCandidate
ParameterRegistry
ReplayProtocol
SignalFitRun
SignalValidationResult
NegativeControlResult
SyntheticPerturbationResult
LifecycleSpec
LifecycleValidationResult
SignalDecisionRecord
ApprovedSignalDefinition
ProductionSignalEpisode
SignalMonitoringResult
ChangeProposal
```

---

## 32. Recommended repository structure

```text
signal/
├── registry/
│   ├── candidate_indicator_register/
│   ├── candidate_signal_family_register/
│   └── approved_signal_definitions/
│
├── research/
│   └── <signal_family_id>/
│       ├── research_question.md
│       ├── evidence_input_manifest.yaml
│       ├── indicator_specs/
│       ├── profiling/
│       ├── baseline_candidates/
│       ├── detector_candidates/
│       ├── parameter_registry.yaml
│       ├── replay_runs/
│       ├── validation/
│       ├── lifecycle/
│       ├── downstream_test/
│       └── decision_record.md
│
├── approved/
│   └── <signal_definition_id>/
│       ├── signal_definition.yaml
│       ├── indicator_spec.yaml
│       ├── baseline_spec.yaml
│       ├── detector_spec.yaml
│       ├── parameter_set.yaml
│       ├── lifecycle_spec.yaml
│       ├── quality_spec.yaml
│       └── validation_release.yaml
│
└── monitoring/
    ├── production_metrics/
    └── change_proposals/
```

---

## 33. Final design summary

The Signal Node should operate as three governed stages:

### 1. Discover

```text
Semantic Evidence
→ Candidate Families
→ Candidate Indicators
→ Indicator Validation
```

### 2. Validate & Govern

```text
Baseline
→ Detector
→ Replay
→ Detection Validation
→ Lifecycle Validation
→ Human Approval
→ Approved SignalDefinition
```

### 3. Operate

```text
Approved Definition
+ Current PIT Evidence
→ Numeric Signal
→ Signal Episode
→ Production Monitoring
→ Governed Change Cycle
```

The key design principle is:

> **Facts first, numeric detection second, lifecycle third, interpretation later.**

The Signal Node detects material change. It does not decide client Need, Opportunity or action.

