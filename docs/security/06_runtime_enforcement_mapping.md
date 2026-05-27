# XPScerpto Runtime Enforcement Mapping and Release Gate Specification

**Document ID:** XPS-SOV-MAP-006  
**Status:** Normative enforcement and release-gate mapping specification  
**Scope:** specification-to-contract mapping, compile enforcement, runtime enforcement, adversarial tests, forensic evidence, CI evidence bundles, conformance levels, production release gates  
**Version:** 1.1  
**Document Type:** Normative Security, Enforcement, Traceability, and Release Gate Specification  
**Applies To:** platform, platform.api, hardware authority, SIMD, AES, FHE, PQC, provider path, sealed capability capsule admission, failure semantics, audit ledger, CI evidence, release governance

---

## 1. Purpose

This document maps XPScerpto's formal sovereign specifications to enforceable implementation contracts.

A specification that cannot be mapped to compile contracts, runtime enforcement, adversarial tests, and independently verifiable evidence is incomplete for production XPScerpto use.

Documentation alone is not sovereign closure.

A runtime check alone is not sovereign closure.

A compile guard alone is not sovereign closure.

A test without evidence is not sovereign closure.

Evidence controlled only by the executor is not sovereign production evidence.

The required enforcement chain is:

```text
Specification
    ↓
Compile Contracts
    ↓
Runtime Enforcement
    ↓
Adversarial Tests
    ↓
Forensic / Audit Evidence
    ↓
Independent Verification
    ↓
Release Gate Decision
```

This document defines how normative requirements from the sovereign specification layer are converted into compile-time boundaries, runtime gates, test obligations, evidence artifacts, conformance levels, and release decisions.

A production sovereign claim is valid only when every required authority path can be traced through this complete chain.

## 2. Normative Language

The terms **MUST**, **MUST NOT**, **REQUIRED**, **SHALL**, **SHALL NOT**, **SHOULD**, **MAY**, and **OPTIONAL** are normative.

A component that violates a **MUST** or **MUST NOT** requirement is non-conforming even if functional tests pass.

A production implementation MUST treat this document as an enforcement specification, not as advisory traceability documentation.

## 3. Normative References

This specification is part of the XPScerpto sovereign specification layer.

| Document | Role |
|---|---|
| XPS-SOV-TM-001 — Sovereign Threat Model | Defines threat boundary, adversaries, protected assets, assumptions, and non-claims. |
| XPS-SOV-AUTH-002 — Sovereign Authority Model | Defines authority lineage, epochs, revocation, admission, and safe halt behavior. |
| XPS-SOV-CAP-003 — Sealed Capability Capsule Specification | Defines capsule format, validation, replay prevention, lifecycle, and production admission. |
| XPS-SOV-FAIL-004 — Failure Semantics and Recovery Specification | Defines deterministic failure behavior, denial, quarantine, safe halt, and recovery authority. |
| XPS-SOV-AUD-005 — Sovereign Audit Ledger and Verification Specification | Defines ledger evidence, verification, witness binding, retention, and production claim gating. |
| XPS-SOV-MAP-006 — Runtime Enforcement Mapping and Release Gate Specification | Defines the mapping from normative specifications to enforcement, tests, evidence, and release gates. |

If this document conflicts with another XPScerpto sovereign specification, the stricter authority-denying, evidence-denying, or release-blocking interpretation MUST apply.

## 4. Core Enforcement Principle

Every sovereign claim MUST be enforceable.

The following statements are normative:

```text
A normative requirement that has no enforcement mapping is incomplete.
An enforcement mapping that has no runtime or compile contract is incomplete.
A runtime gate that has no adversarial test is incomplete.
A test that has no evidence artifact is incomplete.
Evidence controlled only by the executor is incomplete for production.
A release gate that ignores incomplete evidence is non-conforming.
```

A requirement may exist at a lower maturity level, but it MUST NOT be described as production sovereign closure until it reaches the required conformance level for the claimed path.

## 5. Enforcement Chain

The enforcement chain is:

```text
Normative Specification
    ↓
Contract Identifier
    ↓
Compile Contract
    ↓
Runtime Gate
    ↓
Adversarial Test
    ↓
Evidence Artifact
    ↓
Audit Ledger / Forensic Record
    ↓
Independent Verification
    ↓
Release Decision
```

Each layer has a distinct role.

No single layer is sufficient.

Runtime checks do not excuse missing compile boundaries.

Compile boundaries do not excuse missing runtime admission.

Runtime admission does not excuse missing adversarial tests.

Tests do not excuse missing evidence.

Evidence does not excuse missing independent verification.

Independent verification does not excuse missing release gate policy.

## 6. Enforcement Layers

| Layer | Responsibility | Failure mode |
|---|---|---|
| specification | Defines normative authority, admission, audit, and failure behavior. | Requirement cannot be tested or enforced. |
| contract identifier | Provides stable traceability from requirement to implementation. | Requirement becomes ambiguous or untraceable. |
| compile contracts | Prevent invalid imports, forbidden APIs, raw execution paths, and policy-incompatible builds. | Build/configure/static scan fails. |
| runtime enforcement | Validates authority predicates and denies unsafe execution. | Deny, halt, quarantine, revoke, or fail closed. |
| adversarial tests | Prove invalid cases fail closed and valid cases do not bypass authority. | CI/test fails or marks coverage incomplete. |
| forensic evidence | Records decision, failure, route, capsule, policy, and proof artifacts. | Evidence verification fails or remains insufficient. |
| audit ledger | Makes execution claims tamper-evident and independently verifiable. | Ledger verification fails or production claim is rejected. |
| release gate | Converts enforcement evidence into production claim decision. | Release claim blocked, limited, or downgraded. |

