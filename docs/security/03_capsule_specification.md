# XPScerpto Sealed Capability Capsule Specification

**Document ID:** XPS-SOV-CAP-003  
**Status:** Normative execution specification  
**Scope:** capability issuance, serialization, validation, lifecycle, revocation, replay prevention, audit binding, domain-scoped execution authority  
**Version:** 1.1  
**Document Type:** Normative Security and Runtime Admission Specification  
**Applies To:** platform, platform.api, hardware authority, SIMD, AES, FHE, PQC, provider-mediated evidence paths, runtime admission, audit ledger integration

---

## 1. Purpose

The Sealed Capability Capsule is the executable root of XPScerpto sovereignty. It is the object that transforms policy, build identity, hardware evidence, authority epoch state, domain contracts, and audit commitment into a narrow runtime execution right.

A capsule is not a convenience token. It is not a debug switch, not a route hint, not a performance selector, and not a general permission object. It is a cryptographically sealed authorization object that binds a specific execution right to a specific policy, build, epoch, domain, evidence set, and audit path.

A runtime execution path that requires authority MUST NOT infer authority from ambient state. In particular, authority MUST NOT be inferred from module import status, link status, compile target flags, local hardware probes, environment variables, cached runtime observations, benchmark results, test fixtures, or legacy compatibility paths.

A protected execution path is conforming only when it consumes a validated sealed capsule and the capsule is admitted through the validation rules defined in this specification.

## 2. Normative Language

The terms **MUST**, **MUST NOT**, **REQUIRED**, **SHALL**, **SHALL NOT**, **SHOULD**, **MAY**, and **OPTIONAL** are normative.

A component that violates a **MUST** or **MUST NOT** requirement is non-conforming even if functional tests pass.

A production implementation MUST treat this document as an execution specification, not as advisory guidance.

## 3. Normative References

This specification is part of the XPScerpto sovereign specification layer.

The following documents are normative companions:

| Document | Role |
|---|---|
| XPS-SOV-TM-001 — Sovereign Threat Model | Defines the threat boundary, adversaries, protected assets, and non-claims. |
| XPS-SOV-AUTH-002 — Sovereign Authority Model | Defines authority lineage, epoch transition, capsule admission, revocation, and safe halt behavior. |
| XPS-SOV-CAP-003 — Sealed Capability Capsule Specification | Defines the sealed capsule format, lifecycle, validation, and execution admission binding. |
| XPS-SOV-AUD-004 — Audit Ledger Specification | Defines audit entry structure, tamper evidence, checkpointing, and external verification. |
| XPS-SOV-FAIL-005 — Failure Semantics | Defines deterministic denial, safe halt, quarantine, and diagnostic behavior. |

If this specification conflicts with the Threat Model or Authority Model, the stricter authority-denying interpretation MUST apply.

## 4. Core Security Principle

A capsule grants no ambient authority. A capsule authorizes only the exact execution class it names, under the exact context it binds, during the exact validity window it declares, and only if the runtime can bind the execution decision to audit evidence before protected execution begins.

The following equation is normative:

```text
execution_authority :=
    valid_capsule
AND active_policy_match
AND active_build_match
AND live_epoch
AND platform_authorized_hardware_evidence_match
AND domain_contract_match
AND route_class_admitted
AND kernel_class_admitted
AND audit_commitment_available
AND no_replay_detected
```

If any predicate is false, production execution MUST be denied.

## 5. Capsule Security Goals

A sealed capability capsule exists to provide the following properties:

| Goal | Required property |
|---|---|
| Narrow authority | The capsule grants only a scoped execution right, not general domain authority. |
| Context binding | The capsule is bound to policy, build, epoch, hardware evidence, domain contract, and audit state. |
| Replay resistance | The capsule cannot be reused outside its declared uniqueness, epoch, validity, and audit constraints. |
| Domain isolation | A capsule issued for one domain cannot be consumed by another domain. |
| Route integrity | Requested route and kernel class must match the capsule. |
| Auditability | A production capsule must bind execution to audit evidence. |
| Downgrade resistance | Older parsers, weaker algorithms, missing audit paths, or legacy admission paths cannot weaken production admission. |
| Failure determinism | Every validation failure must map to a deterministic failure code. |

## 6. Capsule Structure

A production capsule MUST contain fields semantically equivalent to the following structure:

```text
CapsuleHeader {
    u32      version;
    u32      capsule_class;
    u128     capsule_id;
    u64      authority_epoch;
    u64      replay_window_id;

    hash256  build_fingerprint;
    hash256  policy_hash;
    hash256  issuer_policy_hash;
    hash256  hardware_evidence_hash;
    hash256  domain_contract_hash;

    u32      execution_domain;
    u32      subject_component;
    u32      allowed_route_class;
    u32      allowed_kernel_class;
    u32      allowed_instruction_family;
    u32      allowed_memory_contract;

    hash256  audit_parent_root;
    u128     audit_reservation_id;
    u32      audit_commitment_policy;
    hash256  pre_execution_witness_hash;

    u64      issued_at;
    u64      not_before;
    u64      expiry;
    u32      time_or_counter_source;

    u32      critical_flags;
    u32      non_critical_flags;
    u32      extension_policy;
    hash256  extension_block_hash;

    u32      issuer_id;
    u32      issuer_key_id;
    u32      signature_algorithm;
    u32      context_string_id;
    bytes    nonce;
    bytes    capability_scope;
    u32      failure_policy;

    bytes    signature;
}
```

