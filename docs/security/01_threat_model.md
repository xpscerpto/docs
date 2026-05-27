# XPScerpto Sovereign Threat Model

| Field | Value |
|---|---|
| Document ID | XPS-SOV-TM-001 |
| Status | Normative security specification |
| Scope | platform, platform.api, hardware authority, SIMD, AES, FHE, PQC, provider path, and runtime audit path |
| Layer | Formal Sovereign Specification Layer |
| Version | 1.1 |
| Document type | Normative Security Threat Model |
| Conformance rule | A component that violates a MUST or MUST NOT requirement is non-conforming, even if functional tests pass. |

## Editorial position

This document is written as a security specification, not as a marketing statement. It does not claim absolute security. It defines the authority chain XPScerpto requires, the threats it is designed to prevent or detect, the assumptions it depends on, and the evidence that must exist before a production execution path may be called sovereign.

A decision is sovereign only when it can be reconstructed from declared authority inputs, checked against a sealed authority lineage, and audited after execution by a party that did not perform the original execution. Any execution decision that cannot be reconstructed from declared authority inputs is outside this threat model and must be treated as non-sovereign.

## 1. Purpose

The purpose of this threat model is to turn XPScerpto's sovereignty principle into an auditable security boundary. The document states what XPScerpto attempts to prevent, what it attempts to detect, what it assumes, what it explicitly does not cover, and what a conforming implementation must prove through build gates, runtime admission checks, schemas, ledgers, and negative tests.

This document does not grant production status to any module, path, or subsystem merely because it exists in the source tree, links successfully, or passes a functional test. Production authority must be explicitly issued, scoped, verified, recorded, and independently auditable.

## 2. Normative language

The key words MUST, MUST NOT, REQUIRED, SHALL, SHALL NOT, SHOULD, MAY, and OPTIONAL are normative.

| Term | Meaning in this document |
|---|---|
| MUST / REQUIRED / SHALL | An absolute requirement. A violation makes the component non-conforming. |
| MUST NOT / SHALL NOT | An absolute prohibition. A violation makes the component non-conforming. |
| SHOULD | A strong recommendation. It becomes mandatory only when tied to a MUST requirement elsewhere. |
| MAY / OPTIONAL | Permitted behavior, provided it does not weaken any mandatory requirement. |

A component remains non-conforming if it violates a MUST or MUST NOT requirement even when the violation is introduced to satisfy a legacy test, improve performance, simplify compatibility, or preserve an old phase name.

## 3. Formal definitions

### 3.1 Sovereign decision

A sovereign decision is an execution decision that can be reconstructed and verified from the following minimum elements:

- the active Platform Policy Root;
- the matching Build Fingerprint;
- a live and non-revoked Authority Epoch;
- a valid Sealed Capability Capsule;
- hardware evidence when hardware-dependent routing is involved;
- the applicable domain contract;
- a pre-execution audit reservation; and
- a post-execution witness.

If any of these elements is missing from a production path, the decision is non-sovereign.

### 3.2 Authority input

An authority input is information that XPScerpto accepts as a basis for an execution decision. An authority input is valid only when it is issued through an authorized path, bound to policy, verifiable, auditable, and not derived from local probing inside a domain executor.

### 3.3 Sealed Capability Capsule

A Sealed Capability Capsule is a scoped execution grant. It does not become authority merely because it exists. It is valid only when it is issued by an authorized issuer, cryptographically sealed or equivalently protected, bound to policy, bound to a build, bound to an epoch, bound to a domain, and resistant to replay.

### 3.4 Authority Epoch

An Authority Epoch is the monotonic authority state under which capabilities are issued and validated. It must support revocation, expiry, replay rejection, and liveness checking at admission time.

### 3.5 Build Fingerprint

A Build Fingerprint is the security identity of a build. It must bind source material, module interfaces, module implementations, import graphs, compiler and build configuration, compile contracts, policy guards, and generated artifacts. A file hash alone is not sufficient.

### 3.6 Audit evidence

Audit evidence is verifiable evidence of a decision, denial, execution, or failure. Executor-local logs are not sovereign audit evidence unless they are bound into an append-only, tamper-evident, externally verifiable ledger.

