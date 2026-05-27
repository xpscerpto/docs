# XPScerpto HW Routing Envelope Contract

**Document status:** Normative engineering documentation  
**Scope:** XPScerpto `hw` unit  
**Claim boundary:** This document does not claim `PRODUCTION_READY`, `SANITIZER_CLEAN`, `FULL_GRAPH_ACCEPTED`, market readiness, or external certification.  
**Authority rule:** Platform observes. HW evaluates. Downstream units obey. FCC records and judges claims.


**Document type:** Downstream execution-envelope contract

---

## Purpose

A routing envelope is the only downstream-consumable form of HW execution authority.

## Required Fields

```text
run_id
envelope_id
capsule_ref
target_unit
workload_class
policy_profile
authorization_result
execution_mode
limits
forbidden_paths
allowed_paths
required_fallbacks
blocked_claims
reason_code
evidence_refs
validity_window_or_invalidation_rule
```

## Unit-Specific Envelopes

```text
SIMD vector width and implementation-path envelope
FHE workload and parallelism envelope
PQC batch and stress envelope
runtime display/enforcement envelope
test fixture/live/replay envelope
sanitizer pressure envelope
FCC replay envelope
```

## Downstream Rule

A downstream unit may choose a safer path than the envelope allows, but it may never choose a broader path.

## Final Rule

```text
Any route that bypasses, widens, mutates, ignores, or overclaims the HW routing envelope must be rejected.
```