This structure defines required semantics, not a mandatory ABI layout. Implementations MAY use a different binary layout only if canonical serialization produces the same semantic fields and validation behavior.

A production implementation MUST NOT omit required semantics by claiming ABI freedom.

## 7. Required Field Semantics

| Field | Required semantics |
|---|---|
| `version` | Capsule schema version. Unknown production versions MUST be rejected unless active policy explicitly declares compatibility. |
| `capsule_class` | Declares production, non-production, diagnostic, migration, or other policy-defined class. |
| `capsule_id` | Globally unique or issuer-unique replay identifier. It MUST be bound into the signature. |
| `authority_epoch` | Epoch in which the capsule may be admitted. It MUST match a live epoch snapshot. |
| `replay_window_id` | Replay-prevention window or ledger-ordering namespace. |
| `build_fingerprint` | Build identity allowed to consume this capsule. |
| `policy_hash` | Active policy root under which the capsule was issued. |
| `issuer_policy_hash` | Policy hash governing the issuer’s authority. |
| `hardware_evidence_hash` | Hash of platform-authorized hardware evidence used for route authorization. |
| `domain_contract_hash` | Hash of the domain execution contract required for admission. |
| `execution_domain` | Domain allowed to consume the capsule, such as SIMD, AES, FHE, PQC, or platform-controlled domain. |
| `subject_component` | Specific component, bridge, or runtime surface allowed to consume the capsule. |
| `allowed_route_class` | Narrow route class allowed for execution. |
| `allowed_kernel_class` | Narrow kernel class allowed for execution. It MUST NOT be interpreted as all kernels in a domain. |
| `allowed_instruction_family` | Instruction family or abstract execution family admitted by policy. |
| `allowed_memory_contract` | Memory safety, isolation, alignment, and access contract required for the route. |
| `audit_parent_root` | Parent audit ledger root or checkpoint root to which this execution will bind. |
| `audit_reservation_id` | Pre-execution audit reservation identifier. |
| `audit_commitment_policy` | Declares whether audit is mandatory, ordered, one-time, checkpointed, or diagnostic-only. |
| `pre_execution_witness_hash` | Hash of the pre-execution witness or admission record. |
| `issued_at` | Issuance timestamp, monotonic counter, ledger counter, or authority counter. |
| `not_before` | Earliest valid admission time or counter. |
| `expiry` | Latest valid admission time or counter. Expired capsules MUST be rejected. |
| `time_or_counter_source` | Declares the validity source: wall-clock, monotonic counter, ledger counter, or epoch counter. |
| `critical_flags` | Flags that affect security, authority, execution, parsing, audit, or compatibility. Unknown critical flags MUST be rejected. |
| `non_critical_flags` | Flags that may be ignored only when policy explicitly allows ignoring them and signature covers them. |
| `extension_policy` | Defines whether extension blocks are forbidden, required, critical, or optional. |
| `extension_block_hash` | Hash of extension block content, or zero-equivalent if no extension exists and policy allows omission. |
| `issuer_id` | Identifier of the authorized issuer. The issuer MUST be allowed by policy. |
| `issuer_key_id` | Identifier of the issuer key used to sign or seal the capsule. |
| `signature_algorithm` | Signature or MAC algorithm identifier allowed by policy. Downgrade to weaker algorithms MUST be rejected. |
| `context_string_id` | Identifier for the context string used in signature/MAC binding. |
| `nonce` | Anti-reuse value bound into the capsule signature. |
| `capability_scope` | Structured or hashed scope defining the exact right granted. |
| `failure_policy` | Required behavior on validation or admission failure. |
| `signature` | Signature or MAC over the canonical capsule body and context. |

## 8. Capsule Classes

| Class | Use | Production admission |
|---|---|---|
| `production` | Authorizes protected runtime execution | Allowed only with full authority chain and audit commitment. |
| `non_production` | Used for tests, development, fixtures, and degraded evidence environments | MUST NOT authorize production operations. |
| `diagnostic` | Used for evidence gathering or failure explanation | MUST NOT authorize protected kernel execution. |
| `migration` | Used for policy, build, or epoch transition under strict policy | MUST be explicitly declared, scoped, and auditable. |
| `quarantine` | Used to retain suspicious or malformed capsule metadata | MUST NOT authorize execution. |

A non-production capsule MUST be distinguishable by class, flags, issuer policy, and signature context.

A production runtime MUST reject non-production, diagnostic, quarantine, or test-issued capsules for production operations.

## 9. Capsule Class Rules

1. A `production` capsule MUST bind policy, build, epoch, domain contract, hardware evidence, route, kernel, and audit commitment.
2. A `non_production` capsule MUST NOT authorize protected production kernels.
3. A `diagnostic` capsule MAY authorize evidence gathering but MUST NOT authorize protected kernel execution.
4. A `migration` capsule MUST only narrow, transition, or re-issue authority under explicit policy.
5. A capsule class MUST be covered by the signature or MAC.
6. A capsule class MUST NOT be inferred from flags alone.
7. Unknown capsule classes MUST be rejected in production.

## 10. Issuance Protocol

Capsule issuance MUST follow this sequence:

