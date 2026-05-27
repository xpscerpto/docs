# XPScerpto HW Snapshot Model

**Document status:** Normative engineering documentation  
**Scope:** XPScerpto `hw` unit  
**Claim boundary:** This document does not claim `PRODUCTION_READY`, `SANITIZER_CLEAN`, `FULL_GRAPH_ACCEPTED`, market readiness, or external certification.  
**Authority rule:** Platform observes. HW evaluates. Downstream units obey. FCC records and judges claims.


**Document type:** Evaluated machine-state and invalidation contract

---

## Purpose

A HW snapshot is evaluated context, not execution permission.

## Snapshot Contents

```text
snapshot_id
run_id
telemetry_generation
created_timestamp_ns
context_class
policy_profile
platform_frame_refs
acceptance_record_refs
rejection_record_refs
sensor_confidence_refs
capability_summary
power_thermal_frequency_summary
pressure_summary
stability_summary
degraded_mode_summary
known_limitations
overall_confidence_state
reason_codes
```

## Snapshot Is Not

```text
execution permission
routing envelope
production proof
sanitizer proof
full graph proof
```

## Invalidation Triggers

```text
telemetry generation changes
policy profile changes
workload class changes
sensor confidence changes
thermal/power/frequency state changes
pressure/stability state changes
host identity changes
custody context changes
```

## Final Rule

```text
No downstream execution may be authorized from a snapshot alone; execution authority must pass through a decision capsule and routing envelope.
```
