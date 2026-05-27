# XPScerpto HW Documentation Root

**Document status:** Normative engineering documentation  
**Scope:** XPScerpto `hw` unit  
**Claim boundary:** This document does not claim `PRODUCTION_READY`, `SANITIZER_CLEAN`, `FULL_GRAPH_ACCEPTED`, market readiness, or external certification.  
**Authority rule:** Platform observes. HW evaluates. Downstream units obey. FCC records and judges claims.


**Document type:** Documentation root and reading guide

---

## Purpose

This directory defines the HW documentation set from first principles.

The `hw` unit is not a hardware inventory helper. It is the physical intelligence and execution authority layer of XPScerpto.

## Canonical Rule

```text
platform observes
hw evaluates
downstream obeys
fcc records and judges claims
```

## Documentation Areas

```text
authority/       authority, input, boundary, degraded/fail-closed
intelligence/    signal, trust, capability, pressure, stability
runtime/         snapshot, decision capsule, routing envelope
power_thermal/   power, energy, heat, frequency, throttling, stability
evidence/        custody, claim decisions, rejection records
validation/      tests, policy scans, overclaim rejection
integration/     platform/SIMD/FHE/PQC/runtime/tests/FCC integration
```

## Reading Order

```text
1. HW_FUNCTIONAL_SCOPE.md
2. authority/HW_AUTHORITY_MODEL.md
3. authority/HW_PLATFORM_INPUT_CONTRACT.md
4. intelligence/HW_INTELLIGENCE_SCOPE.md
5. intelligence/HW_SIGNAL_INTELLIGENCE_MODEL.md
6. intelligence/HW_SENSOR_TRUST_AND_CONFIDENCE_MODEL.md
7. intelligence/HW_CAPABILITY_AUTHORIZATION_MODEL.md
8. intelligence/HW_PRESSURE_INTELLIGENCE_MODEL.md
9. intelligence/HW_STABILITY_INTELLIGENCE_MODEL.md
10. runtime/HW_SNAPSHOT_MODEL.md
11. runtime/HW_DECISION_CAPSULE_CONTRACT.md
12. runtime/HW_ROUTING_ENVELOPE_CONTRACT.md
13. power_thermal/HW_POWER_AND_THERMAL_AUTHORITY.md
14. power_thermal/HW_POWER_THERMAL_MEASUREMENT_AND_STABILITY_CONTRACT.md
15. evidence/HW_EVIDENCE_AND_CUSTODY_CONTRACT.md
16. validation/HW_TEST_AND_VALIDATION_MATRIX.md
17. integration/HW_INTEGRATION_CONTRACTS.md
```

## Final Rule

```text
No signal is truth until accepted.
No capability is permission until authorized.
No snapshot is execution permission.
No downstream route exists without a decision capsule.
No claim exists without evidence.
```