```text
1. collect active policy root
2. verify issuer authority under issuer policy
3. compute or verify build fingerprint
4. obtain platform-authorized hardware evidence
5. bind hardware evidence to live authority epoch
6. identify execution domain and subject component
7. narrow allowed route class and allowed kernel class
8. bind allowed instruction family and memory contract
9. bind domain contract hash
10. reserve or compute audit parent root and audit reservation
11. determine replay window and capsule uniqueness requirements
12. determine validity source and validity window
13. construct canonical capsule body
14. sign or seal canonical capsule body with authorized issuer
15. register capsule_id for replay prevention when required by policy
16. publish issuance evidence or ledger pre-commitment
```

Issuance MUST fail if any required evidence class is missing.

Issuance MUST NOT fabricate missing hardware evidence from local domain probes.

Issuance MUST NOT use diagnostic or test evidence to create a production capsule.

## 11. Validation Protocol

Runtime capsule validation MUST include:

```text
1. parse capsule using the canonical parser
2. reject malformed, ambiguous, non-canonical, duplicated, truncated, or overlong encodings
3. reject trailing garbage unless schema explicitly permits extension blocks
4. verify schema version
5. verify capsule class
6. verify critical flags
7. verify extension policy
8. verify issuer is authorized by active policy
9. verify issuer key is live, allowed, and not revoked
10. verify signature or MAC over canonical body and context
11. verify policy_hash matches active policy root
12. verify issuer_policy_hash matches active issuer policy
13. verify build_fingerprint matches active build
14. verify hardware_evidence_hash matches platform-authorized evidence
15. verify domain_contract_hash matches active domain contract
16. verify authority_epoch is live and not revoked
17. verify not_before <= current_time_or_counter <= expiry
18. verify replay_window_id and capsule_id are acceptable
19. verify capsule_id is not replayed when uniqueness is required
20. verify execution_domain matches the requested domain
21. verify subject_component matches the requesting component
22. verify allowed_route_class admits the requested route
23. verify allowed_kernel_class admits the requested kernel
24. verify allowed_instruction_family admits the requested execution family
25. verify allowed_memory_contract admits the requested memory behavior
26. verify audit_parent_root is available
27. verify audit_reservation_id is valid or can be reserved
28. verify pre_execution_witness_hash can be produced or matched
29. perform final epoch liveness check immediately before admission
30. commit or reserve the audit entry before protected execution
```

Validation success authorizes only the requested route and only under the validated context. It does not create general authority.

## 12. Runtime Admission State Machine

A production runtime MUST use the following admission state machine or one with equivalent semantics:

```text
UNPARSED
  -> PARSED_CANONICAL
  -> STRUCTURE_VALIDATED
  -> ISSUER_VALIDATED
  -> SIGNATURE_VALIDATED
  -> POLICY_BOUND
  -> BUILD_BOUND
  -> HARDWARE_EVIDENCE_BOUND
  -> DOMAIN_CONTRACT_BOUND
  -> EPOCH_LIVE
  -> REPLAY_CHECKED
  -> DOMAIN_MATCHED
  -> ROUTE_MATCHED
  -> KERNEL_MATCHED
  -> MEMORY_CONTRACT_MATCHED
  -> AUDIT_RESERVED
  -> FINAL_EPOCH_RECHECKED
  -> ADMITTED
  -> EXECUTED
  -> POST_EXECUTION_WITNESS_COMMITTED
  -> AUDIT_SEALED
```

Any failed transition MUST produce a deterministic failure code.

Protected execution before `AUDIT_RESERVED` is non-conforming.

Protected execution before `FINAL_EPOCH_RECHECKED` is non-conforming.

A capsule MUST NOT transition from any rejected, revoked, expired, consumed, quarantined, or retained state back to active.

## 13. Capsule Lifecycle

| State | Meaning | Allowed transition |
|---|---|---|
| `requested` | Runtime or platform requests authority | `requested -> issued` or `requested -> denied` |
| `issued` | Capsule has been sealed by an authorized issuer | `issued -> active`, `revoked`, or `expired` |
| `active` | Capsule is valid for admission under current context | `active -> consumed`, `revoked`, or `expired` |
| `admitted` | Runtime has validated and admitted the capsule | `admitted -> executed` or `admitted -> denied_before_execution` |
| `executed` | Authorized route has executed | `executed -> retained_for_audit` |
| `consumed` | Capsule has been used if one-time use is required | `consumed -> retained_for_audit` |
| `revoked` | Capsule is invalid due to epoch, policy, issuer, or evidence revocation | `revoked -> retained_for_audit` |
| `expired` | Capsule validity window ended | `expired -> retained_for_audit` |
| `quarantined` | Capsule is suspicious, malformed, or policy-incompatible | `quarantined -> retained_for_audit` or destruction per policy |
| `retained_for_audit` | Capsule metadata is retained for verification | terminal |

A runtime MUST NOT transition a capsule from `revoked`, `expired`, `consumed`, `quarantined`, or `retained_for_audit` back to `active`.

## 14. Replay Prevention

Replay prevention MUST use at least the following bindings:

1. `capsule_id` uniqueness under issuer or global namespace.
2. `authority_epoch` liveness and revocation state.
3. `policy_hash` and `build_fingerprint` context binding.
4. `hardware_evidence_hash` route binding.
5. `domain_contract_hash` domain behavior binding.
6. `not_before` and `expiry` validity window.
7. `replay_window_id` namespace binding.
8. `nonce` signature binding.
9. `audit_parent_root` or ledger parent binding.
10. `audit_reservation_id` where policy requires ordered or one-time execution.