### 3.7 Non-sovereign execution

Non-sovereign execution is any execution that does not pass the authority, admission, and audit requirements in this document. Production domains must deny non-sovereign execution. Diagnostic execution may exist only if it is explicitly marked non-production and cannot elevate production flags.

## 4. Protected assets

XPScerpto protects the following assets.

| Asset | Security requirement |
|---|---|
| Platform Policy Root | The policy root hash and its interpretation must remain stable for a specific build and authority epoch pair. |
| Build Fingerprint | Build identity must bind modules, compile contracts, import boundaries, policy guards, and generated artifacts. |
| Hardware Evidence | Hardware capability claims must originate from the authorized hardware authority path, not from local domain probing. |
| Authority Epoch | Epoch state must be monotonic, revocable, and bound to capability issuance. |
| Sealed Capability Capsule | Execution rights must be explicit, sealed, scoped, and replay-resistant. |
| Runtime Execution Decision | Execution may occur only after capsule, policy, epoch, domain, route, and audit validation. |
| Audit Ledger | Execution evidence must be append-only, tamper-evident, and externally verifiable. |
| Import Boundaries | Module import boundaries must prevent unauthorized authority access and capability fabrication. |
| Provider Facts | Provider facts must not become authority until filtered, bound, and cross-checked by platform policy. |
| Domain Kernel Route | The selected route and kernel class must match the authority capsule and policy. |

## 5. Sovereignty boundary

The sovereignty boundary begins at the Platform Policy Root and ends at the post-execution audit commitment. It includes policy interpretation, authority issuance, provider fact filtering, hardware evidence ingestion, capsule sealing, runtime admission, domain route validation, kernel execution, pre-execution audit reservation, post-execution witness capture, and external audit verification.

The boundary does not mean that every physical event in the machine is trusted. It means XPScerpto declares which evidence it accepts, which authority chain it requires, and how it must fail when evidence is missing, stale, malformed, or contradictory.

## 6. Trust boundaries

| Boundary | Allowed crossing | Rejected crossing |
|---|---|---|
| External world to platform | Provider-mediated facts filtered by platform policy | Direct domain access to OS, firmware, CPU probing, or environment probing |
| Platform to platform.api | Filtered, policy-bound authority facts | Raw platform internals, provider internals, ABI authority, or runtime private authority |
| platform.api to hardware authority | Policy-bound hardware evidence request and response | Local hardware inference by SIMD, AES, FHE, or PQC |
| Hardware authority to domain bridges | Sealed routing facts and decision capsules | Mutable, unsealed capability flags |
| Domain bridge to execution kernel | Capsule-authorized execution route | Kernel self-selection from probes, target macros, timing, or local caches |
| Execution to audit ledger | Witness commitment and execution evidence | Executor-only logs that cannot be independently verified |
| Tests and adapters to production | Explicitly marked test-only diagnostic surfaces | Leakage of legacy adapters, fixtures, or phase symbols into production authority |

## 7. Formal adversaries

A conforming implementation must either prevent each adversary objective or produce deterministic fail-closed evidence.

