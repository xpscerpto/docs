# XPScerpto HW Power and Thermal Authority Model

**Document status:** Normative engineering documentation  
**Recommended path:** `docs/hw/HW_POWER_AND_THERMAL_AUTHORITY.md`  
**Scope:** XPScerpto `hw` unit, platform telemetry boundary, downstream execution authority, FCC custody linkage  
**Non-goal:** This document does not claim `PRODUCTION_READY`, `SANITIZER_CLEAN`, or `FULL_GRAPH_ACCEPTED`.

---

## 1. Purpose

This document defines how the XPScerpto **HW** unit must handle power, thermal pressure, frequency stability, throttling, and physical execution limits.

The HW unit is not merely a hardware information reader. It is the **physical authority evaluator** of the system. Its purpose is to transform authorized telemetry facts into sealed execution decisions that downstream units must obey.

Central rule:

```text
Platform observes the machine.
HW evaluates the machine.
Downstream units obey sealed HW decisions.
```

The intended authority chain is:

```text
platform provider
    ↓
platform.api telemetry frame
    ↓
hw sensor validation
    ↓
hw power / thermal / frequency analysis
    ↓
hw decision capsule
    ↓
simd / fhe / pqc / runtime / tests
    ↓
FCC custody and claim decisions
```

The prohibited chain is:

```text
SIMD reads CPU or thermal state directly
FHE reads OS sensors directly
tests infer capabilities from compiler macros
runtime accepts UI green state as proof
final_flags becomes a truth source
packaged logs are reused as fresh runtime
```

---

## 2. Scope

This document covers:

```text
- Power telemetry
- Thermal telemetry
- Frequency stability
- Thermal throttling detection
- Power-limit throttling detection
- Sensor trust and confidence
- Degraded hardware mode
- Workload authorization
- Downstream routing decisions
- Claim discipline
- FCC custody integration
- Required test matrix
```

This document does **not** by itself prove:

```text
- PRODUCTION_READY
- SANITIZER_CLEAN
- FULL_GRAPH_ACCEPTED
- Full runtime correctness
- External laboratory-grade power measurement
- Vendor-specific fan-control correctness
- Long-term field reliability
```

HW may prove that a machine is currently safe or unsafe for a specific workload under recorded telemetry. It must not elevate that evidence into broader production claims.

---

## 3. Sovereign Boundary Rule

Only the platform layer is allowed to touch the external world.

The HW unit must not become an uncontrolled OS-probing layer. It receives authorized, filtered, timestamped facts from `platform.api`, then evaluates those facts.

Required rule:

```text
Only platform layer may observe external machine state.
HW receives authorized telemetry facts.
HW emits sealed decisions.
No downstream module may bypass HW.
```

Therefore:

```text
platform provider
    reads OS / firmware / kernel telemetry according to provider policy

platform.api
    passes authorized telemetry frames to HW

hw
    validates, classifies, evaluates, and decides

simd / fhe / pqc / runtime
    consume HW decisions only
```

HW must judge facts. It must not scrape the world independently unless the architecture explicitly defines HW as part of the platform boundary, which is not the preferred XPScerpto model.

---

## 4. Why Power and Thermal Authority Belong to HW

Power and thermal state are not local to one unit.

A SIMD copy kernel can raise package temperature through wide vector lanes. A FHE workload can sustain high CPU and memory pressure for long periods. Full CTest can mix platform, HW, SIMD, crypto, and FCC workloads. Sanitizers can amplify memory use, runtime duration, and heat.

Because the physical machine is shared, each domain must not decide independently whether it is safe to run. The decision must be centralized:

```text
HW evaluates the physical operating envelope.
HW emits the allowed execution envelope.
Downstream units obey the envelope.
```

Example:

```text
FHE requests a long bootstrap stress profile
    ↓
HW observes low thermal headroom
    ↓
HW emits:
    FHE_LONG_RUN_ALLOWED=NO_WITH_REASON_THERMAL_HEADROOM_LOW
    ↓
FHE must not run that heavy profile
```

Example:

```text
SIMD requests AVX512 execution
    ↓
HW observes high package temperature or power-limit pressure
    ↓
HW emits:
    SIMD_AVX512_ALLOWED=NO_WITH_REASON_POWER_THERMAL_PRESSURE
    ↓
SIMD must select a lower authorized path
```