A replayed capsule MUST produce:

```text
AUTHORITY_REPLAY_REJECTED
```

or an equivalent deterministic failure code explicitly mapped by policy.

Replay detection MUST occur before protected execution begins.

## 15. Invalidation Semantics

A capsule MUST be invalidated before protected execution when any of the following triggers occurs:

| Trigger | Required result |
|---|---|
| malformed encoding | reject |
| non-canonical encoding | reject |
| unknown production schema | reject |
| unknown critical flag | reject |
| policy hash mismatch | reject |
| issuer policy mismatch | reject |
| build fingerprint mismatch | reject |
| hardware evidence hash mismatch | reject |
| domain contract hash mismatch | reject |
| authority epoch mismatch | reject or revoke according to epoch policy |
| epoch revoked | reject |
| capsule expired | reject |
| not-before not reached | reject |
| issuer unknown | reject |
| issuer revoked | reject |
| issuer key revoked | reject |
| signature invalid | reject |
| signature algorithm not allowed | reject |
| downgrade attempt detected | reject |
| domain mismatch | reject |
| subject component mismatch | reject |
| route class mismatch | reject |
| kernel class mismatch | reject |
| instruction family mismatch | reject |
| memory contract mismatch | reject |
| audit commitment missing | reject for production |
| duplicate capsule id | reject |
| replay window violation | reject |
| test/debug issuer in production | reject |
| non-production capsule in production | reject |

Invalidation MUST occur before protected execution begins.

## 16. Failure Code Registry

Every false validation predicate MUST map to a deterministic failure code.

A conforming implementation MUST provide failure codes equivalent to the following:

| Failure | Required code |
|---|---|
| malformed capsule | `CAPSULE_PARSE_INVALID` |
| non-canonical encoding | `CAPSULE_NON_CANONICAL` |
| duplicate fields | `CAPSULE_DUPLICATE_FIELD_REJECTED` |
| trailing garbage | `CAPSULE_TRAILING_GARBAGE_REJECTED` |
| unsupported schema | `CAPSULE_SCHEMA_UNSUPPORTED` |
| unknown critical flag | `CAPSULE_CRITICAL_FLAG_UNKNOWN` |
| unknown capsule class | `CAPSULE_CLASS_UNKNOWN` |
| non-production capsule in production | `CAPSULE_CLASS_REJECTED_FOR_PRODUCTION` |
| issuer not allowed | `CAPSULE_ISSUER_REJECTED` |
| issuer revoked | `CAPSULE_ISSUER_REVOKED` |
| issuer key revoked | `CAPSULE_ISSUER_KEY_REVOKED` |
| debug issuer in production | `DEBUG_ISSUER_REJECTED` |
| test issuer in production | `TEST_ISSUER_REJECTED` |
| signature invalid | `CAPSULE_SIGNATURE_INVALID` |
| signature algorithm not allowed | `CAPSULE_SIGNATURE_ALGORITHM_REJECTED` |
| policy mismatch | `CAPSULE_POLICY_MISMATCH` |
| issuer policy mismatch | `CAPSULE_ISSUER_POLICY_MISMATCH` |
| build mismatch | `CAPSULE_BUILD_MISMATCH` |
| hardware evidence mismatch | `CAPSULE_HARDWARE_EVIDENCE_MISMATCH` |
| domain contract mismatch | `CAPSULE_DOMAIN_CONTRACT_MISMATCH` |
| epoch stale | `CAPSULE_EPOCH_STALE` |
| epoch revoked | `CAPSULE_EPOCH_REVOKED` |
| validity window not reached | `CAPSULE_NOT_YET_VALID` |
| expired capsule | `CAPSULE_EXPIRED` |
| replay detected | `AUTHORITY_REPLAY_REJECTED` |
| domain mismatch | `CAPSULE_DOMAIN_MISMATCH` |
| subject component mismatch | `CAPSULE_COMPONENT_MISMATCH` |
| route class mismatch | `CAPSULE_ROUTE_CLASS_REJECTED` |
| kernel class mismatch | `CAPSULE_KERNEL_CLASS_REJECTED` |
| instruction family mismatch | `CAPSULE_INSTRUCTION_FAMILY_REJECTED` |
| memory contract mismatch | `CAPSULE_MEMORY_CONTRACT_REJECTED` |
| audit commitment missing | `CAPSULE_AUDIT_COMMIT_MISSING` |
| audit reservation failure | `CAPSULE_AUDIT_RESERVATION_FAILED` |
| downgrade attempt | `CAPSULE_DOWNGRADE_REJECTED` |
| extension policy violation | `CAPSULE_EXTENSION_POLICY_REJECTED` |
| local authority inference attempted | `LOCAL_AUTHORITY_INFERENCE_REJECTED` |

Failure codes MAY be numerically encoded, but their semantics MUST be stable and auditable.

## 17. Cross-Domain Transfer Rules

Capsules are domain-scoped. A capsule issued for one execution domain MUST NOT be consumed by another domain.

Allowed examples:

```text
platform-issued AES capsule  -> AES domain bridge  -> AES route
platform-issued SIMD capsule -> SIMD domain bridge -> SIMD route
platform-issued FHE capsule  -> FHE domain bridge  -> FHE route
platform-issued PQC capsule  -> PQC domain bridge  -> PQC route
```