| Adversary | Capability | Objective | Required XPScerpto response |
|---|---|---|---|
| Malicious runtime module | Runs in the process and attempts to exceed its declared authority | Acquire execution rights without a valid capsule, bypass audit, or call private authority surfaces | Deny execution, quarantine if policy allows, and emit forensic evidence |
| Unauthorized hardware probing | A domain attempts CPUID, compiler builtins, OS probes, environment probes, or local inference | Self-authorize fast paths without hardware authority | Reject the build if detectable statically; terminate or safe-halt if detected at runtime |
| Forged capability capsule | Attacker creates or mutates a capsule without issuer authority | Cause unauthorized execution | Reject before execution and record forgery evidence when the audit path is available |
| Replayed authority chain | Attacker reuses a once-valid capsule or lineage under a later epoch, build, or policy | Re-enable stale execution rights | Reject by epoch, expiry, capsule identifier, policy hash, build fingerprint, and ledger binding |
| Runtime policy bypass | Runtime invokes a kernel without policy admission | Execute outside the declared policy | Deny execution and bind the denial to audit evidence |
| SIMD dispatch manipulation | Attacker changes route class, dispatch plan, or kernel selection | Force unauthorized upgrade, downgrade, or silent route substitution | Validate the route against the capsule and reject mismatches |
| Tampered audit record | Attacker modifies, deletes, reorders, or fabricates ledger entries | Hide execution or fabricate compliance | Detect through hash chaining, Merkle commitments, and external checkpoints |
| Compromised provider path | Provider returns malicious, stale, malformed, or overly broad facts | Poison platform policy or hardware evidence | Filter, bind, and cross-check provider facts; unverified facts must not become authority |
| Epoch TOCTOU attacker | Attacker races validation against epoch revocation or transition | Execute with a capsule validated under an obsolete epoch | Bind admission to an epoch snapshot and re-check liveness before execution |
| Resource exhaustion adversary | Floods capsules, ledger reservations, provider facts, or large domain requests | Force partial execution or bypass normal denial paths | Apply quota-bound deterministic denial with audit evidence |
| Supply-chain adversary | Uses stale generated files, malicious build options, hidden whitelists, or divergent module modes | Weaken build identity or import policy | Bind build state into the fingerprint and fail closed on mismatch |
| Test-surface leakage adversary | Moves fixture, adapter, or historical phase surfaces into production | Reintroduce non-production authority | Reject production imports and forbidden symbols through scans and build gates |

## 8. Security assumptions

These assumptions are explicit. If an assumption fails, XPScerpto cannot claim the related security property without an additional mitigation layer.

| Assumption | Meaning | Consequence if false |
|---|---|---|
| Compiler integrity before Build Root | The compiler and toolchain are not malicious before the build root is established | The Build Fingerprint cannot be trusted as binding evidence |
| Signing authority integrity | The issuer protects signing material and issuance policy | Capsule non-forgery may fail |
| Bounded hardware honesty | Hardware evidence reflects a declared interface and is not maliciously fabricated below the attestation boundary | Hardware evidence may be false and route authorization becomes unreliable |
| Cryptographic primitive assumptions | Hashes, signatures, MACs, commitments, and KDFs meet their declared security level | Capsule and ledger integrity may fail |
| Deterministic module build | Inputs, options, generated files, and import surfaces are reproducible enough to bind to the Build Fingerprint | Independent replay verification may fail |
| Policy root availability | Runtime can access the policy root hash or a sealed policy descriptor during admission | Execution must deny or safe-halt |
| Monotonic epoch source | Epoch state cannot be rolled back silently within the declared boundary | Replay and revocation guarantees may fail |
| Audit checkpoint availability | External checkpointing is available or its absence is explicitly declared | Audit sovereignty remains incomplete |

## 9. Explicit system limits

The following are outside the declared protection boundary unless a future document defines a mitigation.

| Limit | Boundary statement |
|---|---|
| Physical invasive attacks | Physical extraction, invasive probing, fault injection, and physical side-channel attacks are not fully covered. |
| Malicious CPU vendor collusion | Deliberate fabrication of architectural evidence by a CPU vendor below the hardware honesty boundary is outside this model. |
| Speculative execution beyond declared mitigations | Speculative execution attacks not covered by explicit mitigation flags or hardened kernels are outside this model. |
| Compromised firmware before attestation | Firmware compromise before the evidence boundary can invalidate hardware evidence. |
| DRAM remanence | Memory remanence is outside scope unless encrypted memory and verified wipe contracts are explicitly enabled. |
| Malicious compiler after trusted build statement | A compiler that intentionally emits code violating source-level contracts can invalidate Build Fingerprint semantics. |
| Complete side-channel security | Side-channel completeness is not claimed without a domain-specific leakage model and test contract. |
| Third-party certification | This document is not a third-party production certification. |

## 10. Required security properties

### 10.1 No ambient authority

No module receives execution authority merely because it is linked, imported, compiled, loaded, placed in the repository, associated with a historical phase, or faster than an alternative. Authority must be explicitly issued and validated.

### 10.2 No local capability fabrication

A domain module must not derive execution authority from local CPU probes, compiler target macros, compiler builtins, OS inspection, environment variables, timing probes, cached out-of-band facts, or platform-specific probe APIs.

### 10.3 Fail-closed admission

