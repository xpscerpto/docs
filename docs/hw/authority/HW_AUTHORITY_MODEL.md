# XPScerpto HW Authority Model

**Document status:** Normative engineering documentation  
**Scope:** XPScerpto `hw` unit  
**Claim boundary:** This document does not claim `PRODUCTION_READY`, `SANITIZER_CLEAN`, `FULL_GRAPH_ACCEPTED`, market readiness, or external certification.  
**Authority rule:** Platform observes. HW evaluates. Downstream units obey. FCC records and judges claims.


**Document type:** Authority model and execution-boundary contract

---

## Purpose

This document defines who may observe hardware, who may evaluate it, who must obey decisions, and who judges claims.

## Canonical Chain

```text
platform provider
    ↓
platform.api
    ↓
HW input acceptance
    ↓
HW intelligence and evaluation
    ↓
HW decision capsule
    ↓
HW routing envelope
    ↓
SIMD / FHE / PQC / runtime / tests
    ↓
FCC custody and claim decision
```

## Roles

```text
Platform: observes the external machine.
HW: evaluates authorized facts and emits decisions.
Downstream: executes under HW routing envelopes.
FCC: records evidence and judges claims.
```

## Capability Is Not Permission

```text
CAN = hardware fact
MAY = HW-authorized execution permission
```

Example:

```text
CPU_CAN_AVX512=YES
SIMD_AVX512_ALLOWED=NO_WITH_REASON_THERMAL_HEADROOM_LOW
```

## Prohibited Authority Sources

```text
local CPUID probing
__builtin_cpu_supports
compiler ISA macros as runtime authority
/proc/cpuinfo
/sys/class/thermal
/sys/class/powercap
UI green state
final_flags
packaged logs as fresh runtime
evidence index alone
```

## Final Rule

```text
No hardware-dependent execution is valid unless it descends from an HW decision artifact produced from authorized platform evidence.
```
