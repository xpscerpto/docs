# XPScerpto HW Sensor Trust and Confidence Model

**Document status:** Normative engineering documentation  
**Scope:** XPScerpto `hw` unit  
**Claim boundary:** This document does not claim `PRODUCTION_READY`, `SANITIZER_CLEAN`, `FULL_GRAPH_ACCEPTED`, market readiness, or external certification.  
**Authority rule:** Platform observes. HW evaluates. Downstream units obey. FCC records and judges claims.


**Document type:** Sensor confidence and claim-scope contract

---

## Purpose

A sensor value is not trusted because it exists. It is trusted only for a specific purpose, context, scope, and claim.

## Confidence States

```text
TRUSTED
DEGRADED
UNAVAILABLE
REJECTED
UNKNOWN
CONTRADICTORY
STALE
UNSUPPORTED
```

## Required Trust Dimensions

```text
source_authorized
schema_known
unit_known
timestamp_fresh
run_id_matched
value_present
value_plausible
read_status_consistent
missing_fields_explained
cross_sample_consistent
cross_sensor_consistent
claim_scope_supported
```

## Scoped Trust

Correct:

```text
THERMAL_SENSOR_CONFIDENCE=TRUSTED_FOR_CURRENT_THERMAL_STATE
THERMAL_STABILITY_CONFIDENCE=INSUFFICIENT_WITH_REASON_LOAD_WINDOW_MISSING
```

Incorrect:

```text
SENSOR_TRUSTED=YES
```

without scope.

## Final Rule

```text
Missing, stale, degraded, contradictory, unsupported, or rejected evidence must narrow authority, not expand it.
```