If any required authority predicate is missing, invalid, stale, unverifiable, or contradictory, the runtime must deny execution or safe-halt according to the applicable failure semantics.

### 10.4 Evidence before execution

A production execution path must not run unless it can reserve an audit ledger entry before execution. Execution before audit reservation is non-conforming.

### 10.5 Independent auditability

The executor must not be the sole auditor. Evidence must be verifiable by a role that does not require execution authority for the original decision.

### 10.6 No production elevation from diagnostic paths

A test harness, adapter, fixture, diagnostic path, or legacy compatibility surface must not elevate production claims. In particular, it must not set or justify production-ready, bootstrap-complete, full-transform, or domain-sovereign flags.

## 11. Minimum capsule schema

A production Sealed Capability Capsule must contain at least the following fields:

```text
capsule_id
issuer_id
issuer_policy_hash
issuer_key_id
subject_domain
subject_component
allowed_route_class
allowed_kernel_class
allowed_instruction_family
allowed_memory_contract
policy_root_hash
build_fingerprint
authority_epoch
epoch_expiry
hardware_evidence_hash
domain_contract_hash
audit_parent_root
nonce
issued_at
expires_at
replay_window_id
capability_scope
failure_policy
signature_or_mac
```

### 11.1 Capsule rules

- A capsule missing a required field must be rejected before execution.
- A capsule with an invalid issuer must be rejected.
- A capsule that does not match the active policy root must be rejected.
- A capsule that does not match the active Build Fingerprint must be rejected.
- An expired or revoked capsule must be rejected.
- A capsule that does not match the domain contract must be rejected.
- A capsule without audit parent binding must be rejected in production.
- A capsule signed by a test or debug key must be rejected in production.
- A capsule that grants a route broader than policy permits must be rejected.
- A capsule that can be replayed must be rejected.

## 12. Runtime admission state machine

A production runtime must follow this sequence:

```text
1. Load sealed capsule
2. Verify capsule structure
3. Verify issuer authority
4. Verify signature_or_mac
5. Verify policy_root_hash
6. Verify build_fingerprint
7. Verify authority_epoch monotonicity
8. Verify authority_epoch liveness
9. Verify expiry and replay window
10. Verify hardware_evidence_hash
11. Verify domain_contract_hash
12. Verify allowed_route_class
13. Verify allowed_kernel_class
14. Verify kernel_hash when applicable
15. Reserve audit ledger entry
16. Bind the decision to the epoch snapshot
17. Re-check epoch liveness at admission
18. Execute the authorized kernel
19. Capture post-execution witness
20. Commit the audit ledger transition
21. Publish or checkpoint the audit root
```

Execution before step 15 is non-conforming. Any failure before step 18 must deny execution or safe-halt. Any failure after step 18 must produce a post-execution failure witness.

## 13. Build Fingerprint schema

A production Build Fingerprint must include at least:

```text
source_tree_hash
module_interface_hashes
module_implementation_hashes
public_module_graph_hash
private_module_graph_hash
import_boundary_policy_hash
compiler_id
compiler_version
compiler_target
cmake_version
cmake_generator
cxx_standard
native_or_fallback_modules_mode
exceptions_flag
rtti_flag
asan_flag
ubsan_flag
tsan_flag
lto_flag
werror_flag
platform_policy_guard_config
provider_policy_config
domain_guard_config
generated_files_hash
test_adapter_exclusion_hash
forbidden_symbol_scan_hash
build_epoch
```

### 13.1 Build Fingerprint rules

- A file hash alone is not enough.
- Compile flags must be part of the fingerprint.
- The module import graph must be part of the fingerprint.
- Generated artifacts must be bound to the fingerprint.
- Native and fallback module modes must be distinguished.
- Exception and RTTI policy must be bound to the fingerprint.
- A fingerprint mismatch must prevent production admission.

## 14. Audit Ledger schema

A production audit ledger entry must contain at least:

```text
ledger_entry_id
parent_entry_hash
policy_root_hash
build_fingerprint
authority_epoch
capsule_id
issuer_id
domain
component
route_class
kernel_class
kernel_hash
hardware_evidence_hash
domain_contract_hash
decision_hash
pre_execution_witness
post_execution_witness
result_code
failure_code
monotonic_counter_or_timestamp
external_checkpoint_hash
entry_hash
```

