# XPScerpto HW Pressure Intelligence Model

**Document status:** Normative engineering documentation  
**Scope:** XPScerpto `hw` unit  
**Claim boundary:** This document does not claim `PRODUCTION_READY`, `SANITIZER_CLEAN`, `FULL_GRAPH_ACCEPTED`, market readiness, or external certification.  
**Authority rule:** Platform observes. HW evaluates. Downstream units obey. FCC records and judges claims.


**Document type:** Physical and runtime pressure classification contract

---

## Purpose

Pressure is the cost that execution imposes on the physical machine and runtime environment.

## Pressure Classes

```text
thermal pressure
power pressure
frequency pressure
memory pressure
parallelism pressure
vector pressure
duration pressure
sanitizer pressure
workload pressure
evidence pressure
```

## Pressure Levels

```text
UNKNOWN
LOW
MODERATE
HIGH
SEVERE
CRITICAL
```

## Recommended Actions

```text
ALLOW
ALLOW_WITH_MONITORING
ALLOW_LIMITED
REDUCE_PARALLELISM
REDUCE_VECTOR_WIDTH
SHORTEN_DURATION
REQUIRE_ADDITIONAL_EVIDENCE
ENTER_DEGRADED_MODE
REJECT_WORKLOAD
FAIL_CLOSED
```

## Rejection Rules

Reject unexplained pressure scores, missing pressure evidence as safety, critical pressure ignored, capability overriding pressure, short-run pressure promoted to long-run stability, and sanitizer pressure promoted to sanitizer clean.

## Final Rule

```text
No heavy workload, wide vector path, high parallelism profile, sanitizer run, or long-run stress may proceed unless HW pressure intelligence has classified the pressure and HW authority has emitted a bounded decision.
```