---

## 5. Definitions

### 5.1 Energy

Energy is cumulative work consumed over time. It is commonly represented as:

```text
joules
microjoules
watt-hours
```

If the system provides an energy counter, HW must not treat it as direct power. Power is derived from energy over time:

```text
watts = energy_delta / time_delta
```

or:

```text
P = ΔE / Δt
```

If the energy counter decreases, HW must either handle an explicitly documented wraparound or reject the sample.

### 5.2 Power

Power is the rate of energy consumption, usually measured in watts.

Power alone is not proof of stability. A machine may consume high power and remain stable, or consume moderate power while throttling due to heat, firmware limits, battery mode, or cooling failure.

### 5.3 Thermal State

Thermal state refers to temperature behavior over time, not just one peak value.

HW must interpret temperature together with:

```text
- workload context
- duration
- frequency stability
- throttling signals
- recovery behavior
- sensor confidence
- failure timing
```

A single temperature sample is insufficient for a full thermal stability claim.

### 5.4 Throttling

Throttling means the machine reduces performance or execution capability to remain within thermal, power, current, firmware, or battery constraints.

Recognized categories:

```text
THERMAL_THROTTLING
POWER_LIMIT_THROTTLING
CURRENT_LIMIT_THROTTLING
FREQUENCY_DROOP
BATTERY_POLICY_LIMIT
UNKNOWN_THROTTLING
```

A system can pass a workload and still be physically degraded if it completed while throttled.

---

## 6. Authorized Telemetry Input

HW must consume telemetry from authorized platform surfaces. A conceptual telemetry frame may contain:

```text
PlatformTelemetryFrame {
    run_id;
    timestamp_ns;
    source_id;
    source_authority;
    os_provider;

    cpu_temperature_c;
    gpu_temperature_c;
    nvme_temperature_c;
    board_temperature_c;

    package_energy_uj;
    package_power_watts_if_direct;

    cpu_frequency_mhz;
    min_frequency_mhz;
    max_frequency_mhz;

    fan_rpm_if_available;
    battery_state_if_available;
    ac_power_state_if_available;

    thermal_throttle_signal_if_available;
    power_limit_signal_if_available;

    read_status;
    error_reason_if_any;
}
```

All fields must be treated as evidence, not as trusted truth, until validated.

---

## 7. Telemetry Acceptance Rules

HW must not accept a power or thermal reading merely because it exists.

A sample may be used only if HW verifies:

```text
- The source is authorized.
- The timestamp is fresh.
- The value is physically plausible.
- The unit is known.
- The sample is not stale.
- Missing fields have explicit reasons.
- The source is not UI/server/self-attestation.
- The sample is linked to the current run_id when under custody.
- Nearby samples do not contradict it without explanation.
```

Rejected examples:

```text
temperature = -900°C
temperature = 500°C
energy counter decreases without wrap handling
timestamp is outside the accepted freshness window
source is unknown
read_status says success but values are missing
power value exists without unit or derivation rule
```

Correct decision:

```text
HW_TELEMETRY_REJECTED_WITH_REASON_INVALID_SENSOR_VALUE
```

Incorrect decision:

```text
HW_POWER_STABLE=YES
```

---

## 8. Sensor Confidence Model

Telemetry confidence must be explicit.

Recommended confidence states:

```text
TRUSTED
DEGRADED
UNAVAILABLE
REJECTED
UNKNOWN
```

A conceptual record:

```text
SensorConfidence {
    source_authorized;
    source_fresh;
    value_plausible;
    unit_known;
    monotonic_if_counter;
    cross_sample_consistent;
    platform_error_absent;
    confidence_level;
    degraded_reason;
}
```

Examples:

```text
HW_POWER_SENSOR_CONFIDENCE=DEGRADED_WITH_REASON_ENERGY_COUNTER_AVAILABLE_BUT_NO_DIRECT_POWER_SIGNAL
```

```text
HW_POWER_TELEMETRY=NOT_AVAILABLE_WITH_REASON_PLATFORM_POWER_SENSOR_NOT_EXPOSED
```

Unavailable must never become success.

---

## 9. HW Power Model