A conforming implementation MUST maintain traceability across all layers.

## 7. Contract Identifier Registry

Every sovereign requirement MUST have a stable contract identifier.

The following registry defines minimum required identifiers.

| ID | Requirement | Source document |
|---|---|---|
| `SPEC.NO_AMBIENT_AUTHORITY` | Module presence, import status, link status, or repository location does not grant execution rights. | XPS-SOV-TM-001, XPS-SOV-CAP-003 |
| `SPEC.NO_LOCAL_HW_PROBING` | Domains MUST NOT probe hardware locally to authorize routes. | XPS-SOV-TM-001 |
| `SPEC.AND_GATED_AUTHORITY` | All authority predicates MUST pass before production execution. | XPS-SOV-AUTH-002, XPS-SOV-CAP-003 |
| `SPEC.CAPSULE_REQUIRED` | Protected execution requires a valid sealed capability capsule. | XPS-SOV-CAP-003 |
| `SPEC.EPOCH_REVOCATION` | Revoked, stale, or mismatched epochs invalidate authority. | XPS-SOV-AUTH-002, XPS-SOV-FAIL-004 |
| `SPEC.FAIL_CLOSED` | Missing, invalid, stale, or contradictory evidence denies execution or safe-halts. | XPS-SOV-FAIL-004 |
| `SPEC.AUDIT_REQUIRED` | Production execution requires pre-execution audit commitment when policy demands it. | XPS-SOV-AUD-005 |
| `SPEC.EXECUTOR_NOT_SOLE_AUDITOR` | Executor-only logs are not sovereign production evidence. | XPS-SOV-AUD-005 |
| `SPEC.ROUTE_CAPSULE_BINDING` | Requested route and kernel class MUST match capsule authority. | XPS-SOV-CAP-003 |
| `SPEC.PROVIDER_FILTERING` | External/provider facts must be filtered and bound by platform policy before becoming evidence. | XPS-SOV-TM-001, XPS-SOV-AUTH-002 |
| `SPEC.DOWNGRADE_PREVENTION` | Production failures MUST NOT become silent fallbacks or diagnostic production claims. | XPS-SOV-FAIL-004 |
| `SPEC.LEDGER_TAMPER_DETECTION` | Ledger mutation, deletion, fork, or replay MUST be detected. | XPS-SOV-AUD-005 |
| `SPEC.DIAGNOSTIC_NOT_PRODUCTION` | Diagnostic, fixture, or test-only evidence MUST NOT support production claims. | XPS-SOV-FAIL-004, XPS-SOV-AUD-005 |
| `SPEC.SECRET_SAFE_EVIDENCE` | Evidence MUST NOT expose plaintext secrets, keys, or secret-dependent traces. | XPS-SOV-AUD-005 |
| `SPEC.RELEASE_GATE_REQUIRED` | Production sovereign claims require release gate approval based on evidence. | XPS-SOV-MAP-006 |

## 8. Machine-Readable Enforcement Registry Schema

The enforcement registry SHOULD be machine-readable.

A registry entry MUST be semantically equivalent to:

```text
EnforcementRegistryEntry {
    spec_id;
    source_document_id;
    normative_requirement;
    compile_contract_id;
    forbidden_symbols;
    forbidden_imports;
    forbidden_macros;
    required_api_shape;
    required_build_inputs;
    runtime_gate_id;
    required_runtime_inputs;
    accepted_runtime_states;
    rejected_runtime_states;
    failure_code;
    denial_state;
    test_id;
    expected_test_result;
    evidence_artifact_id;
    evidence_schema;
    audit_ledger_required;
    independent_verification_required;
    minimum_conformance_level;
    release_gate_effect;
    owner_module;
    implementation_surface;
}
```

Rules:

1. Every production sovereign claim MUST map to at least one `EnforcementRegistryEntry`.
2. Every required authority path MUST reach L5 unless explicitly documented as non-production or limited.
3. A missing registry entry for a required production path MUST block production claims.
4. A registry entry without test evidence MUST NOT be promoted to production closure.
5. A registry entry with executor-only evidence MUST NOT reach L5.

## 9. Compile Contract Classes

Compile contracts prevent forbidden authority patterns from entering the production build.

A conforming implementation SHOULD classify compile contracts as follows.

| Compile contract class | Purpose |
|---|---|
| `COMPILE.FORBIDDEN_IMPORT` | Blocks private authority surfaces outside approved modules. |
| `COMPILE.FORBIDDEN_SYMBOL` | Blocks forbidden functions, intrinsics, system probes, or raw APIs. |
| `COMPILE.FORBIDDEN_MACRO_AUTHORITY` | Blocks compiler target macros from becoming authority. |
| `COMPILE.API_SHAPE_REQUIRED` | Requires capsule-aware admission interfaces for protected execution. |
| `COMPILE.NO_RAW_KERNEL_EXPORT` | Prevents direct production kernel entrypoints bypassing admission. |
| `COMPILE.TEST_SURFACE_ISOLATION` | Prevents fixtures, adapters, and diagnostics from leaking into production. |
| `COMPILE.BUILD_POLICY_BINDING` | Ensures build flags and generated configuration match policy. |
| `COMPILE.SPEC_REGISTRY_PRESENT` | Ensures generated registry entries exist for production claims. |
| `COMPILE.FAILURE_CODE_ALIGNMENT` | Ensures failure codes match the normative registry. |
| `COMPILE.LEDGER_SCHEMA_ALIGNMENT` | Ensures audit evidence structures match ledger specification. |

