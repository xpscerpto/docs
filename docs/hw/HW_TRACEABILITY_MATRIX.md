# XPScerpto HW Traceability Matrix

**Document status:** Normative engineering documentation  
**Document type:** Cross-document traceability, function-to-contract mapping, runtime-artifact mapping, test coverage matrix, and claim-boundary index  
**Recommended path:** `docs/hw/HW_TRACEABILITY_MATRIX.md`  
**Applies to:** XPScerpto `hw` documentation set, authority contracts, intelligence models, runtime artifacts, power/thermal evidence, FCC custody, validation matrix, and downstream integrations  
**Non-goal:** This document does not claim `PRODUCTION_READY`, `SANITIZER_CLEAN`, `FULL_GRAPH_ACCEPTED`, market readiness, implementation closure, or runtime test completion by itself.

---

## 1. Purpose

This document provides the traceability matrix for the XPScerpto `hw` documentation set.

The purpose of the matrix is to ensure that every major HW function is traceable to:

```text
1. a canonical owner document,
2. a runtime artifact or contract,
3. an evidence or custody record,
4. required positive tests,
5. required negative or adversarial tests,
6. blocked false claims,
7. downstream integration boundaries.
```

Central rule:

```text
Every HW function must have a document owner.
Every document owner must map to runtime artifacts.
Every runtime artifact must map to validation.
Every validation path must preserve claim boundaries.
```

---

## 2. Traceability Axes

Every HW requirement should be traceable across these axes:

```text
Function
Canonical document
Runtime artifact
Evidence record
Positive validation
Negative validation
Blocked claims
Downstream boundary
```

A requirement that cannot be traced across these axes is incomplete.

---

## 3. Canonical Document Ownership Matrix

| Area | Canonical Document | Owns |
|---|---|---|
| Documentation root | `docs/hw/README.md` | Reading order and documentation package boundary |
| Documentation ownership | `docs/hw/HW_DOCUMENTATION_MAP.md` | Canonical ownership and anti-duplication rules |
| Functional scope | `docs/hw/HW_FUNCTIONAL_SCOPE.md` | HW purpose, responsibilities, and non-goals |
| Authority | `docs/hw/authority/HW_AUTHORITY_MODEL.md` | Platform observes, HW evaluates, downstream obeys, FCC judges |
| Platform input | `docs/hw/authority/HW_PLATFORM_INPUT_CONTRACT.md` | Fact admission and source/timestamp/unit/run binding |
| Boundary policy | `docs/hw/authority/HW_BOUNDARY_AND_BYPASS_POLICY.md` | Forbidden local hardware inference |
| Degraded/fail-closed | `docs/hw/authority/HW_DEGRADED_MODE_AND_FAIL_CLOSED_MODEL.md` | Degraded execution and fail-closed rules |
| Intelligence scope | `docs/hw/intelligence/HW_INTELLIGENCE_SCOPE.md` | Deterministic HW intelligence definition |
| Signal intelligence | `docs/hw/intelligence/HW_SIGNAL_INTELLIGENCE_MODEL.md` | Signal-to-evidence admission |
| Sensor trust | `docs/hw/intelligence/HW_SENSOR_TRUST_AND_CONFIDENCE_MODEL.md` | Scoped confidence and trust |
| Capability authorization | `docs/hw/intelligence/HW_CAPABILITY_AUTHORIZATION_MODEL.md` | CAN vs MAY |
| Pressure intelligence | `docs/hw/intelligence/HW_PRESSURE_INTELLIGENCE_MODEL.md` | Pressure classes and execution cost |
| Stability intelligence | `docs/hw/intelligence/HW_STABILITY_INTELLIGENCE_MODEL.md` | Baseline/load/recovery stability |
| Snapshot | `docs/hw/runtime/HW_SNAPSHOT_MODEL.md` | Evaluated context |
| Decision capsule | `docs/hw/runtime/HW_DECISION_CAPSULE_CONTRACT.md` | Execution authority |
| Routing envelope | `docs/hw/runtime/HW_ROUTING_ENVELOPE_CONTRACT.md` | Downstream execution boundary |
| Power/thermal authority | `docs/hw/power_thermal/HW_POWER_AND_THERMAL_AUTHORITY.md` | Energy, power, heat, frequency, throttling authority |
| Measurement/stability | `docs/hw/power_thermal/HW_POWER_THERMAL_MEASUREMENT_AND_STABILITY_CONTRACT.md` | Measurement windows and physical stability |
| Evidence/custody | `docs/hw/evidence/HW_EVIDENCE_AND_CUSTODY_CONTRACT.md` | Evidence, custody, claim decisions, rejection records |
| Validation | `docs/hw/validation/HW_TEST_AND_VALIDATION_MATRIX.md` | Positive, negative, adversarial, and overclaim tests |
| Integration | `docs/hw/integration/HW_INTEGRATION_CONTRACTS.md` | Platform/SIMD/FHE/PQC/runtime/tests/FCC boundaries |