After validating telemetry, HW should normalize power data into a time-aware model.

Conceptual sample:

```text
HwPowerSample {
    timestamp_ns;
    package_energy_uj;
    computed_package_watts;
    direct_package_watts;
    power_source;
    ac_or_battery;
    power_limit_signal;
    confidence;
}
```

Conceptual analysis window:

```text
HwPowerWindow {
    sample_count;
    duration_ms;
    watts_min;
    watts_mean;
    watts_p50;
    watts_p95;
    watts_p99;
    watts_max;
    energy_total_j;
    power_limit_events;
    missing_sample_count;
    confidence;
}
```

HW must prefer windows over isolated samples. One sample can trigger safety rejection if critical, but it cannot prove sustained stability.

---

## 10. HW Thermal Model

Thermal analysis must also be time-aware.

Conceptual sample:

```text
HwThermalSample {
    timestamp_ns;
    cpu_temp_c;
    gpu_temp_c;
    nvme_temp_c;
    board_temp_c;
    fan_rpm;
    throttle_signal;
    confidence;
}
```

Conceptual analysis window:

```text
HwThermalWindow {
    sample_count;
    duration_ms;
    cpu_temp_min;
    cpu_temp_mean;
    cpu_temp_p95;
    cpu_temp_p99;
    cpu_temp_max;
    thermal_rise_rate_c_per_min;
    thermal_throttle_events;
    cooling_recovery_observed;
    confidence;
}
```

The peak temperature is important, but p95, p99, rise rate, and recovery behavior often reveal more than a single maximum.

---

## 11. Frequency Stability Model

Power and temperature are incomplete without frequency behavior.

HW should track:

```text
cpu_frequency_mhz
minimum_frequency_under_load
expected_frequency_range
frequency_drop_percent
frequency_oscillation
sustained_frequency_floor
```

Conceptual window:

```text
HwFrequencyWindow {
    freq_min_mhz;
    freq_mean_mhz;
    freq_p95_mhz;
    freq_max_mhz;
    freq_drop_percent_from_baseline;
    sustained_floor_broken;
    oscillation_detected;
}
```

If temperature appears acceptable but frequency collapses, HW must not claim full stability.

Correct decision:

```text
HW_FREQUENCY_STABILITY=NO_WITH_REASON_FREQUENCY_DROOP_UNDER_LOAD
```

---

## 12. HW State Classification

### 12.1 Thermal State

Required states:

```text
UNKNOWN
NORMAL
WARM
HOT
THROTTLED
CRITICAL
SENSOR_UNTRUSTED
```

Meaning:

```text
UNKNOWN
    Not enough information exists.

NORMAL
    Temperature is within accepted range and no throttling was detected.

WARM
    Temperature is elevated but not yet a rejection condition.

HOT
    Temperature is high; long or heavy workloads should be limited or rejected.

THROTTLED
    Thermal throttling or frequency drop attributable to heat was observed.

CRITICAL
    Heavy execution must stop or be rejected.

SENSOR_UNTRUSTED
    Thermal data is invalid, stale, contradictory, or unauthorized.
```

### 12.2 Power State

Required states:

```text
UNKNOWN
NORMAL
ELEVATED
LIMITED
POWER_CAPPED
BATTERY_CONSTRAINED
SENSOR_UNTRUSTED
```

Meaning:

```text
UNKNOWN
    Not enough power information exists.

NORMAL
    No visible power restriction exists.

ELEVATED
    Consumption is high but stable.

LIMITED
    Power constraints appear to affect execution.

POWER_CAPPED
    A power cap is explicit or strongly inferred.

BATTERY_CONSTRAINED
    Battery policy or unplugged operation restricts allowed workloads.

SENSOR_UNTRUSTED
    Power data is invalid, stale, contradictory, or unauthorized.
```

---

## 13. HW Decision Capsule

HW must emit sealed decisions, not just measurements.

Conceptual decision capsule:

```text
HwDecisionCapsule {
    run_id;
    telemetry_generation;

    thermal_state;
    power_state;
    frequency_state;
    confidence;

    allowed_parallelism;
    allowed_vector_width_bits;

    heavy_workload_allowed;
    long_run_allowed;
    sanitizer_allowed;
    full_ctest_allowed;
    simd_heavy_path_allowed;
    fhe_heavy_path_allowed;

    degraded_mode_required;
    fail_closed_required;

    reason;
}
```