Compile scans MUST include:

1. source files;
2. module interfaces;
3. module implementation units;
4. generated files;
5. public headers;
6. build-generated configuration;
7. test adapters that may leak into production;
8. CMake or build system surfaces that alter compile policy;
9. demo or diagnostic code linked into production artifacts.

## 10. Runtime Gate Schema

A runtime gate is the executable enforcement point for a sovereign requirement.

A runtime gate MUST be semantically equivalent to:

```text
RuntimeGate {
    runtime_gate_id;
    spec_id;
    required_inputs;
    accepted_states;
    rejected_states;
    failure_code;
    denial_state;
    audit_required;
    evidence_record;
    final_liveness_check_required;
    production_claim_effect;
}
```

Example:

```text
RuntimeGate {
    runtime_gate_id = RUNTIME.CAPSULE_ADMISSION;
    spec_id = SPEC.CAPSULE_REQUIRED;
    required_inputs = policy_hash, build_fingerprint, epoch_snapshot, capsule, audit_reservation;
    accepted_states = valid_capsule, live_epoch, route_matched, audit_reserved;
    rejected_states = missing_capsule, stale_epoch, audit_missing, route_mismatch;
    failure_code = deterministic;
    denial_state = DENY_EXECUTION;
    audit_required = true;
    final_liveness_check_required = true;
    production_claim_effect = block_if_failed;
}
```

A runtime gate MUST NOT silently convert production failure into best-effort execution.

A runtime gate MUST NOT emit production output after denial.

A runtime gate MUST map every rejected state to a deterministic failure code.

## 11. Failure Code Mapping

Runtime gates MUST use failure codes from the appropriate normative source.

| Condition | Failure code source | Required code |
|---|---|---|
| missing capsule | XPS-SOV-FAIL-004 | `NO_CAPSULE` |
| invalid capsule signature | XPS-SOV-CAP-003 | `CAPSULE_SIGNATURE_INVALID` |
| capsule policy mismatch | XPS-SOV-CAP-003 | `CAPSULE_POLICY_MISMATCH` |
| capsule build mismatch | XPS-SOV-CAP-003 | `CAPSULE_BUILD_MISMATCH` |
| hardware evidence mismatch | XPS-SOV-CAP-003 | `CAPSULE_HARDWARE_EVIDENCE_MISMATCH` |
| stale epoch | XPS-SOV-FAIL-004 | `EPOCH_NOT_LIVE` |
| epoch TOCTOU | XPS-SOV-FAIL-004 | `EPOCH_TOCTOU_REJECTED` |
| route mismatch | XPS-SOV-CAP-003 | `CAPSULE_ROUTE_CLASS_REJECTED` or `ROUTE_NOT_AUTHORIZED` |
| missing audit commitment | XPS-SOV-FAIL-004 | `AUDIT_COMMIT_MISSING` |
| ledger invalid | XPS-SOV-AUD-005 | `LEDGER_CHAIN_INVALID` |
| local hardware probe | XPS-SOV-FAIL-004 | `LOCAL_PROBE_ATTEMPT` |
| provider fact rejected | XPS-SOV-FAIL-004 | `PROVIDER_FACT_REJECTED` |
| diagnostic production overclaim | XPS-SOV-FAIL-004 | `PRODUCTION_OVERCLAIM_REJECTED` |
| executor-only evidence | XPS-SOV-AUD-005 | `EXECUTOR_ONLY_EVIDENCE_REJECTED` |
| secret evidence exposure | XPS-SOV-AUD-005 | `LEDGER_SECRET_MATERIAL_EXPOSURE` |
| forbidden downgrade | XPS-SOV-FAIL-004 | `DOWNGRADE_FORBIDDEN` |

A mapping document MUST NOT invent new production failure codes unless the owning specification is updated or the new code is explicitly marked provisional.

## 12. Evidence Artifact Schema

Each enforcement test SHOULD produce a machine-readable artifact equivalent to:

```text
EnforcementEvidence {
    evidence_id;
    spec_id;
    compile_contract_id;
    runtime_gate_id;
    test_id;
    result;
    failure_code;
    denial_state;
    policy_hash;
    build_fingerprint;
    authority_epoch;
    capsule_id;
    domain;
    route_class;
    kernel_class;
    ledger_root;
    witness_commitment;
    timestamp_or_counter;
    executor_id;
    verifier_id;
    source_tree_hash;
    evidence_hash;
}
```

For production gates, `result` MUST NOT be ambiguous.

Allowed result classes are:

```text
PASS
PASS_WITH_NON_PRODUCTION_LIMITATION
FAIL_CLOSED
FAIL_OPEN
PARTIAL_COVERAGE
UNKNOWN_COVERAGE
NOT_EXECUTED
NOT_APPLICABLE_WITH_REASON
```

`FAIL_OPEN` is a release blocker for sovereign production claims.

`UNKNOWN_COVERAGE` is a release blocker for required production authority paths.

`PARTIAL_COVERAGE` MUST NOT be presented as full production closure.

