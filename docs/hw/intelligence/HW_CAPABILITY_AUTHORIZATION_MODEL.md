# XPScerpto HW Capability Authorization Model

**Document status:** Normative engineering documentation  
**Scope:** XPScerpto `hw` unit  
**Claim boundary:** This document does not claim `PRODUCTION_READY`, `SANITIZER_CLEAN`, `FULL_GRAPH_ACCEPTED`, market readiness, or external certification.  
**Authority rule:** Platform observes. HW evaluates. Downstream units obey. FCC records and judges claims.


**Document type:** Capability interpretation and authorization contract

---

## Purpose

This document prevents the collapse of capability into permission.

## Core Rule

```text
CAN does not mean MAY.
```

## Capability Classes

```text
vector capabilities
core and thread capabilities
cache and memory capabilities
atomic and synchronization capabilities
timer and measurement capabilities
power / thermal / frequency capabilities
platform capability facts
```

## Authorization Inputs

```text
capability fact
capability confidence
source authority
policy profile
workload class
requested parallelism
requested vector width
thermal state
power state
frequency state
pressure level
degraded mode
custody requirement
```

## Forbidden Claims

```text
CAPABILITY_EXISTS_THEREFORE_ALLOWED
COMPILER_MACRO_THEREFORE_AUTHORIZED
THREAD_COUNT_HIGH_THEREFORE_FULL_PARALLELISM_ALLOWED
TIMER_AVAILABLE_THEREFORE_BENCHMARK_VALID
```

## Final Rule

```text
No downstream unit may use a hardware capability unless HW has authorized that use for the specific workload, policy, evidence state, and validity window.
```