### 14.1 Audit Ledger rules

- Executor-local logs are not audit evidence.
- A ledger entry without a parent hash is not append-only evidence.
- A ledger that is not tamper-evident does not prove compliance.
- A ledger that cannot be externally verified is insufficient for a sovereign claim.
- Deletion, modification, or reordering of entries must result in LEDGER_CHAIN_INVALID.
- In production, a missing audit commitment means denial.

## 15. Import boundary enforcement matrix

| Source | Forbidden imports or access |
|---|---|
| SIMD | platform.abi, platform.runtime, provider internals, OS probes, hardware internals, CPUID, compiler authorization macros |
| AES | platform internals, provider internals, local CPU probes, environment capability overrides |
| FHE | platform internals, provider internals, local CPU probes, direct SIMD authority, test bootstrap adapters |
| PQC | platform internals, provider internals, local CPU probes, debug issuers |
| platform.api | provider private internals, raw ABI authority, runtime private authority |
| provider | domain kernels, domain private state, capability issuer internals |
| tests and adapters | production authority exports, production include leakage, policy elevation symbols |
| runtime audit | mutable executor-only logs, unsigned ledger transitions |

Any forbidden import or forbidden authority access detected by static scan must fail configure or build.

## 16. Static and runtime detection requirements

### 16.1 Static detection must scan for

```text
__builtin_cpu_supports
__cpuid
__get_cpuid
cpuid.h
/proc/cpuinfo
getauxval
AT_HWCAP
sysctl CPU probes
IsProcessorFeaturePresent
compiler ISA macros used for authorization
std::hardware_destructive_interference_size
environment-variable capability override
direct provider private imports
platform.abi imports outside platform
platform.runtime imports outside platform
test adapter imports from production
legacy authority symbol reintroduction
hidden whitelist exceptions
```

### 16.2 Runtime detection must trap or deny

```text
unauthorized route mutation
capsule mismatch
epoch revocation race
audit reservation failure
kernel hash mismatch
domain bridge bypass
provider fact elevation without binding
debug issuer use in production
capsule replay
ledger parent mismatch
```

## 17. Failure semantics

| Failure class | Production behavior | Diagnostic behavior |
|---|---|---|
| Missing capsule | Deny | Deny or record diagnostic evidence |
| Invalid capsule | Deny and record evidence | Deny and record evidence |
| Missing policy root | Safe-halt or deny | Diagnostic record allowed |
| Build Fingerprint mismatch | Deny | Deny |
| Stale epoch | Deny | Deny |
| Epoch TOCTOU | Deny with EPOCH_TOCTOU_REJECTED | Deny with evidence |
| Missing hardware evidence | Deny or safe-halt | Explicit non-production downgrade |
| Local probe detected | Build reject or terminate | Build reject or terminate |
| Route mismatch | Deny | Deny |
| Missing audit commitment | Deny | Diagnostic-only execution allowed if clearly marked |
| Provider fact unverified | No authority elevation | Diagnostic-only use |
| Ledger tamper | Mark chain invalid | Mark chain invalid |
| Test adapter leakage | Build reject | Build reject |
| Debug issuer in production | Deny | Allowed only when explicitly diagnostic |

## 18. Signing and sealing key lifecycle

A production implementation must define the issuer identity, key generation authority, key storage boundary, key rotation policy, key revocation policy, compromise response, issuer policy binding, production and debug issuer separation, expired-key rejection, and test-key rejection.

The following rules are mandatory:

- Test signing keys must not be accepted in production.
- Debug issuers must not be accepted in production.
- Expired issuer keys must be rejected.
- The provider path must not issue production capability capsules directly.
- A domain executor must not possess a production capability-issuance key.

## 19. Supply-chain and build threats