Forbidden examples:

```text
SIMD capsule -> AES execution
AES capsule  -> FHE execution
PQC capsule  -> SIMD route selection
FHE capsule  -> local authority minting
any domain capsule -> provider authority issuance
any capsule -> kernel self-selection by local probe
```

If cross-domain execution is required, a new capsule MUST be issued or a parent capability MUST explicitly authorize a narrowing transfer.

A transfer MUST only narrow authority. It MUST NOT preserve, widen, translate, or infer authority unless explicitly authorized by a parent capability and admitted by policy.

A narrowing transfer MUST bind:

```text
parent_capsule_id
new_capsule_id
transfer_reason
source_domain
target_domain
narrowed_scope
new_domain_contract_hash
transfer_audit_entry
issuer_authority
signature_or_mac
```

Transfer MUST be auditable and MUST NOT broaden authority.

## 18. Serialization Rules

Canonical serialization MUST be deterministic.

A conforming production serialization MUST satisfy:

1. Field order MUST be fixed by schema version.
2. Integer encoding MUST be fixed-width and endian-declared.
3. Hash fields MUST be fixed length.
4. Variable-length fields MUST include canonical length prefixes.
5. Duplicate fields MUST be rejected.
6. Missing required fields MUST be rejected.
7. Unknown critical fields MUST be rejected.
8. Unknown non-critical fields MAY be ignored only if policy allows the schema and signature covers them.
9. Parsers MUST reject trailing garbage unless extension blocks are explicitly permitted.
10. Textual encodings MUST NOT be used for production admission unless canonical byte normalization is specified.
11. Permissive parse recovery MUST NOT be used for production admission.
12. Ambiguous encodings MUST be rejected.
13. Non-minimal length encodings MUST be rejected unless schema explicitly permits them.
14. Extension blocks MUST be covered by signature or by a signed extension hash.

The canonical parser is part of the security boundary.

## 19. Signature and MAC Semantics

The signature or MAC MUST bind:

```text
context_string
capsule_version
capsule_class
canonical_capsule_body_without_signature
issuer_policy_hash
policy_hash
build_fingerprint
authority_epoch
hardware_evidence_hash
domain_contract_hash
audit_parent_root
audit_reservation_id
capability_scope
```

The context string MUST distinguish XPScerpto capsules from all other signed objects.

The signature algorithm MUST be allowed by the active policy root. Algorithm downgrade is forbidden.

Production capsules intended for external verification SHOULD use digital signatures.

MAC-based capsules MAY be used only when the verifier boundary, shared-key authority, and audit verification model are explicitly declared by policy.

A capsule signed with a deprecated, diagnostic, debug, test, or non-production algorithm MUST be rejected for production.

## 20. Issuer and Key Lifecycle

A production implementation MUST define an issuer and key lifecycle that includes:

1. issuer registration.
2. issuer authority scope.
3. issuer policy binding.
4. issuer key generation boundary.
5. issuer key storage boundary.
6. issuer key activation.
7. issuer key expiry.
8. issuer key rotation.
9. issuer key revocation.
10. emergency issuer disablement.
11. compromise response.
12. test issuer rejection in production.
13. debug issuer rejection in production.
14. audit record for issuer changes.

### 20.1 Issuer Rules

- A provider path MUST NOT issue production capsules directly.
- A domain executor MUST NOT hold a production issuer key.
- A test issuer MUST NOT be accepted in production.
- A debug issuer MUST NOT be accepted in production.
- An expired issuer key MUST be rejected.
- A revoked issuer key MUST be rejected.
- Issuer policy mismatch MUST reject the capsule.
- Issuer compromise MUST invalidate affected capsules according to policy.

## 21. Downgrade Prevention

The runtime MUST reject attempts to downgrade capsule authority, parsing, verification, or audit requirements.

| Downgrade attempt | Required response |
|---|---|
| production capsule parsed as non-production to avoid audit | reject |
| non-production capsule used in production | reject |
| new schema reinterpreted as old schema | reject unless compatibility is explicit in policy |
| stronger signature algorithm replaced with weaker one | reject |
| MAC substituted for signature where external verification is required | reject |
| kernel class broadened by older runtime | reject |
| route class widened by compatibility parser | reject |
| audit commitment omitted by legacy path | reject for production |
| epoch check bypassed by legacy path | reject |
| domain contract omitted by older capsule | reject for production |
| unknown critical flags ignored | reject |
| extension block stripped | reject if extension is critical or hash-bound |
| test adapter used as capsule validator | reject |

A downgrade rejection MUST produce:

```text
CAPSULE_DOWNGRADE_REJECTED
```

or a more specific deterministic code.

## 22. Prohibited Authority Patterns

The following patterns are forbidden:

1. Ambient authority from module import or link status.
2. Implicit execution rights from compile target flags.
3. Local authority fabrication by SIMD, AES, FHE, or PQC.
4. Accepting unsigned or unsealed production capsules.
5. Reusing capsules across domains without re-issuance.
6. Treating audit logs as authority.
7. Treating hardware evidence as authority without policy and epoch binding.
8. Allowing permissive parse recovery for malformed capsules.
9. Treating benchmark performance as execution authority.
10. Treating fixture success as production authorization.
11. Treating provider facts as authority without platform filtering.
12. Allowing old runtime compatibility paths to bypass capsule validation.
13. Allowing diagnostic paths to elevate production flags.
14. Treating route hints as capsule authority.
15. Treating a capsule as a general domain license.