## 13. Evidence Bundle Manifest

A CI or release evidence bundle SHOULD contain a manifest equivalent to:

```text
EvidenceBundleManifest {
    bundle_id;
    source_tree_hash;
    build_fingerprint;
    policy_hash;
    spec_registry_hash;
    compile_guard_report_hash;
    runtime_gate_report_hash;
    test_report_hash;
    ledger_root;
    witness_summary_hash;
    failure_summary_hash;
    conformance_level_summary;
    fail_open_count;
    unknown_coverage_count;
    partial_coverage_count;
    release_decision;
    limitations;
    exact_blockers;
}
```

Rules:

1. A release evidence bundle MUST bind source tree hash and build fingerprint.
2. A release evidence bundle MUST identify the spec registry used.
3. A release evidence bundle MUST include compile, runtime, test, and evidence summaries.
4. A release evidence bundle MUST report `FAIL_OPEN`.
5. A release evidence bundle MUST report `UNKNOWN_COVERAGE`.
6. A release evidence bundle MUST report exact blockers.
7. A production release claim MUST NOT rely on an evidence bundle that cannot be independently reviewed.

## 14. Compile Contract Mapping

### 14.1 No Local Hardware Probing

**Specification**

```text
SPEC.NO_LOCAL_HW_PROBING
```

**Compile Contract**

The build and source policy guard MUST reject unauthorized usage of:

```text
CPUID intrinsics or inline assembly outside authorized hardware authority
__builtin_cpu_supports
compiler target macro based route authorization such as __AVX2__, __AVX512F__, __ARM_NEON
/proc/cpuinfo and OS-specific CPU probe files outside platform provider path
std::hardware_destructive_interference_size as authority evidence
platform.private/provider.private imports outside platform boundary
direct OS probing by SIMD, AES, FHE, or PQC
```

**Runtime Enforcement**

```text
LOCAL_PROBE_ATTEMPT -> TERMINATE_EXECUTION + FORENSIC_ESCALATION
```

**Tests**

```text
LOCAL_PROBE_DETECTED = FAIL_CLOSED
UNAUTHORIZED_PROBE_ATTEMPT -> EXECUTION_REJECTED
```

**Evidence**

```text
failure_code = LOCAL_PROBE_ATTEMPT
status       = TERMINATE_EXECUTION
route        = attempted domain/kernel route
```

### 14.2 Capsule-Required Protected Execution

**Specification**

```text
SPEC.CAPSULE_REQUIRED
```

**Compile Contract**

Production APIs MUST expose protected execution entrypoints only through capsule-aware admission functions.

Raw-key, raw-route, raw-dispatch, or direct-kernel production APIs MUST be quarantined, removed, or explicitly marked non-production.

**Runtime Enforcement**

```text
if capsule == missing:
    DENY_EXECUTION(NO_CAPSULE)
```

**Tests**

```text
MISSING_CAPSULE_PRODUCTION_API = DENIED
RAW_ROUTE_PRODUCTION_ENTRYPOINT = NOT_EXPORTED_OR_REJECTED
DIRECT_KERNEL_PRODUCTION_CALL = DENIED
```

**Evidence**

```text
decision_status = denied
failure_code    = NO_CAPSULE
```

### 14.3 AND-Gated Authority

**Specification**

```text
authorized = policy AND build AND hardware AND epoch AND capsule AND route AND audit
```

**Compile Contract**

Admission code MUST require inputs for:

```text
policy_hash
build_fingerprint
hardware_evidence_hash
epoch_snapshot
sealed_capsule
route_class
kernel_class
audit_reservation_or_ledger_root
```

Compile-time interfaces SHOULD make partial authority objects unrepresentable for production admission.

**Runtime Enforcement**

Every predicate MUST map to a deterministic denial code.

**Tests**

```text
MISSING_POLICY_HASH = SAFE_HALT
MISSING_BUILD_FINGERPRINT = SAFE_HALT
MISSING_HARDWARE_EVIDENCE = SAFE_HALT
STALE_EPOCH = REVOKE_CAPSULE
AUDIT_COMMIT_MISSING = DENY_EXECUTION
ROUTE_MISMATCH = DENY_EXECUTION
```

**Evidence**

Each denial MUST record the missing predicate and the authority lineage fragment if available.

### 14.4 Route Binding

**Specification**

```text
requested_kernel_class must be admitted by capsule.allowed_kernel_class
requested_route_class must be admitted by capsule.allowed_route_class
```

**Compile Contract**

Dispatch plans MUST be typed or tagged by route class.

Domains MUST NOT convert an unauthorized route class into an authorized one by:

```text
cast
enum alias
macro
fallback table
string lookup
debug override
test adapter
```

**Runtime Enforcement**

```text
if request.kernel_class not admitted by capsule.allowed_kernel_class:
    DENY_EXECUTION(ROUTE_NOT_AUTHORIZED)

if request.route_class not admitted by capsule.allowed_route_class:
    DENY_EXECUTION(CAPSULE_ROUTE_CLASS_REJECTED)
```

**Tests**

```text
SIMD_DISPATCH_MANIPULATION = ROUTE_NOT_AUTHORIZED
AES_FAST_PATH_WITH_SCALAR_CAPSULE = DENIED
FHE_BOOTSTRAP_ROUTE_WITH_EVAL_ONLY_CAPSULE = DENIED
PQC_LEGACY_ROUTE_WITH_PRODUCTION_CAPSULE = DENIED
```