| Threat | Required control |
|---|---|
| Stale generated files | Bind generated files to the Build Fingerprint. |
| Malicious CMake option | Include build and toolchain flags in the fingerprint. |
| Native/fallback module divergence | Record and validate the active module mode. |
| Test adapter leakage | Reject production imports from adapters or fixtures. |
| Legacy symbol reintroduction | Scan for forbidden symbols and fail closed. |
| Hidden whitelist | Require a whitelist-free authority policy. |
| Stale build cache | Bind build epoch and cache provenance. |
| Partial target build overclaim | Prohibit claims before full materialization. |
| Target-only sanitizer overclaim | Separate target sanitizer evidence from full-tree sanitizer truth. |

## 20. Side-channel posture

This document does not claim complete side-channel security. A domain that wants to make a side-channel claim must publish a domain-specific contract that defines:

```text
constant_time_contract
memory_access_contract
branch_behavior_contract
cache_behavior_contract
timing_leakage_model
secret_dependent_trace_policy
test_coverage
negative_tests
failure_semantics
```

Without such a contract, side-channel completeness remains outside the claim.

## 21. Resource exhaustion

Resource exhaustion must fail closed and remain auditable.

| Attack | Required response |
|---|---|
| Capsule flood | Quota-bound denial |
| Ledger flood | Bounded reservation failure |
| Provider fact explosion | Filtered rejection |
| Oversized FHE bootstrap request | Admission denial |
| Excessive rotation corpus | Bounded planner denial |
| SIMD route stress | Deterministic route rejection |
| Audit storage exhaustion | Production denial with no silent execution |

Resource exhaustion must not produce partial authority elevation.

## 22. Evidence privacy and secret safety

Audit evidence must be sufficient for verification, but it must not disclose plaintext secrets, private keys, raw user data, secret-dependent traces, FHE secret-key material, PQC private-key material, AES key material, or debug-only secrets.

Evidence that contains sensitive material must be classified as diagnostic non-production evidence and must not be used to support a production claim.

## 23. Threat-to-control matrix

| Threat | Primary control | Secondary control | Evidence artifact |
|---|---|---|---|
| Local hardware probing | Compile guard, import scan, runtime trap | Domain bridge route validation | UNAUTHORIZED_PROBE_ATTEMPT |
| Capsule forgery | Signature or MAC validation | Issuer policy hash binding | CAPSULE_AUTH_INVALID |
| Authority replay | Epoch monotonicity, expiry, capsule id uniqueness | Ledger parent root binding | AUTHORITY_REPLAY_REJECTED |
| SIMD dispatch manipulation | Route class comparison against capsule | Kernel hash binding | ROUTE_MISMATCH_REJECTED |
| Audit tamper | Hash chain and Merkle root | External checkpoint verification | LEDGER_CHAIN_INVALID |
| Provider compromise | Platform provider filtering | Evidence source cross-check | PROVIDER_FACT_REJECTED |
| Epoch TOCTOU race | Validation and admission epoch snapshot | Final epoch liveness check | EPOCH_TOCTOU_REJECTED |
| Debug issuer misuse | Issuer class validation | Production/debug separation | DEBUG_ISSUER_REJECTED |
| Test adapter leakage | Production import scan | Forbidden symbol scan | TEST_SURFACE_LEAK_REJECTED |
| Resource exhaustion | Quota-bound admission | Deterministic denial | RESOURCE_BOUND_REJECTED |
| Missing audit commitment | Pre-execution ledger reservation | Production denial | AUDIT_COMMIT_MISSING |
| Build Fingerprint mismatch | Fingerprint validation | Module graph validation | BUILD_FINGERPRINT_MISMATCH |

## 24. Minimum conformance tests

A conforming XPScerpto build must provide tests or policy scans covering at least the following cases.

| Test class | Required result |
|---|---|
| Forged capsule | Execution denied |
| Stale epoch capsule | Capsule revoked or execution denied |
| Missing hardware evidence | Production safe-halt or explicit non-production downgrade |
| Local probe attempt | Build rejection or runtime termination |
| Route mismatch | Execution denied |
| Missing audit commitment | Production execution denied |
| Ledger tamper | Verification failure |
| Provider private import outside platform | Build rejection |
| Debug issuer in production | Denied |
| Test adapter imported by production | Build rejection |
| Build Fingerprint mismatch | Admission denied |
| Replayed capsule id | Denied |
| Epoch TOCTOU race | Denied |
| Oversized FHE request | Bounded denial |
| Unauthorized SIMD route | Denied |
| Hidden whitelist exception | Build rejection |
| Production flag elevation from diagnostic path | Rejected |

