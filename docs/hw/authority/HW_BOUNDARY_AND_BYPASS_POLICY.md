# HW Boundary and Bypass Policy

**Document status:** Normative engineering documentation  
**Scope:** XPScerpto `hw` unit  
**Claim boundary:** This document does not claim `PRODUCTION_READY`, `SANITIZER_CLEAN`, `FULL_GRAPH_ACCEPTED`, market readiness, or external certification.  
**Authority rule:** Platform observes. HW evaluates. Downstream units obey. FCC records and judges claims.


**Document type:** Forbidden hardware inference and policy-scan contract

---

## Purpose

This document defines bypasses that must be rejected outside approved platform observation.

## Forbidden Outside Platform Provider Boundary

```text
/proc/cpuinfo
/sys/class/thermal
/sys/class/powercap
/sys/devices/system/cpu
lm-sensors
sensors command output
rdmsr
cpuid
__cpuid
__get_cpuid
__builtin_cpu_supports
__builtin_cpu_init
runtime routing from defined(__AVX2__)
runtime routing from defined(__AVX512F__)
runtime routing from defined(__ARM_NEON)
direct thermal device reads
direct powercap reads
direct frequency sysfs reads
```

## Compiler Macro Rule

Compiler macros may describe compile-time availability. They must not authorize runtime routes.

## Downstream Rule

SIMD, FHE, PQC, runtime, and tests must not recreate HW authority locally.

## Final Rule

```text
Any hardware inference outside platform observation and HW evaluation is an authority violation.
```