**Evidence**

Record:

```text
requested_route
requested_kernel_class
capsule_route_class
capsule_kernel_class
execution_domain
policy_hash
authority_epoch
failure_code
```

### 14.5 Audit Commitment Required

**Specification**

```text
SPEC.AUDIT_REQUIRED
```

**Compile Contract**

Production admission APIs MUST require an audit commitment, audit reservation, or ledger writer capability.

Executor-only log interfaces MUST NOT satisfy the production audit contract.

**Runtime Enforcement**

```text
if audit_commitment_root missing:
    EXECUTION_NOT_SOVEREIGN
    DENY_EXECUTION(AUDIT_COMMIT_MISSING)
```

**Tests**

```text
EXECUTOR_ONLY_LOG_ACCEPTED = FAIL
AUDIT_COMMIT_MISSING = EXECUTION_NOT_SOVEREIGN
AUDIT_RESERVATION_FAILURE = DENIED
```

**Evidence**

Record denied request hash and reason.

If ledger is unavailable, return terminal status and do not claim sovereign execution.

### 14.6 Executor Not Sole Auditor

**Specification**

```text
SPEC.EXECUTOR_NOT_SOLE_AUDITOR
```

**Compile Contract**

Production evidence APIs MUST distinguish executor, ledger writer, verifier, and checkpoint roles.

Executor-only logging APIs MUST NOT satisfy production evidence requirements.

**Runtime Enforcement**

```text
if evidence_source == executor_only:
    reject_production_claim(EXECUTOR_ONLY_EVIDENCE_REJECTED)
```

**Tests**

```text
EXECUTOR_ONLY_LOG_REJECTED
LEDGER_VERIFIER_REQUIRED
EXTERNAL_CHECKPOINT_REQUIRED_WHEN_POLICY_DEMANDS
```

**Evidence**

Record verifier identity, ledger root, witness commitment, and evidence source class.

## 15. Full Enforcement Matrix

| Specification | Compile contract | Runtime enforcement | Test assertion | Evidence |
|---|---|---|---|---|
| no ambient authority | no direct production kernel exports | deny direct route | `AMBIENT_AUTH_DENIED` | `NO_CAPSULE` |
| no local hardware probing | static scan/import guard | terminate detected probe | `LOCAL_PROBE_DETECTED=FAIL_CLOSED` | `UNAUTHORIZED_PROBE_ATTEMPT` |
| sealed capsule required | capsule-aware API surface | verify capsule before route | `FORGED_CAPSULE_DENIED` | `CAPSULE_SIGNATURE_INVALID` |
| epoch revocation | epoch type required at admission | reject stale epoch | `STALE_EPOCH_REVOKED` | `EPOCH_NOT_LIVE` |
| route binding | route class typed/narrowed | deny route mismatch | `ROUTE_MISMATCH_DENIED` | `ROUTE_NOT_AUTHORIZED` |
| audit required | audit root required in production API | deny missing audit | `AUDIT_MISSING_DENIED` | `AUDIT_COMMIT_MISSING` |
| provider filtering | provider private imports banned outside platform | reject unfiltered provider fact | `PROVIDER_BYPASS_DENIED` | `PROVIDER_FACT_REJECTED` |
| ledger tamper detection | canonical ledger schema | reject invalid root | `LEDGER_TAMPER_DETECTED` | `LEDGER_CHAIN_INVALID` |
| downgrade prevention | production/non-production types separated | reject production downgrade | `PROD_DOWNGRADE_DENIED` | `DOWNGRADE_FORBIDDEN` |
| executor not sole auditor | ledger verifier interface separated | reject executor-only evidence | `EXECUTOR_ONLY_LOG_REJECTED` | `EXECUTOR_ONLY_EVIDENCE_REJECTED` |
| diagnostic not production | test/diagnostic surfaces isolated | reject production overclaim | `DIAGNOSTIC_OVERCLAIM_REJECTED` | `PRODUCTION_OVERCLAIM_REJECTED` |
| secret-safe evidence | redaction and commitment schema | quarantine secret-bearing evidence | `SECRET_EVIDENCE_REJECTED` | `LEDGER_SECRET_MATERIAL_EXPOSURE` |
| release gate required | release decision schema generated | block invalid release claim | `FAIL_OPEN_BLOCKS_RELEASE` | `RELEASE_DECISION_DENIED` |

## 16. Domain-Specific Mapping

### 16.1 SIMD

SIMD MUST consume routing signals from hardware authority only.

SIMD MUST NOT perform CPUID, compiler macro inference, builtin CPU support checks, OS file probing, environment probing, or local timing probes to authorize route selection.

Required evidence:

```text
SIMD_ROUTE_AUTHORIZED_BY_HW_CAPSULE
SIMD_LOCAL_PROBE_ABSENT
SIMD_ROUTE_MATCHES_CAPSULE
SIMD_EXECUTION_LEDGER_COMMITTED
SIMD_DISPATCH_PLAN_BOUND
```

Required negative tests:

```text
SIMD_CPUID_DIRECT_USE_REJECTED
SIMD_BUILTIN_CPU_SUPPORTS_REJECTED
SIMD_AVX2_MACRO_AUTH_REJECTED
SIMD_AVX512_MACRO_AUTH_REJECTED
SIMD_NEON_MACRO_AUTH_REJECTED
SIMD_ROUTE_WITHOUT_HW_CAPSULE_DENIED
SIMD_DISPATCH_TABLE_TAMPER_DENIED
SIMD_EXECUTOR_ONLY_LOG_REJECTED
```