Example:

```text
thermal_state=HOT
power_state=ELEVATED
frequency_state=STABLE
allowed_parallelism=4
allowed_vector_width_bits=128
fhe_heavy_path_allowed=NO
simd_heavy_path_allowed=LIMITED
reason=THERMAL_HEADROOM_LOW
```

The decision capsule is the surface consumed by downstream modules.

---

## 14. Downstream Rule for SIMD

SIMD must not perform local hardware inference.

Allowed chain:

```text
HW Snapshot
    ↓
HW Routing Envelope
    ↓
SIMD Dispatch Plan
    ↓
SIMD Execution
```

If HW restricts vector execution due to power or heat:

```text
SIMD_AVX512_ALLOWED=NO_WITH_REASON_HW_POWER_THERMAL_POLICY
SIMD_AVX2_ALLOWED=NO_WITH_REASON_HW_POWER_THERMAL_POLICY
SIMD_128_ALLOWED=YES_WITH_HW_AUTHORITY
```

SIMD must not reopen the decision using:

```text
__builtin_cpu_supports
/proc/cpuinfo
compiler ISA macros
local CPUID probing
OS sensor reads
```

That would break the XPScerpto authority model.

---

## 15. Downstream Rule for FHE

FHE workloads may be long-running and physically heavy. FHE must request authority, not inspect the machine.

Allowed chain:

```text
FHE workload request
    ↓
HW evaluates power / thermal / frequency state
    ↓
HW emits permission or rejection
    ↓
FHE runs, limits, or refuses
```

Example decisions:

```text
FHE_BOOTSTRAP_STRESS_ALLOWED=NO_WITH_REASON_THERMAL_STATE_HOT
FHE_LONG_RUN_ALLOWED=NO_WITH_REASON_POWER_LIMIT_DETECTED
FHE_PARALLELISM_LIMIT=4
FHE_RUNTIME_ALLOWED=YES_WITH_HW_AUTHORITY
```

FHE must not claim:

```text
FHE_STRESS_STABLE=YES
```

unless supported by workload completion, HW telemetry, custody ledger, and claim decision evidence.

---

## 16. FCC Integration

FCC does not replace HW. FCC records, validates, and cross-examines claims.

HW provides:

```text
- sensor inventory
- telemetry windows
- decision capsules
- workload authorization decisions
- throttling events
- rejected samples
- degraded-mode reasons
- post-failure physical state
```

FCC records and validates these through:

```text
runtime_custody_ledger.jsonl
claim_decisions.jsonl
rejection_records.jsonl
witness_logs/
```

Any claim such as:

```text
POWER_THERMAL_STABLE
SUSTAINED_RUNTIME_COMPLETED
NO_THROTTLING_DETECTED
```

must reference:

```text
custody_refs
witness_refs
telemetry_refs
workload_refs
decision_refs
rejection_refs_if_any
```

---

## 17. Allowed Claim Statuses

The following ambiguous statuses are prohibited:

```text
ok
green
passed
accepted
assumed
trusted
verified by final_flags
verified by evidence index
verified by UI
```

Allowed statuses:

```text
YES_WITH_LIVE_CUSTODY
FAILED_WITH_LIVE_CUSTODY
NO_WITH_REASON
NOT_RUN_WITH_REASON
NOT_AVAILABLE_WITH_REASON
REJECTED_WITH_REASON
OUT_OF_SCOPE_WITH_REASON
```

Correct examples:

```text
HW_POWER_TELEMETRY=NOT_AVAILABLE_WITH_REASON_PLATFORM_POWER_SENSOR_NOT_EXPOSED
```

```text
HW_THERMAL_STABILITY=NO_WITH_REASON_THERMAL_THROTTLING_DETECTED_UNDER_FULL_CTEST
```

Incorrect examples:

```text
HW_POWER_TELEMETRY=OK
HW_THERMAL_STABILITY=GREEN
```

---

## 18. Unavailable Does Not Mean Safe

Missing telemetry must never become a success claim.

If no power sensor exists:

```text
POWER_TELEMETRY=NOT_AVAILABLE_WITH_REASON_NO_AUTHORIZED_POWER_SENSOR
```

not:

```text
POWER_STABLE=YES
```

If no thermal sensor exists:

```text
THERMAL_TELEMETRY=NOT_AVAILABLE_WITH_REASON_NO_AUTHORIZED_THERMAL_SENSOR
```

not:

```text
NO_THERMAL_ISSUES=YES
```

If telemetry is incomplete, HW may enter degraded mode. Degraded execution may be allowed for limited workloads, but full stability claims must remain blocked.

---

## 19. Degraded Mode

When readings are missing, stale, low-confidence, or partially available, HW must support degraded mode.

Example:

```text
HW_DEGRADED_MODE=YES_WITH_REASON_POWER_SENSOR_UNAVAILABLE
```

In degraded mode, HW may:

```text
- reduce parallelism
- reject long-run stress
- reject AVX512 or other heavy vector paths
- allow only short or low-risk workloads
- deny full power/thermal stability claims
- request an external or sacrificial hardware lane for stronger evidence
```

Degraded mode is not a failure by itself. It is a claim boundary.

---

## 20. Full Power and Thermal Execution Scenario

A correct power/thermal run should follow this sequence:

```text
1. Create run_id.
2. Start FCC runtime custody ledger.
3. Start witness process.
4. Platform captures sensor inventory.
5. HW validates sensor inventory.
6. HW captures idle baseline.
7. HW opens telemetry sampling window.
8. Workload begins.
9. HW continuously updates power, thermal, and frequency windows.
10. HW emits decision updates if state changes.
11. Workload ends or fails.
12. HW captures post-run state.
13. HW writes claim decisions.
14. FCC validates custody and witness linkage.
15. Reports are emitted.
```

If the workload fails:

```text
POST_FAILURE_RECORD_CREATED=YES
HW_FAILURE_TELEMETRY_ATTACHED=YES
HW_CLAIMS_BLOCKED=YES
```

A later successful rerun must not erase prior failure records.

---

## 21. Idle Baseline

No stress run should begin without an idle baseline unless a precise reason is recorded.

Required baseline fields:

```text
IDLE_BASELINE_CAPTURED=YES
IDLE_DURATION_SECONDS=<n>
IDLE_CPU_TEMP_C_MEAN=<value>
IDLE_CPU_TEMP_C_MAX=<value>
IDLE_PACKAGE_WATTS_MEAN=<value or NOT_AVAILABLE_WITH_REASON>
IDLE_FREQ_MHZ_MEAN=<value>
IDLE_LOAD_AVERAGE=<value>
```

The baseline helps detect:

```text
- high starting temperature
- abnormal idle consumption
- background load
- frequency drop under later load
- unreliable cooling conditions
```

If baseline is unavailable:

```text
HW_IDLE_BASELINE=NOT_RUN_WITH_REASON_<exact_reason>
```

---

## 22. Build Pressure

Full build is a valid physical workload because it stresses CPU, memory, compiler frontend/backend, and modules.

Required records:

```text
BUILD_PRESSURE_STARTED=YES
BUILD_PRESSURE_DURATION_MS=<value>
BUILD_MAX_TEMP_C=<value or NOT_AVAILABLE_WITH_REASON>
BUILD_MEAN_WATTS=<value or NOT_AVAILABLE_WITH_REASON>
BUILD_FREQ_MIN_MHZ=<value or NOT_AVAILABLE_WITH_REASON>
BUILD_THROTTLE_EVENTS=<count or NOT_AVAILABLE_WITH_REASON>
BUILD_EXIT_CODE=<value>
```

Build pressure alone must not prove runtime stability.

Correct claim:

```text
HW_BUILD_PRESSURE_CAPTURED=YES_WITH_LIVE_CUSTODY
```

Incorrect claim:

```text
SYSTEM_STABLE=YES
```

---

## 23. Full CTest Pressure

Full CTest is stronger than build pressure because it exercises runtime behavior.

Required records:

```text
FULL_CTEST_PRESSURE_STARTED=YES
FULL_CTEST_TOTAL=<value>
FULL_CTEST_PASSED=<value>
FULL_CTEST_FAILED=<value>
FULL_CTEST_MAX_TEMP_C=<value or NOT_AVAILABLE_WITH_REASON>
FULL_CTEST_POWER_WINDOW=<ref or NOT_AVAILABLE_WITH_REASON>
FULL_CTEST_THROTTLE_EVENTS=<count or NOT_AVAILABLE_WITH_REASON>
FULL_CTEST_FAILURES_IF_ANY=<list or none>
```

If any test fails:

```text
FULL_GRAPH_ACCEPTED=NO_WITH_REASON_FULL_CTEST_FAILURES
```

Even when power and thermal behavior looks stable, graph correctness is a separate claim.

---

## 24. Sanitizer Pressure

Sanitizer builds and sanitizer runtime must be separated.

Required categories:

```text
SANITIZER_CONFIGURE
SANITIZER_BUILD
SANITIZER_RUNTIME
SANITIZER_THERMAL_WINDOW
SANITIZER_POWER_WINDOW
```

HW may record sanitizer physical pressure, but must not claim sanitizer cleanliness.

Correct:

```text
HW_SANITIZER_PRESSURE_CAPTURED=YES_WITH_LIVE_CUSTODY
```

Incorrect:

```text
SANITIZER_CLEAN=YES
```

unless full sanitizer runtime executed and passed under the sanitizer gate.

---

## 25. Long-Run Stress

Sustained stability requires long-run execution.

Possible workloads:

```text
- repeated full build
- repeated full CTest
- sustained SIMD memory pressure
- sustained BFV / CKKS runtime loops
- HW telemetry polling stress
- FCC custody replay loop
```

Required records:

```text
LONG_RUN_DURATION_MS=<value>
THERMAL_SATURATION_REACHED=YES/NO/UNKNOWN_WITH_REASON
MAX_TEMP_C=<value or NOT_AVAILABLE_WITH_REASON>
P95_TEMP_C=<value or NOT_AVAILABLE_WITH_REASON>
P99_TEMP_C=<value or NOT_AVAILABLE_WITH_REASON>
WATTS_MEAN=<value or NOT_AVAILABLE_WITH_REASON>
WATTS_P95=<value or NOT_AVAILABLE_WITH_REASON>
FREQ_FLOOR_MHZ=<value or NOT_AVAILABLE_WITH_REASON>
THROTTLE_EVENTS=<count or NOT_AVAILABLE_WITH_REASON>
CRASH=YES/NO
OOM=YES/NO
TIMEOUT=YES/NO
```

If long-run did not execute:

```text
SUSTAINED_POWER_THERMAL_STABILITY=NOT_RUN_WITH_REASON_LONG_RUN_NOT_EXECUTED
```

---

## 26. Limits of HW Claims

HW may claim:

```text
The machine is currently safe for this workload.
The machine is thermally degraded.
The power sensor is unavailable.
The workload should be limited.
The workload must be rejected.
The physical claim is unproven.
```

HW must not claim by itself:

```text
The product is production ready.
The full graph is accepted.
The sanitizer is clean.
The entire system is market ready.
```

Those claims require broader gates.

---

## 27. Correct Claim Examples

### Stable workload window

```text
HW_POWER_THERMAL_EVALUATION=YES_WITH_LIVE_CUSTODY
HW_THERMAL_STATE=NORMAL
HW_POWER_STATE=NORMAL
HW_FREQUENCY_STABILITY=YES_WITH_LIVE_CUSTODY
HW_THROTTLING_DETECTED=NO_WITH_LIVE_CUSTODY
HW_HEAVY_WORKLOAD_ALLOWED=YES_WITH_HW_AUTHORITY
```

### Hot state

```text
HW_THERMAL_STATE=HOT
HW_HEAVY_WORKLOAD_ALLOWED=NO_WITH_REASON_THERMAL_HEADROOM_LOW
HW_SIMD_HEAVY_PATH_ALLOWED=NO_WITH_REASON_THERMAL_STATE_HOT
HW_FHE_LONG_RUN_ALLOWED=NO_WITH_REASON_THERMAL_STATE_HOT
```

### Missing power sensor

