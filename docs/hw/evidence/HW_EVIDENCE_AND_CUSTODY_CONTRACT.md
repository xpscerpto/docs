# XPScerpto HW Evidence and Custody Contract

**Document status:** Normative engineering documentation  
**Scope:** XPScerpto `hw` unit  
**Claim boundary:** This document does not claim `PRODUCTION_READY`, `SANITIZER_CLEAN`, `FULL_GRAPH_ACCEPTED`, market readiness, or external certification.  
**Authority rule:** Platform observes. HW evaluates. Downstream units obey. FCC records and judges claims.


**Document type:** Evidence, custody, rejection, and claim decision contract

---

## Purpose

No HW claim without evidence. No evidence without custody when required. No rejection without durable rejection record. No later green run may erase an earlier failure.

## Evidence Classes

```text
platform input evidence
acceptance evidence
rejection evidence
signal interpretation evidence
sensor confidence evidence
capability authorization evidence
pressure evidence
stability evidence
snapshot evidence
decision capsule evidence
routing envelope evidence
workload execution evidence
measurement window evidence
failure evidence
witness evidence
claim decision evidence
```

## Allowed Claim Statuses

```text
YES_WITH_LIVE_CUSTODY
YES_WITH_BOUNDED_EVIDENCE
DEGRADED_WITH_REASON
NO_WITH_REASON
FAILED_WITH_LIVE_CUSTODY
REJECTED_WITH_REASON
NOT_RUN_WITH_REASON
NOT_AVAILABLE_WITH_REASON
INSUFFICIENT_EVIDENCE_WITH_REASON
OUT_OF_SCOPE_WITH_REASON
```

## Forbidden Truth Sources

```text
UI_STATUS
FINAL_FLAGS
EVIDENCE_INDEX_ONLY
PACKAGED_LOG_AS_FRESH_RUNTIME
UNKNOWN_SOURCE
SELF_ATTESTATION_WITHOUT_CUSTODY
```

## Final Rule

```text
Evidence is not truth until it is scoped, sourced, custody-bound when required, reason-coded, and judged against the claim it is asked to support.
```
