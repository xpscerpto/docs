# XPScerpto HW Platform Input Contract

**Document status:** Normative engineering documentation  
**Scope:** XPScerpto `hw` unit  
**Claim boundary:** This document does not claim `PRODUCTION_READY`, `SANITIZER_CLEAN`, `FULL_GRAPH_ACCEPTED`, market readiness, or external certification.  
**Authority rule:** Platform observes. HW evaluates. Downstream units obey. FCC records and judges claims.


**Document type:** Platform input, telemetry admission, and fact-vs-claim contract

---

## Purpose

Platform input is evidence, not authority. Platform provides authorized facts; HW accepts, rejects, evaluates, and decides.

## Frame Classes

```text
PlatformIdentityFrame
PlatformTopologyFrame
PlatformCapabilityFrame
PlatformPowerThermalFrame
PlatformFrequencyFrame
PlatformMemoryPressureFrame
PlatformProviderStatusFrame
RunContext
WorkloadRequest
CustodyContext
```

## Common Required Fields

```text
run_id
frame_id
frame_kind
schema_version
timestamp_ns
provider_id
provider_authority
source_kind
read_status
missing_field_reasons
provider_error_reason_if_any
```

## Fact vs Claim

Allowed platform fact:

```text
cpu_temperature_c=74.2
package_energy_uj=180043921
logical_thread_count=16
```

Forbidden platform claim:

```text
SYSTEM_STABLE=YES
FHE_LONG_RUN_SAFE=YES
SIMD_AVX512_ALLOWED=YES
PRODUCTION_READY=YES
```

## Rejection Rules

Reject unknown source, missing run_id, stale timestamp, unknown unit, impossible value, final_flags, UI status, packaged-log-as-fresh-runtime, and claim injection.

## Final Rule

```text
No platform-provided fact may influence HW authority until it is source-bound, time-bound, schema-bound, unit-bound, run-bound, and acceptance-recorded.
```
