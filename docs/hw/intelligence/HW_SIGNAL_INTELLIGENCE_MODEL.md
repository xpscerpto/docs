# XPScerpto HW Signal Intelligence Model

**Document status:** Normative engineering documentation  
**Scope:** XPScerpto `hw` unit  
**Claim boundary:** This document does not claim `PRODUCTION_READY`, `SANITIZER_CLEAN`, `FULL_GRAPH_ACCEPTED`, market readiness, or external certification.  
**Authority rule:** Platform observes. HW evaluates. Downstream units obey. FCC records and judges claims.


**Document type:** Signal interpretation and evidence-admission contract

---

## Purpose

A signal is not truth. A signal is an accused observation until HW admits or rejects it.

## Lifecycle

```text
raw signal
    ↓
source-bound
    ↓
schema-bound
    ↓
unit-bound
    ↓
time-bound
    ↓
run-bound
    ↓
plausibility-checked
    ↓
context-classified
    ↓
contradiction-checked
    ↓
accepted evidence or rejected signal
```

## Signal Contexts

```text
LIVE_RUNTIME
TEST_FIXTURE
FORENSIC_REPLAY
PACKAGED_EVIDENCE
EXTERNAL_LAB
UI_DISPLAY
UNKNOWN_CONTEXT
```

## Rejection Rules

Reject source unknown, unit unknown, stale timestamp, run_id mismatch, impossible value, fixture promoted to live, replay promoted to fresh, UI as evidence, final_flags as truth, packaged logs as fresh runtime.

## Final Rule

```text
A signal is not truth.
A signal is not permission.
A signal is not a claim.
```