```text
HW_POWER_TELEMETRY=NOT_AVAILABLE_WITH_REASON_PLATFORM_POWER_SENSOR_NOT_EXPOSED
HW_POWER_STATE=UNKNOWN
HW_POWER_CLAIM=NO_WITH_REASON_POWER_TELEMETRY_UNAVAILABLE
HW_THERMAL_EVALUATION=YES_WITH_LIVE_CUSTODY
```

### Throttling observed

```text
HW_THERMAL_THROTTLING_DETECTED=YES_WITH_LIVE_CUSTODY
HW_FREQUENCY_STABILITY=NO_WITH_REASON_THERMAL_THROTTLING_DETECTED
HW_LONG_RUN_ALLOWED=NO_WITH_REASON_THROTTLED_UNDER_LOAD
```

---

## 28. Required Test Matrix

### 28.1 Fixture Tests

Fixture tests do not touch live hardware. They validate HW logic with synthetic telemetry.

Required cases:

```text
temp normal -> NORMAL
temp warm -> WARM
temp hot -> HOT
temp critical -> CRITICAL
missing sensor -> UNKNOWN or DEGRADED
stale sample -> REJECTED
invalid negative temperature -> REJECTED
impossible high temperature -> REJECTED
energy counter wrap -> handled or rejected explicitly
energy counter decreases unexpectedly -> REJECTED
frequency drop -> THROTTLED
power cap signal -> POWER_CAPPED
```

### 28.2 Platform Boundary Tests

Required cases:

```text
HW does not read OS sensors directly.
HW accepts telemetry only through platform.api.
SIMD cannot bypass HW.
FHE cannot bypass HW.
Raw CPU probing remains outside SIMD/FHE.
```

### 28.3 Live Telemetry Tests

Required cases:

```text
HW_SENSOR_INVENTORY_CAPTURED=YES
HW_LIVE_SNAPSHOT_CAPTURED=YES
HW_DECISION_CAPSULE_EMITTED=YES
HW_TELEMETRY_CONFIDENCE_CLASSIFIED=YES
```

### 28.4 Stress Tests

Required cases:

```text
HW_BUILD_PRESSURE_CAPTURED=YES
HW_FULL_CTEST_PRESSURE_CAPTURED=YES/NOT_RUN_WITH_REASON
HW_LONG_RUN_PRESSURE_CAPTURED=YES/NOT_RUN_WITH_REASON
HW_THROTTLE_EVENTS_RECORDED=YES/NO_WITH_CUSTODY/NOT_AVAILABLE_WITH_REASON
```

### 28.5 Overclaim Rejection Tests

Required rejections:

```text
POWER_STABLE=YES without telemetry
THERMAL_STABLE=YES without workload context
NO_THROTTLING=YES without frequency/throttle evidence
FULL_GRAPH_ACCEPTED from HW-only result
PRODUCTION_READY from power test
SANITIZER_CLEAN from non-sanitizer run
sensor missing treated as success
build pass treated as runtime stability
```

---

## 29. Required Flags

Recommended positive flags:

```text
HW_POWER_AUTHORITY_MODEL_ENFORCED=YES
HW_POWER_TELEMETRY_SOURCE_PLATFORM_API_ONLY=YES
HW_DIRECT_OS_POWER_PROBING_REJECTED=YES
HW_SENSOR_INVENTORY_CAPTURED=YES
HW_SENSOR_CONFIDENCE_CLASSIFIED=YES
HW_IDLE_BASELINE_CAPTURED=YES
HW_POWER_WINDOW_COMPUTED=YES
HW_THERMAL_WINDOW_COMPUTED=YES
HW_FREQUENCY_WINDOW_COMPUTED=YES
HW_THROTTLING_STATE_CLASSIFIED=YES
HW_DECISION_CAPSULE_EMITTED=YES
HW_DOWNSTREAM_LIMITS_EMITTED=YES
HW_DEGRADED_MODE_SUPPORTED=YES
HW_OVERCLAIM_REJECTED=YES
```

Recommended failure and limitation flags:

```text
HW_POWER_TELEMETRY=NOT_AVAILABLE_WITH_REASON_<reason>
HW_THERMAL_TELEMETRY=NOT_AVAILABLE_WITH_REASON_<reason>
HW_SENSOR_CONFIDENCE=REJECTED_WITH_REASON_<reason>
HW_DECISION_CAPSULE=NO_WITH_REASON_<reason>
HW_HEAVY_WORKLOAD_ALLOWED=NO_WITH_REASON_<reason>
```