### 16.2 AES

AES production AEAD paths MUST require sealed key or capability authority.

Raw key production paths MUST be quarantined or explicitly non-production.

AES-NI, PCLMULQDQ, ARM crypto, or portable kernels MUST be selected only by authority route, not local probe.

Required evidence:

```text
AES_PRODUCTION_CAPSULE_REQUIRED
AES_RAW_KEY_PRODUCTION_REJECTED
AES_KERNEL_ROUTE_BOUND_TO_CAPSULE
AES_AUDIT_COMMIT_BOUND
AES_KEY_MATERIAL_NOT_IN_EVIDENCE
```

Required negative tests:

```text
AES_RAW_KEY_PRODUCTION_REJECTED
AES_AESNI_LOCAL_PROBE_REJECTED
AES_DEBUG_KEY_PRODUCTION_REJECTED
AES_AUDIT_MISSING_DENIED
AES_KEY_MATERIAL_EVIDENCE_LEAK_REJECTED
AES_UNAUTHORIZED_SCALAR_FALLBACK_DENIED
```

### 16.3 FHE

FHE execution routes, including CKKS, BFV, bootstrap, rotation, key-switch, EvalMod, CoeffToSlot, SlotToCoeff, and BSGS paths, MUST bind to explicit domain capability and policy.

Fixture lanes, public KAT lanes, evidence-only lanes, and diagnostic bootstrap proofs MUST be marked non-production unless a production capsule is issued and admitted through the sovereign chain.

Required evidence:

```text
FHE_OPERATION_CAPSULE_BOUND
FHE_BOOTSTRAP_ROUTE_AUTHORIZED
FHE_FIXTURE_NOT_PRODUCTION
FHE_WITNESS_COMMITMENT_BOUND
FHE_SECRET_KEY_PRODUCTION_PATH_REJECTED
FHE_PRODUCTION_CLAIM_ALLOWED_ONLY_WITH_L5_EVIDENCE
```

Required negative tests:

```text
FHE_FIXTURE_AS_PRODUCTION_REJECTED
FHE_BOOTSTRAP_INCOMPLETE_CLAIM_REJECTED
FHE_SECRET_KEY_PRODUCTION_PATH_REJECTED
FHE_PARTIAL_TRANSFORM_AS_FULL_REJECTED
FHE_MISSING_ROTATION_CORPUS_DENIED
FHE_DIAGNOSTIC_LEDGER_AS_PRODUCTION_REJECTED
FHE_TEST_ADAPTER_IN_PRODUCTION_REJECTED
```

### 16.4 PQC

PQC KEM and signature operations MUST bind production key lifecycle and algorithm policy to sealed capabilities.

External, legacy, weak, unsupported, diagnostic, or debug algorithm lanes MUST be quarantined unless policy explicitly allows diagnostic use.

Required evidence:

```text
PQC_ALGORITHM_POLICY_BOUND
PQC_KEY_CAPSULE_REQUIRED
PQC_LEGACY_RAW_BOUNDARY_QUARANTINED
PQC_AUTHORITY_REPLAY_REJECTED
PQC_DEBUG_KEY_ABSENT
PQC_PRIVATE_KEY_NOT_IN_EVIDENCE
```

Required negative tests:

```text
PQC_DEBUG_KEY_REJECTED
PQC_TEST_KEY_REJECTED
PQC_UNSUPPORTED_SECURITY_LEVEL_DENIED
PQC_LEGACY_ALGORITHM_DIAGNOSTIC_ONLY
PQC_REPLAYED_CAPSULE_REJECTED
PQC_PRIVATE_KEY_EVIDENCE_LEAK_REJECTED
```

### 16.5 Provider Path

Providers may interact with the external world or system only through platform-controlled paths.

Provider facts are not authority until filtered, bound, and admitted by platform policy.

Required evidence:

```text
PROVIDER_FACT_FILTERED
PROVIDER_PRIVATE_IMPORT_BLOCKED
PROVIDER_COMPROMISE_QUARANTINE_PATH_TESTED
PROVIDER_FACT_NOT_AUTHORITY_UNTIL_BOUND
```

Required negative tests:

```text
PROVIDER_PRIVATE_IMPORT_REJECTED
PROVIDER_UNFILTERED_FACT_DENIED
PROVIDER_COMPROMISE_QUARANTINES_DEPENDENTS
PROVIDER_FACT_AS_AUTHORITY_REJECTED
PROVIDER_DOMAIN_DIRECT_ACCESS_REJECTED
```

## 17. Conformance Levels

| Level | Meaning | Production claim allowed |
|---|---|---|
| L0 narrative | Principle documented but not enforceable. | no |
| L1 specified | Normative specification exists. | no |
| L2 compile-guarded | Compile/import/static checks exist and run. | no |
| L3 runtime-enforced | Runtime gates deny invalid authority. | no by itself |
| L4 adversarial-tested | Negative tests prove fail-closed behavior. | limited only if no independent evidence required |
| L5 independently-verifiable | Ledger/witness evidence can be verified outside executor-only truth. | yes for relevant path |

A production sovereign claim requires L5 for the relevant path.

A path below L4 MUST NOT be presented as production sovereign closure.