---

## 4. Function-to-Artifact Matrix

| HW Function | Owner Document | Runtime Artifact | Evidence Artifact | Required Validation |
|---|---|---|---|---|
| Accept platform facts | `HW_PLATFORM_INPUT_CONTRACT.md` | `Platform*Frame`, `HwTelemetryAcceptanceRecord` | input acceptance record | valid/invalid frame tests |
| Reject invalid input | `HW_PLATFORM_INPUT_CONTRACT.md` | `HwRejectionRecord` | rejection record | stale/missing/unit/source rejection |
| Interpret signals | `HW_SIGNAL_INTELLIGENCE_MODEL.md` | `HwSignalInterpretationRecord` | signal evidence or rejection | fixture/replay/UI/final_flags rejection |
| Classify sensor trust | `HW_SENSOR_TRUST_AND_CONFIDENCE_MODEL.md` | `HwSensorConfidenceRecord` | confidence evidence | trusted/degraded/unavailable/rejected tests |
| Interpret capability facts | `HW_CAPABILITY_AUTHORIZATION_MODEL.md` | `HwCapabilityInterpretationRecord` | capability evidence | CAN-not-MAY tests |
| Authorize capability use | `HW_CAPABILITY_AUTHORIZATION_MODEL.md` | `HwCapabilityAuthorizationRecord` | authorization evidence | vector/thread/timer authorization tests |
| Classify pressure | `HW_PRESSURE_INTELLIGENCE_MODEL.md` | `HwPressureInterpretationRecord` | pressure evidence | thermal/power/frequency/memory pressure tests |
| Evaluate stability | `HW_STABILITY_INTELLIGENCE_MODEL.md` | `HwStabilityRecord` | stability evidence | baseline/load/recovery/frequency tests |
| Create snapshot | `HW_SNAPSHOT_MODEL.md` | `HwSnapshot` | snapshot evidence | snapshot creation/invalidation tests |
| Emit decision | `HW_DECISION_CAPSULE_CONTRACT.md` | `HwDecisionCapsule` | decision evidence | allow/limit/degrade/reject/fail-closed tests |
| Emit route | `HW_ROUTING_ENVELOPE_CONTRACT.md` | `HwRoutingEnvelope` | route evidence | downstream narrowing/widening tests |
| Evaluate physical authority | `HW_POWER_AND_THERMAL_AUTHORITY.md` | physical decision capsule | physical authority evidence | missing sensor/throttle/frequency tests |
| Measure stability | `HW_POWER_THERMAL_MEASUREMENT_AND_STABILITY_CONTRACT.md` | measurement windows | baseline/load/recovery evidence | sample/window/derivation tests |
| Record custody | `HW_EVIDENCE_AND_CUSTODY_CONTRACT.md` | custody ledger event | custody evidence | evidence refs/rejection persistence tests |
| Validate rules | `HW_TEST_AND_VALIDATION_MATRIX.md` | validation report | test evidence | positive/negative/overclaim matrix |
| Integrate downstream | `HW_INTEGRATION_CONTRACTS.md` | integration record | integration evidence | platform/SIMD/FHE/PQC/FCC tests |

---

## 5. Requirement-to-Test Matrix