## 25. Conformance obligations

Every security requirement must be traceable to an identifier, normative statement, static enforcement method, runtime enforcement method, evidence artifact, minimum test, and failure behavior.

| ID | Requirement | Static enforcement | Runtime enforcement | Evidence | Minimum test |
|---|---|---|---|---|---|
| XPS-SOV-REQ-001 | Domains must not perform local capability probing | Import and token scan | Runtime trap | UNAUTHORIZED_PROBE_ATTEMPT | Local probe attempt |
| XPS-SOV-REQ-002 | A capsule is required before execution | API guard scan | Admission denial | CAPSULE_MISSING | Missing capsule |
| XPS-SOV-REQ-003 | Capsule authenticity is required | Issuer and key validation | Denial | CAPSULE_AUTH_INVALID | Forged capsule |
| XPS-SOV-REQ-004 | Epoch must be live | Epoch binding scan | Liveness re-check | EPOCH_STALE_REJECTED | Stale epoch |
| XPS-SOV-REQ-005 | Route must match capsule | Route class contract | Route validation | ROUTE_MISMATCH_REJECTED | Route mismatch |
| XPS-SOV-REQ-006 | Audit must precede execution | Ledger API guard | Deny if no reservation | AUDIT_COMMIT_MISSING | Missing audit path |
| XPS-SOV-REQ-007 | Build Fingerprint must bind admission | Fingerprint generation | Admission match | BUILD_FINGERPRINT_MISMATCH | Modified build |
| XPS-SOV-REQ-008 | Provider facts are not authority by default | Provider boundary scan | Fact filtering | PROVIDER_FACT_REJECTED | Malformed provider fact |
| XPS-SOV-REQ-009 | Test adapters are forbidden in production | Import and symbol scan | Production admission impossible | TEST_SURFACE_LEAK_REJECTED | Adapter leakage |
| XPS-SOV-REQ-010 | Diagnostic paths cannot elevate production | Flag scan | Flag denial | PRODUCTION_OVERCLAIM_REJECTED | Fake production flag |
| XPS-SOV-REQ-011 | Ledger must be tamper-evident | Schema validation | Chain verification | LEDGER_CHAIN_INVALID | Tampered ledger |
| XPS-SOV-REQ-012 | Resource exhaustion must fail closed | Quota config scan | Bounded denial | RESOURCE_BOUND_REJECTED | Oversized request |

## 26. Conformance levels

| Level | Name | Requirements |
|---|---|---|
| L0 | Diagnostic Only | Tests and diagnostics only; no production claim |
| L1 | Build Sovereignty | Import and build guards are closed; forbidden symbols are rejected |
| L2 | Runtime Admission Sovereignty | Capsule, policy, epoch, route, and domain admission are enforced |
| L3 | Audit Sovereignty | Ledger is tamper-evident and externally verifiable |
| L4 | Domain Sovereignty | SIMD, AES, FHE, and PQC do not fabricate local authority |
| L5 | Production Candidate | L1 through L4 plus full regression, sanitizer truth, and no diagnostic overclaim |

No component may claim L5 if the full-tree build is incomplete, regression materialization is incomplete, sanitizer runtime evidence is missing or unclean, production flags are raised by diagnostic paths, test surfaces leak into production, or the audit path cannot be externally verified.

## 27. Domain-specific rules

### 27.1 SIMD

SIMD must not perform CPUID locally, use compiler macros for ISA authorization, read /proc/cpuinfo, call compiler CPU-support builtins, choose a kernel from a local probe, or modify the dispatch plan after capsule validation.

SIMD must receive routing authority from the hardware authority path, validate the allowed kernel class, bind kernel selection to the capsule, and emit audit evidence for route rejection.

### 27.2 AES

AES must not select AES-NI or fallback routes through local probing, rely on environment variables for capability, or make side-channel claims without an independent domain contract.

AES must use authority-routed capabilities, reject route mismatch, and prevent key material from leaking into audit evidence.

### 27.3 FHE

