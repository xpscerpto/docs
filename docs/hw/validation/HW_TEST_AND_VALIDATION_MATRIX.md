# XPScerpto HW Test and Validation Matrix

**Document status:** Normative engineering documentation  
**Scope:** XPScerpto `hw` unit  
**Claim boundary:** This document does not claim `PRODUCTION_READY`, `SANITIZER_CLEAN`, `FULL_GRAPH_ACCEPTED`, market readiness, or external certification.  
**Authority rule:** Platform observes. HW evaluates. Downstream units obey. FCC records and judges claims.


**Document type:** Validation, policy scan, and overclaim rejection contract

---

## Purpose

HW is not validated by green output. HW is validated by proving that correct authority paths work and incorrect authority paths are rejected.

## Required Validation Areas

```text
functional scope
authority boundary
platform input
signal intelligence
sensor trust and confidence
capability authorization
pressure intelligence
stability intelligence
snapshot model
decision capsule
routing envelope
power/thermal authority
measurement and stability
evidence and custody
downstream integration
policy scans
overclaim rejection
live telemetry
stress and long-run
sanitizer boundary
```

## Required Negative Tests

```text
SIMD local CPUID route authorization
FHE local thermal sensor authorization
PQC local pressure inference
runtime UI green truth
test compiler macro authority
final_flags truth
packaged logs fresh-runtime claim
fixture promoted to live proof
replay promoted to fresh runtime
missing sensor becomes success
not-run becomes success
```

## Policy Scan Targets

```text
/proc/cpuinfo
/sys/class/thermal
/sys/class/powercap
__builtin_cpu_supports
cpuid
compiler ISA macro runtime routing
direct power/thermal/frequency sysfs reads
```

## Final Rule

```text
Every HW claim path must have an adversarial rejection test for the strongest false claim it could accidentally support.
```
