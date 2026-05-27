# HW Documentation Map

**Document status:** Normative engineering documentation  
**Scope:** XPScerpto `hw` unit  
**Claim boundary:** This document does not claim `PRODUCTION_READY`, `SANITIZER_CLEAN`, `FULL_GRAPH_ACCEPTED`, market readiness, or external certification.  
**Authority rule:** Platform observes. HW evaluates. Downstream units obey. FCC records and judges claims.


**Document type:** Documentation ownership and anti-duplication map

---

## Purpose

This file defines why each HW document exists and prevents documentation drift.

## Allowed Document Kinds

```text
FUNCTIONAL_SCOPE
AUTHORITY_MODEL
INPUT_CONTRACT
INTELLIGENCE_MODEL
RUNTIME_CONTRACT
MEASUREMENT_CONTRACT
EVIDENCE_CONTRACT
VALIDATION_MATRIX
INTEGRATION_CONTRACT
```

## Canonical Ownership

```text
Functional identity:
    HW_FUNCTIONAL_SCOPE.md

Authority:
    authority/HW_AUTHORITY_MODEL.md
    authority/HW_PLATFORM_INPUT_CONTRACT.md
    authority/HW_BOUNDARY_AND_BYPASS_POLICY.md
    authority/HW_DEGRADED_MODE_AND_FAIL_CLOSED_MODEL.md

Intelligence:
    intelligence/HW_INTELLIGENCE_SCOPE.md
    intelligence/HW_SIGNAL_INTELLIGENCE_MODEL.md
    intelligence/HW_SENSOR_TRUST_AND_CONFIDENCE_MODEL.md
    intelligence/HW_CAPABILITY_AUTHORIZATION_MODEL.md
    intelligence/HW_PRESSURE_INTELLIGENCE_MODEL.md
    intelligence/HW_STABILITY_INTELLIGENCE_MODEL.md

Runtime artifacts:
    runtime/HW_SNAPSHOT_MODEL.md
    runtime/HW_DECISION_CAPSULE_CONTRACT.md
    runtime/HW_ROUTING_ENVELOPE_CONTRACT.md

Power and thermal:
    power_thermal/HW_POWER_AND_THERMAL_AUTHORITY.md
    power_thermal/HW_POWER_THERMAL_MEASUREMENT_AND_STABILITY_CONTRACT.md

Evidence and validation:
    evidence/HW_EVIDENCE_AND_CUSTODY_CONTRACT.md
    validation/HW_TEST_AND_VALIDATION_MATRIX.md

Integration:
    integration/HW_INTEGRATION_CONTRACTS.md
```

## Rejection Rule

A new document is rejected if it duplicates canonical ownership, mixes status with normative rules, or raises production/sanitizer/full-graph claims.