FHE must not use test bootstrap adapters in production, raise FHE_BOOTSTRAP_COMPLETE from a fixture or diagnostic path, use secret-key material in a production pipeline outside an explicit contract, raise full-transform flags without independent production closure, or treat partial materialization as full regression.

FHE must bind bootstrap admission to policy, capsule, and audit evidence; reject missing rotation corpus when required by the production route; reject CapabilityIncomplete in production; and keep fixture proof separate from production proof.

### 27.4 PQC

PQC must not use debug or test key material in production, accept provider facts as authority directly, or make a general quantum-resistance claim without primitive-level declaration.

PQC must protect private-key material from audit leakage, bind the algorithm suite to the policy root, and reject unsupported or security-none targets in production.

### 27.5 Provider path

A provider must not issue authority directly, bypass platform filtering, export private facts to domains, or own a domain kernel route.

A provider may supply facts only. Those facts must be filtered by platform policy, assigned an evidence source class, and rejected when stale, malformed, or unbound.

## 28. Non-claims

This threat model does not claim that XPScerpto is unbreakable, side-channel complete, formally verified in a theorem prover, quantum-resistant for every primitive, certified for production by a third party, protected against malicious CPU vendor collusion, protected against a malicious compiler beyond the trusted build boundary, production-ready because the build succeeds, complete because a fixture succeeds, or performance-dominant because a benchmark succeeds.

It claims only that, when implemented and tested according to this document, XPScerpto has a declared authority chain, explicit failure semantics, verifiable capsule admission, Build Fingerprint binding, auditable evidence structures, enforceable import boundaries, deterministic enforcement obligations, no ambient authority, no local capability fabrication, and a clear separation between diagnostic and production paths.

## 29. Final conformance decision

A component conforms to this document only if it can prove all of the following:

```text
XPS_SOV_POLICY_ROOT_BOUND=YES
XPS_SOV_BUILD_FINGERPRINT_BOUND=YES
XPS_SOV_CAPSULE_SCHEMA_ENFORCED=YES
XPS_SOV_RUNTIME_ADMISSION_STATE_MACHINE_ENFORCED=YES
XPS_SOV_EPOCH_LIVENESS_CHECKED=YES
XPS_SOV_AUDIT_BEFORE_EXECUTION=YES
XPS_SOV_LEDGER_TAMPER_EVIDENT=YES
XPS_SOV_IMPORT_BOUNDARIES_ENFORCED=YES
XPS_SOV_LOCAL_PROBING_REJECTED=YES
XPS_SOV_PROVIDER_FACTS_FILTERED=YES
XPS_SOV_TEST_ADAPTERS_EXCLUDED_FROM_PRODUCTION=YES
XPS_SOV_DIAGNOSTIC_PATH_CANNOT_ELEVATE_PRODUCTION=YES
XPS_SOV_FAILURES_FAIL_CLOSED=YES
```

Any of the following states prohibits a production claim:

```text
XPS_SOV_MISSING_CAPSULE=YES
XPS_SOV_INVALID_CAPSULE_ACCEPTED=YES
XPS_SOV_EPOCH_STALE_ACCEPTED=YES
XPS_SOV_LOCAL_PROBE_ALLOWED=YES
XPS_SOV_AUDIT_COMMIT_MISSING=YES
XPS_SOV_LEDGER_TAMPER_UNDETECTED=YES
XPS_SOV_PROVIDER_FACT_ELEVATED_WITHOUT_BINDING=YES
XPS_SOV_TEST_ADAPTER_IN_PRODUCTION=YES
XPS_SOV_DIAGNOSTIC_OVERCLAIM=YES
XPS_SOV_BUILD_FINGERPRINT_MISMATCH_ACCEPTED=YES
```

## 30. Normative conclusion

Sovereignty in XPScerpto is not a slogan, a benchmark result, a build artifact, or a phase label. It is a security property that exists only when authority is declared, sealed, scoped, validated, bound to an epoch, bound to policy, bound to a build, bound to a domain, bound to audit evidence, and independently verifiable.

Any execution path that does not pass through that chain is non-sovereign. Any claim that lacks the evidence required by this document is rejected. Any failure that does not fail closed violates the sovereignty boundary. Any diagnostic, test, adapter, fixture, or legacy surface that elevates a production claim violates this specification.