## 23. Minimal Validation Predicate

A production capsule is valid only if:

```text
valid_capsule :=
    canonical_parse_ok
AND schema_allowed
AND capsule_class == production
AND critical_flags_allowed
AND extension_policy_satisfied
AND issuer_allowed
AND issuer_key_live
AND signature_or_mac_valid
AND signature_algorithm_allowed
AND policy_hash == active_policy_hash
AND issuer_policy_hash == active_issuer_policy_hash
AND build_fingerprint == active_build_fingerprint
AND hardware_evidence_hash == active_hardware_evidence_hash
AND domain_contract_hash == active_domain_contract_hash
AND authority_epoch == live_epoch
AND not_before <= now_or_counter <= expiry
AND replay_window_valid
AND capsule_id_not_replayed
AND execution_domain == requested_domain
AND subject_component == requested_component
AND allowed_route_class admits requested_route_class
AND allowed_kernel_class admits requested_kernel_class
AND allowed_instruction_family admits requested_instruction_family
AND allowed_memory_contract admits requested_memory_contract
AND audit_parent_root_available
AND audit_reservation_available
AND final_epoch_liveness_check_ok
```

Every false predicate MUST map to a deterministic failure code.

## 24. Domain-Specific Capsule Rules

### 24.1 SIMD

A SIMD capsule MUST bind:

```text
execution_domain = SIMD
allowed_route_class
allowed_kernel_class
allowed_instruction_family
hardware_evidence_hash
domain_contract_hash
allowed_memory_contract
```

SIMD MUST NOT use a capsule to authorize local CPUID probing, compiler macro authorization, `/proc/cpuinfo` inspection, `__builtin_cpu_supports`, or local ISA inference.

A SIMD capsule authorizes only the route admitted by hardware authority and platform policy.

### 24.2 AES

An AES capsule MUST bind the algorithm family, implementation class, memory contract, and key-handling domain.

AES MUST NOT use a capsule to authorize key leakage, local AES-NI probing, debug key paths, or audit exposure of key material.

### 24.3 FHE

An FHE capsule MUST distinguish production runtime authority from fixture, diagnostic, bootstrap proof, or test adapter authority.

FHE MUST NOT treat a fixture capsule as:

```text
FHE_BOOTSTRAP_COMPLETE
FHE_FULL_EVALMOD
FHE_FULL_COEFF_TO_SLOT
FHE_FULL_SLOT_TO_COEFF
FHE_PRODUCTION_READY
```

unless a separate production admission contract proves those claims under this specification.

An FHE capsule MUST bind domain contract, audit commitment, route class, and any production bootstrap route constraints required by policy.

### 24.4 PQC

A PQC capsule MUST bind algorithm suite, security level, key usage scope, and domain contract.

PQC MUST NOT accept debug keys, test keys, or unsupported algorithm modes in production capsules.

### 24.5 Platform-Controlled Domains

A platform-controlled capsule MUST NOT grant provider internals, raw ABI authority, or private runtime authority to downstream domains unless explicitly narrowed, sealed, and auditable.

## 25. Privacy and Secret-Safety Rules

Capsules MUST NOT contain:

1. plaintext secrets.
2. private keys.
3. raw user data.
4. AES key material.
5. FHE secret key material.
6. PQC private key material.
7. secret-dependent traces.
8. unredacted hardware identifiers unless policy classifies them as safe.
9. provider-private facts not approved for audit.
10. diagnostic-only memory dumps.

Hardware evidence hashes MUST NOT expose raw hardware identifiers unless policy classifies those identifiers as safe for audit.

Audit-bound capsule metadata MUST be sufficient for verification but MUST NOT become a secret-exfiltration channel.

## 26. Audit Binding

A production capsule MUST bind to audit evidence before protected execution.

At minimum, audit binding MUST include:

```text
audit_parent_root
audit_reservation_id
audit_commitment_policy
pre_execution_witness_hash
capsule_id
authority_epoch
policy_hash
build_fingerprint
execution_domain
allowed_route_class
allowed_kernel_class
result_code
```

A runtime MUST NOT execute a protected production route if it cannot reserve or bind audit evidence before execution.

Executor-local logs MUST NOT be treated as audit commitment.

A missing audit commitment MUST produce:

```text
CAPSULE_AUDIT_COMMIT_MISSING
```

or a more specific audit failure code.

## 27. Time and Counter Validity

A production policy MUST declare whether capsule validity is evaluated using:

```text
wall_clock_time
monotonic_counter
ledger_counter
authority_epoch_counter
hardware_attested_counter
```

A runtime MUST NOT mix validity sources unless policy explicitly permits the conversion.

If wall-clock time is used, policy MUST declare the trusted time boundary.

If a monotonic counter is used, rollback MUST be detected or treated as denial.

If validity cannot be evaluated, production admission MUST fail closed.

## 28. Extension Blocks

Extension blocks MAY be used only when policy allows them.

An extension block MUST declare whether it is:

```text
critical
non_critical
diagnostic_only
production_admissible
migration_only
```

Critical extension blocks MUST be understood and validated. Unknown critical extensions MUST be rejected.

