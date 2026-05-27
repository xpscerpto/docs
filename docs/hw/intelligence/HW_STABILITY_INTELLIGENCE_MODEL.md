# XPScerpto HW Stability Intelligence Model

**Document status:** Normative engineering documentation  
**Scope:** XPScerpto `hw` unit  
**Claim boundary:** This document does not claim `PRODUCTION_READY`, `SANITIZER_CLEAN`, `FULL_GRAPH_ACCEPTED`, market readiness, or external certification.  
**Authority rule:** Platform observes. HW evaluates. Downstream units obey. FCC records and judges claims.


**Document type:** Stability, windows, throttling, and recovery contract

---

## Purpose

Stability is sustained, contextual, evidence-bound behavior over time.

## Required Windows

```text
idle baseline window
load window
recovery window when required
```

## Stability Evidence

```text
sample count
window duration
missing sample ratio
temperature distribution
power distribution
frequency floor
frequency droop detection
throttling evidence
workload execution proof
failure status
custody linkage
```

## Important Separations

```text
fresh sample != stability
completed run != stability
short run != long-run stability
physical stability != software correctness
physical stability != production readiness
```

## Required Rejections

```text
single sample proves stability
normal temperature hides frequency droop
no-throttle claim without throttle/frequency evidence
build stability proves production ready
sanitizer pressure proves sanitizer clean
physical CTest stability proves full graph accepted
```

## Final Rule

```text
Stability is not a state the machine claims.
Stability is a scoped conclusion HW earns from evidence over time.
```