A path at L4 but without independent verification MUST NOT be described as fully sovereign production evidence.

## 18. Level Promotion Rules

Conformance levels MUST be promoted only by evidence.

```text
L0 -> L1 requires normative specification.
L1 -> L2 requires compile contract and executed static/build guard.
L2 -> L3 requires runtime gate and deterministic failure behavior.
L3 -> L4 requires adversarial negative tests proving fail-closed behavior.
L4 -> L5 requires ledger/witness evidence verifiable outside executor-only truth.
```

Rules:

1. A level MUST NOT be promoted by narrative claim.
2. A level MUST NOT be promoted by one passing smoke test.
3. A level MUST NOT be promoted by diagnostic-only evidence.
4. A level MUST NOT be promoted by target-only evidence if the claim is full production closure.
5. A level MUST NOT be promoted when coverage is unknown.
6. A level MUST NOT be promoted when evidence cannot be independently reviewed.
7. A level MUST NOT be promoted when required failure cases are untested.

## 19. Coverage Result Classes

Allowed enforcement result classes are:

| Result | Meaning | Release effect |
|---|---|---|
| `PASS` | Requirement fully satisfied for declared scope. | may support promotion |
| `PASS_WITH_NON_PRODUCTION_LIMITATION` | Requirement passes only in non-production or diagnostic scope. | no production closure |
| `FAIL_CLOSED` | Invalid condition fails safely. | may support negative evidence |
| `FAIL_OPEN` | Invalid condition executes, bypasses, or overclaims. | release blocker |
| `PARTIAL_COVERAGE` | Some but not all required cases are covered. | no full production claim |
| `UNKNOWN_COVERAGE` | Evidence is insufficient to determine coverage. | release blocker for required production paths |
| `NOT_EXECUTED` | Required test or gate was not run. | blocker unless not applicable |
| `NOT_APPLICABLE_WITH_REASON` | Requirement does not apply and reason is recorded. | allowed only with reviewable reason |

`UNKNOWN_COVERAGE` MUST block production sovereign claims for required authority paths.

`PARTIAL_COVERAGE` MUST be reported as a limitation.

`FAIL_OPEN` MUST block release claims.

## 20. Production Overclaim Prevention

A production overclaim occurs when a report, release note, test result, or evidence bundle describes a path as more complete, more sovereign, more verified, or more production-ready than the evidence supports.

The following are forbidden:

1. A requirement at L1-L4 MUST NOT be described as production sovereign closure.
2. A diagnostic pass MUST NOT be described as production enforcement.
3. A compile-only guard MUST NOT be described as runtime closure.
4. A runtime-only denial MUST NOT be described as independently verifiable.
5. A target-only sanitizer run MUST NOT be described as full-tree sanitizer truth.
6. A smoke test MUST NOT be described as full regression.
7. A fixture proof MUST NOT be described as production proof.
8. A public KAT lane MUST NOT be described as production cryptographic closure.
9. A fail-closed report MUST NOT be described as feature completion.
10. A missing audit path MUST NOT be replaced by executor-only logs.
11. A partial FHE transform proof MUST NOT be described as full bootstrap readiness.
12. A local performance result MUST NOT be described as authority-routed production evidence unless authority mapping verifies.

Any production overclaim MUST produce:

```text
PRODUCTION_OVERCLAIM_REJECTED
```

or an equivalent failure code from the failure semantics specification.

## 21. Release Gate Rules

A release claiming sovereign production closure MUST satisfy:

```text
all required spec IDs >= L5
AND no FAIL_OPEN tests
AND no UNKNOWN_COVERAGE on required production authority paths
AND no unclassified authority bypass path
AND no production raw execution API without capsule admission
AND ledger verification passes
AND authority replay tests pass
AND local probing guards pass
AND executor-only evidence is rejected
AND diagnostic evidence is not counted as production
AND exact blockers are empty or explicitly outside claimed scope
```

A release may claim readiness, foundation status, diagnostic status, or partial closure when some paths are L1-L4, but it MUST state limitations explicitly.

A release MUST NOT claim production sovereign closure if any required path is below L5.

A release MUST NOT claim production sovereign closure if audit verification is unavailable for paths that require audit evidence.

## 22. Release Decision Schema

A release decision SHOULD be machine-readable.

A release decision MUST be semantically equivalent to:

```text
ReleaseDecision {
    release_id;
    source_tree_hash;
    build_fingerprint;
    policy_hash;
    required_spec_ids;
    required_paths;
    required_path_levels;
    fail_open_count;
    unknown_coverage_count;
    partial_coverage_count;
    not_executed_required_count;
    executor_only_evidence_count;
    diagnostic_overclaim_count;
    production_claim_allowed;
    limitations;
    exact_blockers;
    evidence_bundle_id;
    release_gate_signature_or_mac;
}
```

Rules:

1. `production_claim_allowed` MUST be false if `fail_open_count > 0`.
2. `production_claim_allowed` MUST be false if any required path is below L5.
3. `production_claim_allowed` MUST be false if `unknown_coverage_count > 0` for a required production authority path.
4. `production_claim_allowed` MUST be false if required audit evidence is missing.
5. `production_claim_allowed` MUST be false if executor-only evidence is used for a production claim.
6. `production_claim_allowed` MUST be false if diagnostic evidence is counted as production evidence.
7. Exact blockers MUST be listed, not hidden behind a generic failure.