| Requirement | Positive Test | Negative Test | Expected Rejection |
|---|---|---|---|
| Platform provides facts, not claims | valid fact frame accepted | platform injects `SYSTEM_STABLE=YES` | `REJECTED_WITH_REASON_CLAIM_INJECTION` |
| HW consumes authorized input only | authorized provider accepted | unknown provider frame | `REJECTED_WITH_REASON_SOURCE_UNKNOWN` |
| Signal is not truth | accepted signal becomes evidence only | raw signal used as authority | `REJECTED_WITH_REASON_SIGNAL_NOT_AUTHORITY` |
| Sensor trust is scoped | thermal trusted for current state | same reading used for long-run stability | `REJECTED_WITH_REASON_SCOPE_MISMATCH` |
| Capability is not permission | capability fact recorded | `CPU_CAN_AVX512` authorizes route | `REJECTED_WITH_REASON_CAPABILITY_NOT_AUTHORIZATION` |
| Pressure limits execution | high pressure limits route | capability overrides pressure | `REJECTED_WITH_REASON_PRESSURE_IGNORED` |
| Stability requires windows | load window supports scoped stability | single sample proves stability | `REJECTED_WITH_REASON_WINDOW_MISSING` |
| Snapshot is not permission | snapshot created | downstream routes from snapshot | `REJECTED_WITH_REASON_SNAPSHOT_NOT_AUTHORITY` |
| Capsule starts authority | capsule authorizes scoped workload | execution without capsule | `REJECTED_WITH_REASON_CAPSULE_MISSING` |
| Envelope bounds downstream | downstream chooses narrower path | downstream widens vector/parallelism | `REJECTED_WITH_REASON_DOWNSTREAM_WIDENING_ATTEMPT` |
| Evidence requires custody | claim has evidence refs | claim accepted without refs | `REJECTED_WITH_REASON_EVIDENCE_MISSING` |
| Rejections persist | rejection record emitted | later green run erases failure | `REJECTED_WITH_REASON_FAILURE_ERASE_ATTEMPT` |
| Integration preserves authority | SIMD/FHE/PQC consume envelopes | local hardware inference | `REJECTED_WITH_REASON_AUTHORITY_BYPASS` |

---

## 6. False-Claim Rejection Matrix

| False Claim | Owner Document | Required Rejection |
|---|---|---|
| `SIGNAL_EXISTS_THEREFORE_TRUE` | `HW_SIGNAL_INTELLIGENCE_MODEL.md` | signal not admitted as evidence |
| `SENSOR_EXISTS_THEREFORE_TRUSTED` | `HW_SENSOR_TRUST_AND_CONFIDENCE_MODEL.md` | scoped confidence required |
| `CAPABILITY_EXISTS_THEREFORE_ALLOWED` | `HW_CAPABILITY_AUTHORIZATION_MODEL.md` | authorization record required |
| `SNAPSHOT_EXISTS_THEREFORE_EXECUTION_ALLOWED` | `HW_SNAPSHOT_MODEL.md` | decision capsule required |
| `CAPSULE_ALLOW_LIMITED_THEREFORE_FULL_ALLOW` | `HW_DECISION_CAPSULE_CONTRACT.md` | degraded/limited scope preserved |
| `ENVELOPE_LIMITED_THEREFORE_FULL_ROUTE` | `HW_ROUTING_ENVELOPE_CONTRACT.md` | envelope widening rejected |
| `POWER_SAMPLE_THEREFORE_POWER_STABLE` | `HW_POWER_THERMAL_MEASUREMENT_AND_STABILITY_CONTRACT.md` | power window required |
| `NORMAL_TEMPERATURE_THEREFORE_NO_THROTTLING` | `HW_POWER_AND_THERMAL_AUTHORITY.md` | frequency/throttle evidence required |
| `BUILD_PRESSURE_THEREFORE_PRODUCTION_READY` | `HW_TEST_AND_VALIDATION_MATRIX.md` | production gate required |
| `SANITIZER_PRESSURE_THEREFORE_SANITIZER_CLEAN` | `HW_EVIDENCE_AND_CUSTODY_CONTRACT.md` | sanitizer runtime gate required |
| `FULL_CTEST_PHYSICAL_STABILITY_THEREFORE_FULL_GRAPH_ACCEPTED` | `HW_TEST_AND_VALIDATION_MATRIX.md` | full graph correctness gate required |
| `UI_GREEN_THEREFORE_TRUTH` | `HW_EVIDENCE_AND_CUSTODY_CONTRACT.md` | UI not evidence |
| `FINAL_FLAGS_THEREFORE_TRUTH` | `HW_EVIDENCE_AND_CUSTODY_CONTRACT.md` | final_flags not truth |
| `PACKAGED_LOG_THEREFORE_FRESH_RUNTIME` | `HW_EVIDENCE_AND_CUSTODY_CONTRACT.md` | packaged evidence not fresh execution |
| `FIXTURE_THEREFORE_LIVE_PROOF` | `HW_SIGNAL_INTELLIGENCE_MODEL.md` | fixture/liveness boundary preserved |
| `REPLAY_THEREFORE_FRESH_EXECUTION` | `HW_EVIDENCE_AND_CUSTODY_CONTRACT.md` | replay/fresh boundary preserved |