---

## 30. Prohibited Behaviors

The following must be rejected:

```text
- Accepting final_flags as truth.
- Accepting UI green state as truth.
- Accepting evidence index as proof.
- Reusing packaged logs as fresh runtime evidence.
- Treating targeted runs as full graph evidence.
- Treating missing sensors as stability.
- Treating temperature alone as power evidence.
- Treating watts alone as thermal stability.
- Treating build success alone as runtime stability.
- Treating HW-only pass as production readiness.
- Allowing SIMD or FHE to read hardware state directly.
- Erasing a prior failure after a later green rerun.
```

---

## 31. Conceptual C++ Model

This is a conceptual model, not a mandatory implementation form.

```cpp
namespace xps::hw {

enum class SensorTrust {
    Trusted,
    Degraded,
    Unavailable,
    Rejected,
    Unknown
};

enum class ThermalState {
    Unknown,
    Normal,
    Warm,
    Hot,
    Throttled,
    Critical,
    SensorUntrusted
};

enum class PowerState {
    Unknown,
    Normal,
    Elevated,
    Limited,
    PowerCapped,
    BatteryConstrained,
    SensorUntrusted
};

struct PowerThermalSample {
    u64 timestamp_ns;

    bool cpu_temp_available;
    double cpu_temp_c;

    bool package_energy_available;
    u64 package_energy_uj;

    bool package_watts_available;
    double package_watts;

    bool frequency_available;
    u32 cpu_frequency_mhz;

    bool thermal_throttle_signal;
    bool power_limit_signal;

    SensorTrust trust;
};

struct PowerThermalWindow {
    u32 sample_count;
    u64 duration_ns;

    double temp_max_c;
    double temp_p95_c;
    double temp_mean_c;

    double watts_max;
    double watts_p95;
    double watts_mean;

    u32 throttle_events;
    u32 missing_samples;

    SensorTrust trust;
};

struct HwDecisionCapsule {
    ThermalState thermal_state;
    PowerState power_state;

    bool heavy_workload_allowed;
    bool long_run_allowed;
    bool sanitizer_allowed;
    bool simd_heavy_path_allowed;
    bool fhe_heavy_path_allowed;

    u32 allowed_parallelism;
    u32 allowed_vector_width_bits;

    const char* reason;
};

}
```

The central requirement is not the exact structure names. The central requirement is the authority model:

```text
Telemetry samples are not enough.
Analysis windows are not enough.
A sealed HW decision capsule is the permitted downstream interface.
```

---

## 32. Acceptance Criteria

The HW power and thermal authority model is accepted only if:

```text
- HW does not read external machine state outside authorized platform.api telemetry.
- Telemetry has an explicit trust model.
- Unavailable data never becomes success.
- Invalid readings are rejected.
- Power is derived correctly from energy counters when necessary.
- Thermal, power, and frequency behavior are analyzed over time.
- Throttling is recorded and not hidden.
- HW emits decision capsules for downstream units.
- SIMD and FHE cannot bypass HW authority.
- FCC custody links physical claims to evidence.
- Failure records cannot be erased by rerun.
```

The model is rejected if:

```text
- Missing sensors produce stability claims.
- Build pass produces production readiness.
- HW permits SIMD or FHE local hardware inference.
- final_flags prove a claim.
- UI proves a claim.
- Documentation describes behavior not implemented in runtime.
```

---

## 33. Final Engineering Rule

The final rule for XPScerpto HW power and thermal handling is:

```text
No power claim without telemetry.
No thermal claim without workload context.
No stability claim without a time window.
No downstream hardware inference outside HW.
No production claim from HW-only evidence.
No success from missing sensors.
No erased failure.
```

The intended final authority chain is:

```text
platform.api telemetry
    ↓
HW validation
    ↓
HW power / thermal / frequency windows
    ↓
HW decision capsule
    ↓
downstream execution limits
    ↓
FCC custody and claim decisions
```

Under this model, HW becomes the physical execution authority of XPScerpto. It decides when heavy execution is allowed, when it must be limited, when it must fail closed, and when a claim remains unproven.
