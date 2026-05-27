# HW Degraded Mode and Fail-Closed Model

**Document status:** Normative engineering documentation  
**Scope:** XPScerpto `hw` unit  
**Claim boundary:** This document does not claim `PRODUCTION_READY`, `SANITIZER_CLEAN`, `FULL_GRAPH_ACCEPTED`, market readiness, or external certification.  
**Authority rule:** Platform observes. HW evaluates. Downstream units obey. FCC records and judges claims.


**Document type:** Degraded execution and fail-closed authority contract

---

## Purpose

This document defines HW behavior when evidence is incomplete, degraded, contradictory, stale, or unsafe.

## Degraded Mode

Degraded mode is truthful limitation, not failure.

Examples:

```text
POWER_SENSOR_UNAVAILABLE
THERMAL_SENSOR_DEGRADED
FREQUENCY_SENSOR_UNKNOWN
HOST_RUNNING_ON_BATTERY
SAMPLE_COUNT_TOO_LOW
```

Allowed degraded actions:

```text
reduce parallelism
reduce vector width
allow local short run only
reject long-run stress
block full stability claims
request additional evidence
```

## Fail-Closed

HW must fail closed for unauthorized source, critical thermal state, frequency collapse, sensor contradiction, downstream bypass, claim overreach, or missing custody for required claims.

## Final Rule

```text
Unknown is not safe.
Degraded is not full success.
Fail-closed is the correct result when authority cannot be proven safely.
```