---

## 7. Runtime Artifact Dependency Chain

The runtime artifacts must appear in this order:

```text
Platform*Frame
    ↓
HwTelemetryAcceptanceRecord / HwRejectionRecord
    ↓
HwSignalInterpretationRecord
    ↓
HwSensorConfidenceRecord
    ↓
HwCapabilityAuthorizationRecord / HwPressureRecord / HwStabilityRecord
    ↓
HwSnapshot
    ↓
HwDecisionCapsule
    ↓
HwRoutingEnvelope
    ↓
DownstreamExecutionRecord
    ↓
HwEvidenceRecord / FccClaimDecisionRecord
```

Forbidden shortcuts:

```text
Platform*Frame -> downstream execution
Signal -> claim
Capability fact -> routing
Snapshot -> execution
Decision capsule -> production readiness
Routing envelope -> full graph acceptance
```

---

## 8. Downstream Traceability Matrix

| Downstream Unit | Consumes | Must Not Do | Required Tests |
|---|---|---|---|
| Platform | observation boundary | emit final HW claims | claim-injection rejection |
| HW | accepted platform facts | directly scrape OS sensors outside platform boundary | platform boundary tests |
| SIMD | SIMD routing envelope | local CPUID/macros as authority | vector-limit and bypass tests |
| FHE | FHE workload envelope | local sensor/probing or targeted-test promotion | long-run/stress rejection tests |
| PQC | PQC workload envelope | local batch safety inference | batch/stress limit tests |
| Runtime | decision and route display/enforcement | UI truth creation | UI-not-truth tests |
| Tests | fixture/live/replay/stress contexts | promote fixture or targeted evidence | evidence-context tests |
| FCC | evidence and claim records | accept final_flags/UI/evidence-index-only truth | custody and claim rejection tests |
| Docs | normative claim boundaries | claim implementation closure without runtime evidence | docs overclaim review |

---

## 9. Evidence Traceability Matrix

| Evidence Type | Required Fields | May Support | Must Not Support Alone |
|---|---|---|---|
| Platform frame | source, timestamp, unit, run_id | input acceptance | execution permission |
| Signal record | context, source, reason | signal admission/rejection | truth |
| Confidence record | scope, state, limitation | scoped trust | universal safety |
| Snapshot | evidence refs, generation, confidence | evaluated context | execution permission |
| Decision capsule | target, workload, policy, reason | execution authority | production readiness |
| Routing envelope | target limits, forbidden paths | downstream execution | full graph acceptance |
| Measurement window | samples, duration, missing ratio | scoped stability | product readiness |
| Rejection record | object, reason, impact | negative proof | success |
| Failure record | failure kind, refs, reason | diagnosis | erased history |
| Claim decision | status, reason, evidence refs | claim result | broader unscoped claims |

---

## 10. Validation Coverage Index

The validation matrix must cover:

```text
positive path
negative path
degraded path
fail-closed path
stale path
missing evidence path
fixture path
replay path
packaged evidence path
UI truth path
final_flags truth path
evidence-index-only path
downstream bypass path
overclaim path
invalidation path
```

A requirement without negative validation is incomplete.

A claim without overclaim rejection is unsafe.

A runtime artifact without invalidation tests is unsafe.

---

## 11. Traceability Acceptance Criteria

This traceability matrix is accepted only if:

```text
every HW function has a canonical owner document
every owner document maps to runtime artifacts or explicit non-runtime scope
every authority artifact maps to validation tests
every positive claim path maps to a negative rejection path
every overclaim path has a rejection owner
every downstream unit has an integration boundary
every evidence type has claim boundaries
no production/sanitizer/full-graph claim is raised by documentation alone
```

---

## 12. Traceability Rejection Criteria

The traceability model is violated if:

```text
a function has no owner document
a document defines authority without runtime artifact or explicit scope
a runtime artifact has no validation path
a positive path has no negative test
a downstream integration can widen HW authority
fixture evidence can become live proof
replay evidence can become fresh runtime
UI, final_flags, evidence index, or packaged logs can become truth
a document claims production readiness, sanitizer cleanliness, or full graph acceptance
```

---

## 13. Final Engineering Rule

The final traceability rule is:

```text
A HW rule is not complete until it is documented, represented, evidenced, validated, and protected from overclaim.
```

The final rejection rule is:

```text
Any HW claim path without an owner, artifact, evidence record, negative test, and claim boundary must be treated as incomplete.
```
