# XPScerpto HW Functional Scope

**Document status:** Normative engineering documentation  
**Scope:** XPScerpto `hw` unit  
**Claim boundary:** This document does not claim `PRODUCTION_READY`, `SANITIZER_CLEAN`, `FULL_GRAPH_ACCEPTED`, market readiness, or external certification.  
**Authority rule:** Platform observes. HW evaluates. Downstream units obey. FCC records and judges claims.


**Document type:** Functional scope and responsibility boundary

---

## Purpose

The `hw` unit transforms authorized platform evidence into reasoned, auditable, sealed execution authority.

## Primary Functions

```text
platform input acceptance
telemetry validation
sensor trust classification
machine snapshot construction
topology and memory interpretation
capability interpretation
capability authorization
power / thermal / frequency / pressure reasoning
stability reasoning
degraded and fail-closed operation
decision capsule emission
routing envelope emission
downstream bypass prevention
FCC evidence linkage
```

## Inputs

```text
PlatformIdentityFrame
PlatformTopologyFrame
PlatformCapabilityFrame
PlatformPowerThermalFrame
PlatformFrequencyFrame
PlatformMemoryPressureFrame
RunContext
CustodyContext
PolicyProfile
WorkloadRequest
```

## Outputs

```text
HwTelemetryAcceptanceRecord
HwSensorConfidenceRecord
HwSnapshot
HwCapabilityAuthorization
HwPressureClassification
HwStabilityClassification
HwDecisionCapsule
HwRoutingEnvelope
HwRejectionRecord
HwEvidenceReference
```

## Non-Goals

HW does not claim production readiness, sanitizer cleanliness, full graph acceptance, market readiness, or external certification.

## Final Rule

```text
HW does not merely describe the machine.
HW judges whether the machine may be used for a specific execution purpose.
```