Non-critical extensions MAY be ignored only if:

1. policy allows ignoring them.
2. the signature covers them or covers their hash.
3. ignoring them cannot broaden authority.
4. ignoring them cannot bypass audit.
5. ignoring them cannot weaken epoch, route, kernel, or domain validation.

Extension stripping MUST be rejected if the extension is critical or hash-bound.

## 29. Required Tests

A conforming implementation MUST test at least the following cases:

| Test | Required outcome |
|---|---|
| bit-flipped capsule body | signature validation fails |
| bit-flipped signature | signature validation fails |
| stale epoch | reject |
| revoked epoch | reject |
| wrong build fingerprint | reject |
| wrong policy hash | reject |
| wrong issuer policy hash | reject |
| wrong hardware evidence hash | reject |
| wrong domain contract hash | reject |
| wrong domain | reject |
| wrong subject component | reject |
| broader route request | reject |
| broader kernel request | reject |
| wrong instruction family | reject |
| wrong memory contract | reject |
| expired capsule | reject |
| not-before not reached | reject |
| non-production capsule in production API | reject |
| diagnostic capsule in production API | reject |
| duplicate capsule id | replay rejected |
| malformed serialization | reject |
| non-canonical serialization | reject |
| duplicate field | reject |
| trailing garbage | reject |
| unknown critical flag | reject |
| unknown critical extension | reject |
| stripped critical extension | reject |
| deprecated signature algorithm | reject |
| debug issuer in production | reject |
| test issuer in production | reject |
| missing audit commitment | production execution denied |
| audit reservation failure | production execution denied |
| final epoch revocation race | denied before execution |
| cross-domain capsule use | reject |
| downgrade parser attempt | reject |
| local authority inference attempt | reject |

## 30. Canonical Serialization Test Vectors

A conforming implementation SHOULD include canonical serialization test vectors.

At minimum, the test vector suite SHOULD include:

1. valid production capsule.
2. valid non-production capsule rejected by production API.
3. canonical byte order example.
4. signature preimage example.
5. expected capsule hash.
6. duplicate field rejection example.
7. trailing garbage rejection example.
8. unknown critical flag rejection example.
9. unknown non-critical flag accepted only under policy example.
10. endian mismatch rejection example.
11. extension block stripping rejection example.
12. malformed length prefix rejection example.
13. overlong variable-length field rejection example.
14. context-string mismatch rejection example.

A production implementation MUST NOT rely on parser behavior that is not covered by canonical serialization tests or equivalent formal parser validation.

## 31. Conformance Matrix

| ID | Requirement | Static check | Runtime check | Evidence | Required test |
|---|---|---|---|---|---|
| CAP-REQ-001 | Canonical serialization required | schema/parser review | reject non-canonical | `CAPSULE_NON_CANONICAL` | duplicate/trailing garbage |
| CAP-REQ-002 | Issuer must be authorized | issuer registry scan | reject unknown issuer | `CAPSULE_ISSUER_REJECTED` | unknown issuer |
| CAP-REQ-003 | Issuer key must be live | key registry scan | reject revoked key | `CAPSULE_ISSUER_KEY_REVOKED` | revoked key |
| CAP-REQ-004 | Signature binds context | signature preimage test | reject context mismatch | `CAPSULE_SIGNATURE_INVALID` | altered context |
| CAP-REQ-005 | Policy root must match | policy hash check | reject mismatch | `CAPSULE_POLICY_MISMATCH` | wrong policy |
| CAP-REQ-006 | Build must match | build fingerprint generation | reject mismatch | `CAPSULE_BUILD_MISMATCH` | wrong build |
| CAP-REQ-007 | Hardware evidence must match | evidence hash generation | reject mismatch | `CAPSULE_HARDWARE_EVIDENCE_MISMATCH` | wrong evidence |
| CAP-REQ-008 | Domain contract must match | contract hash check | reject mismatch | `CAPSULE_DOMAIN_CONTRACT_MISMATCH` | wrong contract |
| CAP-REQ-009 | Epoch must be live | epoch registry check | final liveness re-check | `CAPSULE_EPOCH_STALE` | revoked epoch |
| CAP-REQ-010 | Replay must be rejected | replay registry check | reject duplicate | `AUTHORITY_REPLAY_REJECTED` | duplicate id |
| CAP-REQ-011 | Domain scope enforced | domain binding scan | reject mismatch | `CAPSULE_DOMAIN_MISMATCH` | SIMD capsule used by AES |
| CAP-REQ-012 | Route class narrowed | route contract check | reject broader route | `CAPSULE_ROUTE_CLASS_REJECTED` | broader route |
| CAP-REQ-013 | Kernel class narrowed | kernel contract check | reject broader kernel | `CAPSULE_KERNEL_CLASS_REJECTED` | broader kernel |
| CAP-REQ-014 | Audit required before execution | audit API guard | deny if missing | `CAPSULE_AUDIT_COMMIT_MISSING` | no audit root |
| CAP-REQ-015 | Debug/test issuers rejected | issuer class scan | reject in production | `DEBUG_ISSUER_REJECTED` | debug issuer |
| CAP-REQ-016 | Downgrade rejected | compatibility scan | reject downgrade | `CAPSULE_DOWNGRADE_REJECTED` | old parser path |
| CAP-REQ-017 | Cross-domain transfer narrowed | transfer policy scan | reject broadening | `CAPSULE_DOMAIN_MISMATCH` | AES capsule used by FHE |
| CAP-REQ-018 | Secret-safe metadata | metadata scan | reject forbidden secret fields | `CAPSULE_SECRET_METADATA_REJECTED` | secret in capsule |

