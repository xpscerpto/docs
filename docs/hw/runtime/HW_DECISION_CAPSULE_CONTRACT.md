# XPScerpto HW Decision Capsule Contract

**Document status:** Normative engineering documentation  
**Scope:** XPScerpto `hw` unit  
**Claim boundary:** This document does not claim `PRODUCTION_READY`, `SANITIZER_CLEAN`, `FULL_GRAPH_ACCEPTED`, market readiness, or external certification.  
**Authority rule:** Platform observes. HW evaluates. Downstream units obey. FCC records and judges claims.


**Document type:** Runtime execution-authority contract

---

## Purpose

The HW decision capsule is the first artifact that may authorize hardware-dependent execution.

## Decision Classes

```text
ALLOW
ALLOW_LIMITED
DEGRADED_ALLOW
REJECT
FAIL_CLOSED
NOT_AVAILABLE_WITH_REASON
NOT_RUN_WITH_REASON
OUT_OF_SCOPE_WITH_REASON
```

## Required Fields

```text
run_id
capsule_id
snapshot_ref
snapshot_generation
target_unit
workload_class
policy_profile
context_class
requested_mode
decision_class
decision_result
validity_window_or_invalidation_rule
constraints
degraded_mode_required
fail_closed_required
reason_code
evidence_refs
rejection_refs_if_any
blocked_claims
routing_envelope_requirements
```

## Final Rule

```text
No SIMD path, FHE workload, PQC batch, sanitizer runtime, full-test pressure, or heavy workload may execute under HW authority unless a fresh, scoped, reason-coded, evidence-bound decision capsule authorizes or limits it.
```
