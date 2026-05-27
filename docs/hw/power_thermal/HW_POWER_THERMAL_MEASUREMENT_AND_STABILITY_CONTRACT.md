# XPScerpto HW Power/Thermal Measurement and Stability Contract

**Document status:** Normative engineering documentation  
**Scope:** XPScerpto `hw` unit  
**Claim boundary:** This document does not claim `PRODUCTION_READY`, `SANITIZER_CLEAN`, `FULL_GRAPH_ACCEPTED`, market readiness, or external certification.  
**Authority rule:** Platform observes. HW evaluates. Downstream units obey. FCC records and judges claims.


**Document type:** Measurement, windowing, and stability evidence contract

---

## Purpose

Measurement is not stability. Stability is measured behavior over time under a scoped workload.

## Measurement Rules

```text
energy != direct power
power = energy_delta / time_delta only with valid units, timestamps, counter identity, and monotonicity or wrap proof
temperature requires source, unit, timestamp, confidence, and workload context
frequency evidence is required for throttling and stability claims
```

## Required Windows

```text
idle baseline
load window
recovery window when required
power window
thermal window
frequency window
```

## Rejection Rules

Reject unitless physical values, stale samples, single-sample stability, completed-run stability, missing-sensor safety, normal-temperature hiding frequency droop, fixture-as-live, replay-as-fresh-runtime, and build/sanitizer/CTest overclaims.

## Final Rule

```text
No physical stability claim may be made from a value.
It must be made from a scoped, sufficient, confidence-classified evidence window.
```
