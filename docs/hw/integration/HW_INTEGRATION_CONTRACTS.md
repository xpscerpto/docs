# XPScerpto HW Integration Contracts

**Document status:** Normative engineering documentation  
**Scope:** XPScerpto `hw` unit  
**Claim boundary:** This document does not claim `PRODUCTION_READY`, `SANITIZER_CLEAN`, `FULL_GRAPH_ACCEPTED`, market readiness, or external certification.  
**Authority rule:** Platform observes. HW evaluates. Downstream units obey. FCC records and judges claims.


**Document type:** Cross-unit integration and authority consumption contract

---

## Purpose

Every integration consumes HW authority. No integration recreates HW authority locally.

## Integration Model

```text
platform provider
    ↓
platform.api
    ↓
HW input contract
    ↓
HW intelligence
    ↓
HW snapshot
    ↓
HW decision capsule
    ↓
HW routing envelope
    ↓
downstream unit execution
    ↓
FCC evidence and claim decision
```

## Platform

Platform provides facts, not final claims.

## SIMD

SIMD consumes routing envelopes and must not use local CPUID, compiler macros, or OS reads as authority.

## FHE

FHE consumes workload envelopes for long-run, bootstrap, stress, sanitizer, and high-parallelism workloads.

## PQC

PQC consumes batch/stress/long-run workload envelopes.

## Runtime

Runtime displays and enforces decisions but must not create truth from UI state.

## Tests

Tests distinguish fixture, live, replay, packaged, stress, and sanitizer evidence.

## FCC

FCC records evidence and claim decisions, rejecting UI, final_flags, evidence-index-only, packaged-log-as-fresh-runtime, fixture-as-live, and replay-as-fresh.

## Final Rule

```text
A downstream unit may narrow HW authority, but it may not widen, reinterpret, bypass, or relabel it.
```