## 32. Production Admission Gate

A production capsule admission gate MUST report equivalent final flags:

```text
CAPSULE_SCHEMA_VALIDATED=YES
CAPSULE_CANONICAL_SERIALIZATION_ENFORCED=YES
CAPSULE_CLASS_PRODUCTION=YES
CAPSULE_ISSUER_AUTHORIZED=YES
CAPSULE_ISSUER_KEY_LIVE=YES
CAPSULE_SIGNATURE_VALID=YES
CAPSULE_POLICY_BOUND=YES
CAPSULE_BUILD_BOUND=YES
CAPSULE_HARDWARE_EVIDENCE_BOUND=YES
CAPSULE_DOMAIN_CONTRACT_BOUND=YES
CAPSULE_EPOCH_LIVE=YES
CAPSULE_REPLAY_REJECTED_IF_DUPLICATE=YES
CAPSULE_DOMAIN_MATCHED=YES
CAPSULE_ROUTE_CLASS_MATCHED=YES
CAPSULE_KERNEL_CLASS_MATCHED=YES
CAPSULE_MEMORY_CONTRACT_MATCHED=YES
CAPSULE_AUDIT_RESERVED_BEFORE_EXECUTION=YES
CAPSULE_FINAL_EPOCH_RECHECKED=YES
```

The following states MUST block production admission:

```text
CAPSULE_PARSE_INVALID=YES
CAPSULE_NON_CANONICAL_ACCEPTED=YES
CAPSULE_SIGNATURE_INVALID_ACCEPTED=YES
CAPSULE_POLICY_MISMATCH_ACCEPTED=YES
CAPSULE_BUILD_MISMATCH_ACCEPTED=YES
CAPSULE_EPOCH_STALE_ACCEPTED=YES
CAPSULE_REPLAY_ACCEPTED=YES
CAPSULE_DOMAIN_MISMATCH_ACCEPTED=YES
CAPSULE_ROUTE_BROADENED=YES
CAPSULE_KERNEL_BROADENED=YES
CAPSULE_AUDIT_COMMIT_MISSING=YES
CAPSULE_DEBUG_ISSUER_ACCEPTED_IN_PRODUCTION=YES
CAPSULE_TEST_ISSUER_ACCEPTED_IN_PRODUCTION=YES
CAPSULE_DIAGNOSTIC_CLASS_ACCEPTED_FOR_PRODUCTION=YES
CAPSULE_LOCAL_AUTHORITY_INFERENCE_ALLOWED=YES
```

## 33. Non-Claims

This specification does not claim that a capsule alone makes a system production-ready.

A capsule does not prove:

1. full platform production closure.
2. full SIMD performance dominance.
3. full AES side-channel safety.
4. full FHE bootstrap completion.
5. full PQC certification.
6. sanitizer cleanliness.
7. full regression materialization.
8. third-party certification.
9. hardware truth below the declared evidence boundary.
10. correctness of a domain kernel.

A capsule proves only that a specific execution right was sealed, scoped, validated, admitted, and audit-bound under the declared policy and authority model.

## 34. Final Conformance Statement

A component conforms to this specification only if all production execution paths requiring authority consume a validated sealed capability capsule, and no protected execution path can be reached by local inference, ambient authority, unsealed inputs, debug issuers, test fixtures, legacy adapters, or diagnostic-only routes.

A conforming implementation MUST prove:

```text
XPS_CAP_SCHEMA_ENFORCED=YES
XPS_CAP_CANONICAL_SERIALIZATION_ENFORCED=YES
XPS_CAP_SIGNATURE_CONTEXT_BOUND=YES
XPS_CAP_ISSUER_LIFECYCLE_ENFORCED=YES
XPS_CAP_POLICY_BOUND=YES
XPS_CAP_BUILD_BOUND=YES
XPS_CAP_HARDWARE_EVIDENCE_BOUND=YES
XPS_CAP_DOMAIN_CONTRACT_BOUND=YES
XPS_CAP_EPOCH_BOUND=YES
XPS_CAP_REPLAY_PREVENTED=YES
XPS_CAP_DOMAIN_SCOPED=YES
XPS_CAP_ROUTE_SCOPED=YES
XPS_CAP_KERNEL_SCOPED=YES
XPS_CAP_AUDIT_BOUND_BEFORE_EXECUTION=YES
XPS_CAP_DOWNGRADE_REJECTED=YES
XPS_CAP_TEST_DEBUG_PRODUCTION_REJECTED=YES
XPS_CAP_FAILURE_CODES_DETERMINISTIC=YES
```

Any protected production route that executes without these properties is non-conforming.

Any capsule used as a broad permission token is non-conforming.

Any diagnostic capsule that authorizes production execution is non-conforming.

Any legacy path that bypasses capsule validation is non-conforming.

Any authority inferred from local probing, module import, link status, compile flags, provider facts, or audit logs is non-conforming.

The sealed capability capsule is valid only as a narrow, cryptographically bound, policy-bound, build-bound, epoch-bound, domain-bound, route-bound, audit-bound execution authorization.