## 23. Required Implementation Hooks

The following hooks SHOULD be implemented after or alongside this formal specification layer:

1. machine-readable spec registry generated from sovereign documents;
2. compile guard mapping forbidden symbols/imports/macros to spec IDs;
3. runtime failure code enum aligned with XPS-SOV-FAIL-004;
4. capsule validation API aligned with XPS-SOV-CAP-003;
5. audit ledger entry serializer/verifier aligned with XPS-SOV-AUD-005;
6. CI evidence bundle that emits `EnforcementEvidence` records;
7. release gate that blocks `FAIL_OPEN` for sovereign production claims;
8. conformance level calculator;
9. production overclaim detector;
10. domain-specific negative test inventory;
11. traceability report generator;
12. exact-blocker report generator.

A production release process SHOULD fail if these hooks are missing for claimed production paths.

## 24. Traceability Matrix

A traceability matrix MUST connect specification requirements to implementation evidence.

A traceability record SHOULD be equivalent to:

```text
TraceabilityRecord {
    spec_id;
    source_document_id;
    requirement_text;
    contract_id;
    implementation_surface;
    owner_module;
    runtime_gate_id;
    test_target;
    evidence_file;
    ledger_entry_id;
    conformance_level;
    release_gate_effect;
}
```

Minimum traceability table:

| Spec requirement | Compile guard | Runtime gate | Test | Evidence | Release gate |
|---|---|---|---|---|---|
| no ambient authority | no direct kernel export | capsule admission required | ambient auth denied | denial record | block if missing |
| no local hw probing | forbidden probe scan | local probe terminates | CPUID probe rejected | forensic record | block if fail-open |
| capsule required | capsule-aware API | no capsule denies | missing capsule denied | NO_CAPSULE record | block if bypass |
| epoch live | epoch input required | stale epoch denied | stale epoch revoked | epoch denial record | block if accepted |
| route binding | route typed | mismatch denied | route mismatch denied | route evidence | block if mismatch accepted |
| audit required | audit root required | missing audit denied | audit missing denied | audit denial | block if executor-only |
| ledger valid | canonical ledger schema | root checked | ledger tamper detected | ledger incident | block if invalid |
| provider filtered | provider private imports banned | unfiltered fact denied | provider bypass denied | provider rejection | block if bypass |
| diagnostic not production | test surface isolated | overclaim denied | fixture overclaim rejected | overclaim record | block if overclaim |
| independent verification | verifier interface separated | executor-only evidence rejected | executor log rejected | verifier commitment | block if executor-only |

Traceability gaps MUST be reported as `UNKNOWN_COVERAGE` unless explicitly not applicable with reason.

## 25. Final Conformance Statement

This mapping is satisfied only when every sovereign claim can be traced from a normative specification to compile contracts, runtime enforcement, adversarial tests, forensic evidence, audit verification, and release gate decision.

Documentation without enforcement is not sovereign closure.

Enforcement without tests is not sovereign closure.

Tests without evidence are not sovereign closure.

Evidence without independent verification is not sovereign production evidence.

A production release claim that cannot be traced through this mapping is non-conforming.

A conforming implementation MUST prove:

```text
XPS_MAP_SPEC_REGISTRY_DEFINED=YES
XPS_MAP_COMPILE_CONTRACTS_DEFINED=YES
XPS_MAP_RUNTIME_GATES_DEFINED=YES
XPS_MAP_FAILURE_CODES_ALIGNED=YES
XPS_MAP_EVIDENCE_SCHEMA_DEFINED=YES
XPS_MAP_EVIDENCE_BUNDLE_DEFINED=YES
XPS_MAP_DOMAIN_NEGATIVE_TESTS_DEFINED=YES
XPS_MAP_CONFORMANCE_LEVELS_DEFINED=YES
XPS_MAP_LEVEL_PROMOTION_EVIDENCE_BASED=YES
XPS_MAP_COVERAGE_CLASSES_DEFINED=YES
XPS_MAP_OVERCLAIM_REJECTED=YES
XPS_MAP_RELEASE_GATE_DEFINED=YES
XPS_MAP_RELEASE_DECISION_SCHEMA_DEFINED=YES
XPS_MAP_TRACEABILITY_MATRIX_DEFINED=YES
```

A release claiming sovereign production closure MUST prove:

```text
XPS_RELEASE_ALL_REQUIRED_PATHS_L5=YES
XPS_RELEASE_FAIL_OPEN_COUNT=0
XPS_RELEASE_UNKNOWN_REQUIRED_COVERAGE=0
XPS_RELEASE_RAW_PRODUCTION_BYPASS=0
XPS_RELEASE_LOCAL_PROBING_BYPASS=0
XPS_RELEASE_EXECUTOR_ONLY_EVIDENCE=0
XPS_RELEASE_DIAGNOSTIC_OVERCLAIM=0
XPS_RELEASE_LEDGER_VERIFICATION_PASSED=YES
XPS_RELEASE_AUTHORITY_REPLAY_TESTS_PASSED=YES
XPS_RELEASE_EXACT_BLOCKERS_NONE_FOR_CLAIMED_SCOPE=YES
```

If these statements cannot be proven, the release MUST NOT claim sovereign production closure.

The XPScerpto runtime enforcement mapping is valid only when specifications become contracts, contracts become gates, gates become tests, tests become evidence, evidence becomes independently verifiable, and release claims are denied unless the full chain is complete.
