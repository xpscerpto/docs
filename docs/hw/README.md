<p align="center">
  <img src="/docs/assets/hw.svg" alt="XPScerpto — Governed Sovereign Cryptographic Infrastructure" width="100%">
</p>

<div align="center">

# XPScerpto HW Unit

## Governed Physical Intelligence and Execution Authorization Layer

**Document status:** Normative engineering documentation  
**Scope:** XPScerpto `hw` unit  
**Document type:** Documentation root and reading guide  

</div>

---

## Platform Definition

XPScerpto is a governed sovereign cryptographic infrastructure platform for sensitive systems, built around evidence-bound claims, strict authority boundaries, and fail-closed operational truth.

---

## Claim Boundary

This document does not claim `PRODUCTION_READY`, `SANITIZER_CLEAN`, `FULL_GRAPH_ACCEPTED`, market readiness, regulatory approval, or external certification.

---

## Authority Rule

```text
Platform observes.
HW evaluates.
Downstream units obey.
FCC records and judges claims.
```

This rule is mandatory.

Any implementation, test, documentation, or integration path that violates this rule is invalid.

---

## Purpose

This directory defines the HW documentation set from first principles.

The `hw` unit is not a hardware inventory helper, benchmark helper, feature detector, telemetry collector, or optimization shortcut.

The `hw` unit is the physical intelligence and execution authorization layer of XPScerpto. It evaluates platform-observed facts, assigns trust and confidence, authorizes capabilities, rejects unsafe assumptions, and emits decision capsules that downstream units must obey.

HW does not observe the operating system directly.  
HW does not probe hardware directly.  
HW does not grant permission from raw signals.  
HW does not allow downstream units to self-authorize execution routes.

---

## Canonical Rule

```text
platform observes
hw evaluates
downstream obeys
fcc records and judges claims
```

---

## Documentation Areas

```text
authority/       authority, input, boundary, degraded/fail-closed behavior
intelligence/    signal, trust, capability, pressure, stability
runtime/         snapshot, decision capsule, routing envelope
power_thermal/   power, energy, heat, frequency, throttling, stability
evidence/        custody, claim decisions, rejection records
validation/      tests, policy scans, overclaim rejection
integration/     platform/SIMD/FHE/PQC/runtime/tests/FCC integration
```

---

## Reading Order

```text
1.  HW_FUNCTIONAL_SCOPE.md
2.  authority/HW_AUTHORITY_MODEL.md
3.  authority/HW_PLATFORM_INPUT_CONTRACT.md
4.  intelligence/HW_INTELLIGENCE_SCOPE.md
5.  intelligence/HW_SIGNAL_INTELLIGENCE_MODEL.md
6.  intelligence/HW_SENSOR_TRUST_AND_CONFIDENCE_MODEL.md
7.  intelligence/HW_CAPABILITY_AUTHORIZATION_MODEL.md
8.  intelligence/HW_PRESSURE_INTELLIGENCE_MODEL.md
9.  intelligence/HW_STABILITY_INTELLIGENCE_MODEL.md
10. runtime/HW_SNAPSHOT_MODEL.md
11. runtime/HW_DECISION_CAPSULE_CONTRACT.md
12. runtime/HW_ROUTING_ENVELOPE_CONTRACT.md
13. power_thermal/HW_POWER_AND_THERMAL_AUTHORITY.md
14. power_thermal/HW_POWER_THERMAL_MEASUREMENT_AND_STABILITY_CONTRACT.md
15. evidence/HW_EVIDENCE_AND_CUSTODY_CONTRACT.md
16. validation/HW_TEST_AND_VALIDATION_MATRIX.md
17. integration/HW_INTEGRATION_CONTRACTS.md
```

---

## Enforcement Meaning

The documentation set is normative.

It defines what HW is allowed to accept, reject, authorize, downgrade, record, and expose to downstream units.

A passing implementation must prove that:

```text
raw platform facts are not treated as truth
snapshots are not treated as permission
capabilities are not enabled without authorization
degraded states fail closed where required
downstream routing requires a decision capsule
claims require evidence custody
FCC remains the final claim judge
```

---

## Non-Goals

The HW unit is not responsible for:

```text
direct OS probing
CPUID execution
compiler ISA shortcut authorization
benchmark-only routing
raw telemetry trust
downstream self-selection of execution paths
market or certification claims
production readiness claims
```

These responsibilities are either outside HW or forbidden by the authority model.

---

## Final Rule

```text
No signal is truth until accepted.
No capability is permission until authorized.
No snapshot is execution permission.
No downstream route exists without a decision capsule.
No claim exists without evidence.
No evidence becomes final without FCC judgment.
```
