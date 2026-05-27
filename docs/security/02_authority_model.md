# XPScerpto Sovereign Authority Model

**Document ID:** XPS-SOV-AUTH-002  
**Status:** Normative authority specification  
**Scope:** authority lineage, issuer model, capsule admission, runtime authorization, epoch transition, revocation, safe halt, production claim control  
**Version:** 1.1  
**Document Class:** Formal Sovereign Specification Layer  
**Compliance Rule:** Any component that violates a MUST or MUST NOT requirement is non-conforming, even if functional tests pass.

---

## 1. Purpose

This document defines the formal authority model for XPScerpto.

It specifies how authority is created, narrowed, sealed, validated, revoked, and bound to runtime execution. The purpose of this model is to prevent authority from being inferred from module presence, build success, namespace visibility, local hardware probing, test fixtures, legacy adapters, or performance results.

Authority in XPScerpto is not a boolean attached to a module. It is not granted merely because code exists inside the repository. It is not inherited automatically through imports. It is not created by a successful unit test or benchmark.

Authority is a lineage of evidence.

A runtime decision is authorized only when every required element of that lineage is valid under AND-gated semantics. If any required predicate is missing, invalid, stale, unverifiable, contradictory, or revoked, the decision is not sovereign and MUST fail closed.

This document operationalizes the authority portion of the XPScerpto Sovereign Threat Model. It does not replace the threat model. Instead, it defines the concrete authority rules required for capsule admission, epoch liveness, revocation, route binding, safe halt, and production claim control.

## 2. Relationship to Other Specifications

XPScerpto security specifications are intended to form a connected normative set. This document is one part of that set.

| Document | Role |
|---|---|
| XPS-SOV-TM-001 — Sovereign Threat Model | Defines the sovereign threat model, protected assets, adversaries, boundaries, assumptions, and non-claims. |
| XPS-SOV-AUTH-002 — Sovereign Authority Model | Defines authority lineage, issuer rules, capsule admission, epoch transition, revocation, safe halt, and production claim control. |
| XPS-SOV-CAP-003 — Sealed Capability Capsule Specification | Defines capsule structure, canonical serialization, validation, lifecycle, replay prevention, downgrade prevention, and production/non-production separation. |
| XPS-SOV-FAIL-004 — Failure Semantics and Recovery Specification | Defines deterministic denial, quarantine, downgrade, safe halt, forensic escalation, and recovery authority. |
| XPS-SOV-AUD-005 — Sovereign Audit Ledger and Verification Specification | Defines audit ledger structure, tamper evidence, witness binding, external verification, retention, and production claim gating. |
| XPS-SOV-MAP-006 — Runtime Enforcement Mapping and Release Gate Specification | Maps normative specifications to compile contracts, runtime gates, tests, forensic evidence, conformance levels, and release decisions. |

Where documents overlap, the stricter authority-denying, execution-denying, or claim-denying interpretation MUST apply.

## 3. Normative Language

The terms MUST, MUST NOT, REQUIRED, SHALL, SHALL NOT, SHOULD, MAY, and OPTIONAL are normative.

| Term | Meaning |
|---|---|
| MUST / REQUIRED / SHALL | A mandatory requirement. Violation makes the component non-conforming. |
| MUST NOT / SHALL NOT | A mandatory prohibition. Violation makes the component non-conforming. |
| SHOULD | A strong recommendation. Deviation requires justification but is not automatically non-conforming unless tied to a MUST. |
| MAY / OPTIONAL | Allowed behavior, provided it does not violate any mandatory requirement. |

A component that violates a MUST or MUST NOT requirement is non-conforming even if:

- it compiles successfully;
- it passes functional tests;
- it improves performance;
- it satisfies a legacy test;
- it is hidden behind a diagnostic path;
- it is imported only indirectly;
- it is required for a benchmark;
- it is present only in a fallback implementation.

## 4. Core Authority Principle

The core XPScerpto authority rule is:

```text
No evidence -> no capsule
No capsule  -> no execution
No execution -> safe halt
```

This rule is not advisory. It is a production admission requirement.

A production execution decision MUST NOT occur unless the runtime can prove all of the following:

1. a valid platform policy root exists;
2. the build fingerprint is valid and policy-bound;
3. required hardware evidence is valid and policy-authorized;
4. the authority epoch is live;
5. the capability capsule is valid, narrow, sealed, and unexpired;
6. the requested domain and kernel route are allowed by the capsule;
7. the runtime route is bound to the decision;
8. an audit commitment is available before execution.

A missing predicate is not a warning. It is a denial condition.

## 5. Normative Authority Chain

The normative authority chain is:

```text
Platform Policy Root
    ↓
Build Fingerprint
    ↓
Hardware Evidence
    ↓
Authority Epoch
    ↓
Sealed Capability Capsule
    ↓
Runtime Execution Decision
    ↓
Audit Commitment
```

No layer in this chain is sufficient by itself.

A valid build fingerprint without policy is not authority. Hardware evidence without a policy root is not authority. A capsule without a live epoch is not authority. A runtime route without a capsule is not authority. A log without an audit commitment is not authority. A benchmark without admission evidence is not authority.

Authority exists only when the chain is complete, valid, narrow, and auditable.

## 6. Core Definitions

| Term | Definition |
|---|---|
| Platform Policy Root | The canonical policy root hash or sealed policy descriptor that defines allowed evidence, issuers, domains, kernel classes, routing behavior, failure semantics, and audit requirements. |
| Build Fingerprint | A stable digest over source/module identity, compile contracts, import-boundary policy, build options, target configuration, generated authority metadata, and policy guard configuration. |
| Hardware Evidence | A platform-authorized evidence object describing hardware facts that may be used for execution routing. It is not raw local probing by domains. |
| Authority Epoch | A monotonic liveness, freshness, and revocation domain for authority issuance and runtime admission. |
| Issuer | A policy-recognized authority actor that may issue, delegate, narrow, or bind authority only within its declared scope. |
| Sealed Capability Capsule | A signed, MAC-protected, or otherwise cryptographically sealed object granting a narrow execution right to a domain, route class, kernel class, policy hash, build fingerprint, hardware evidence hash, epoch, and expiry. |
| Runtime Execution Decision | The deterministic admission decision that either admits exactly one execution route or denies, halts, quarantines, or records diagnostic-only evidence. |
| Authority Lineage | The complete evidence chain required to reconstruct why a decision was admitted or denied. |
| Safe Halt | A protected failure behavior in which the requested operation is not performed, no fallback execution occurs, and no production claim is elevated. |
| Diagnostic Path | A non-production path used for testing, debugging, or evidence capture. It MUST NOT issue or elevate production authority. |

## 7. Authority Lineage

Each production decision MUST be reconstructable from an authority lineage equivalent to:

```text
AuthorityLineage {
    policy_hash,
    policy_version,
    build_fingerprint,
    build_mode,
    hardware_evidence_hash,
    hardware_evidence_class,
    authority_epoch,
    epoch_state,
    issuer_id,
    issuer_key_id,
    capsule_id,
    capsule_scope_hash,
    execution_domain,
    allowed_route_class,
    allowed_kernel_class,
    requested_route_class,
    requested_kernel_class,
    audit_parent_root,
    audit_commitment_root,
    decision_hash,
    denial_code_or_admission_code
}
```

An authority lineage is not valid merely because its fields are present. It is valid only when each field can be independently verified against the platform policy root, build fingerprint, hardware evidence, epoch state, capsule binding, and audit commitment.

The lineage MUST be narrow enough that an independent verifier can answer:

1. Which policy allowed or denied this decision?
2. Which build was admitted?
3. Which hardware evidence was used?
4. Which authority epoch was live?
5. Which issuer granted the right?
6. Which capsule granted the right?
7. Which domain was requested?
8. Which route and kernel class were allowed?
9. Which audit commitment recorded the result?
10. Which denial or admission code was produced?

If any answer cannot be reconstructed, the decision is not sovereign.

## 8. AND-Gated Authority Semantics

A production execution decision is authorized only if all predicates below are true:

```text
authorized :=
    policy_root_valid
AND build_fingerprint_valid
AND build_allowed_by_policy
AND hardware_evidence_valid
AND hardware_evidence_allowed_by_policy
AND authority_epoch_live
AND issuer_authorized_by_policy
AND capsule_signature_valid
AND capsule_policy_hash_matches
AND capsule_build_fingerprint_matches
AND capsule_hardware_evidence_hash_matches
AND capsule_epoch_matches_live_epoch
AND capsule_not_expired
AND capsule_not_revoked
AND capsule_domain_matches_request
AND capsule_kernel_class_allows_route
AND runtime_route_bound_to_decision
AND audit_commitment_available
AND no_diagnostic_elevation
```

An implementation MUST NOT reinterpret this as OR-gated fallback.

A missing predicate is equivalent to a false predicate. A contradictory predicate is equivalent to a false predicate. An unverifiable predicate is equivalent to a false predicate. A stale predicate is equivalent to a false predicate.

## 9. Negative Authority Rules

The following facts MUST NOT be treated as authority:

- successful compilation;
- successful linking;
- successful unit tests;
- benchmark superiority;
- module import visibility;
- namespace access;
- repository location;
- legacy compatibility;
- test fixture success;
- diagnostic ledger presence;
- provider statement without platform binding;
- hardware probe result outside the hardware authority path;
- environment variable override;
- compiler target macro;
- local CPU probing;
- fallback route availability;
- old phase result;
- adapter success;
- human approval without policy binding.

A component is non-conforming if it derives authority from module reachability, link visibility, namespace access, test harness availability, or diagnostic-only evidence.

## 10. Import Visibility Is Not Authority

Module import does not imply authority transfer.

A module may import a type, interface, or bridge surface without receiving authority to issue, broaden, validate, or execute a capability.

The following are forbidden:

1. deriving authority from an imported namespace;
2. deriving authority from link visibility;
3. deriving authority from a public type;
4. deriving authority from a test-only adapter;
5. deriving authority from a previously accepted fixture;
6. deriving authority from a private provider surface;
7. deriving authority from a local probe performed inside a domain.

Any implementation that treats import visibility as authority is non-conforming.

## 11. Trust Composition Model

XPScerpto composes trust by cross-checking independent evidence classes.

| Evidence Class | Trust Role | Rejected Use |
|---|---|---|
| Policy | Declares what is allowed | Cannot prove runtime fact by itself. |
| Build | Declares what was built | Cannot grant execution by itself. |
| Hardware Evidence | Declares route-relevant facts | Cannot bypass policy, epoch, or capsule binding. |
| Epoch | Declares liveness and revocation state | Cannot validate a forged capsule. |
| Issuer | Grants scoped authority under policy | Cannot override policy, build, epoch, or hardware mismatch. |
| Capsule | Grants a narrow execution right | Cannot broaden its own scope. |
| Ledger | Records and verifies decision evidence | Cannot retroactively authorize invalid execution. |

The trust model is compositional and fail-closed. Contradictory evidence MUST be treated as invalid authority, not as a warning.

## 12. Issuer Model

Authority issuance MUST be policy-bound. No component may issue production authority unless the platform policy root explicitly recognizes it as an issuer for the relevant authority class.

| Issuer Type | May Issue Production Capsules? | Notes |
|---|---|---|
| Platform Policy Issuer | Yes | Root-controlled authority issuance. |
| Platform API Delegated Issuer | Limited | May issue only when policy grants explicit delegation. |
| Hardware Authority Issuer | Limited | May bind hardware evidence; MUST NOT override policy or build. |
| Domain Bridge Issuer | No by default | May narrow an existing route only if explicitly delegated. |
| Test Issuer | No | Diagnostic only. MUST be rejected in production. |
| Debug Issuer | No | MUST be rejected in production. |
| Provider | No | Provides facts only. MUST NOT issue authority. |
| Domain Executor | No | Consumes authority. MUST NOT mint authority. |

### 12.1 Issuer Rules

1. An issuer MUST be recognized by the platform policy root.
2. An issuer MUST operate within a declared scope.
3. An issuer MUST NOT broaden its delegated authority.
4. An issuer MUST NOT issue capsules for domains outside its policy scope.
5. A test issuer MUST NOT be accepted in production.
6. A debug issuer MUST NOT be accepted in production.
7. A provider MUST NOT issue capability capsules.
8. A domain executor MUST NOT issue production authority to itself.

## 13. Sealed Capability Capsule

A sealed capability capsule is the minimum production object that grants a narrow execution right.

A production capsule MUST include, at minimum:

```text
SealedCapabilityCapsule {
    capsule_id,
    issuer_id,
    issuer_key_id,
    issuer_policy_hash,
    subject_domain,
    subject_component,
    allowed_route_class,
    allowed_kernel_class,
    allowed_instruction_family,
    allowed_memory_contract,
    policy_root_hash,
    build_fingerprint,
    hardware_evidence_hash,
    domain_contract_hash,
    authority_epoch,
    epoch_expiry,
    audit_parent_root,
    capability_scope_hash,
    nonce,
    issued_at,
    expires_at,
    replay_window_id,
    failure_policy,
    signature_or_mac
}
```

### 13.1 Capsule Rules

1. A capsule missing a required field MUST be rejected.
2. A capsule with an invalid signature or MAC MUST be rejected.
3. A capsule with an unrecognized issuer MUST be rejected.
4. A capsule issued by a revoked issuer MUST be rejected.
5. A capsule bound to a different policy root MUST be rejected.
6. A capsule bound to a different build fingerprint MUST be rejected.
7. A capsule bound to incompatible hardware evidence MUST be rejected.
8. A capsule bound to an inactive epoch MUST be rejected.
9. A capsule outside its expiry window MUST be rejected.
10. A capsule outside its replay window MUST be rejected.
11. A capsule with a missing audit parent root MUST be rejected in production.
12. A capsule that allows a broader route than policy permits MUST be rejected.
13. A capsule signed by a test or debug issuer MUST be rejected in production.

## 14. Authority Inheritance

Authority inheritance is permitted only in the narrowing direction.

```text
parent authority scope >= child authority scope
```

A child authority may:

- reduce domains;
- reduce route classes;
- reduce kernel classes;
- reduce instruction families;
- reduce memory contracts;
- shorten expiry;
- narrow policy subset;
- narrow evidence subset;
- restrict audit behavior.

A child authority MUST NOT:

- add domains;
- add route classes;
- add kernel classes;
- extend expiry;
- change issuer trust;
- replace evidence bindings;
- override policy;
- bypass epoch state;
- convert diagnostic authority into production authority.

### 14.1 Parent/Child Scope Proof

Every inherited authority object MUST produce:

```text
parent_scope_hash
child_scope_hash
subset_proof
```

The child scope MUST be proven to be a subset of the parent scope. If the subset relation cannot be proven, the inherited authority MUST be rejected.

### 14.2 Domain Inheritance Rules

1. Platform may issue authority to platform.api only through declared policy surfaces.
2. platform.api may expose filtered authority to hardware authority only through policy-bound interfaces.
3. hardware authority may produce routing facts and sealed decision material for domains.
4. SIMD, AES, FHE, and PQC may consume authority but MUST NOT mint authority for themselves.
5. Domain bridge code MAY narrow a route but MUST NOT broaden a capsule.
6. Test harnesses MUST NOT transfer authority to production paths.

## 15. Authority Invalidation

Authority MUST be invalidated when any condition below occurs.

| Condition | Required Result |
|---|---|
| Policy root changes | Capsules tied to the old policy are invalid unless policy declares explicit migration compatibility. |
| Build fingerprint changes | Capsules tied to the old build are invalid for the new build. |
| Hardware evidence changes materially | Capsules bound to old evidence are invalid for affected routes. |
| Authority epoch advances with revocation | Capsules in the revoked epoch are invalid. |
| Capsule expiry is reached | Capsule is invalid. |
| Issuer is revoked | Capsules issued by that issuer are invalid unless policy defines a narrow survival window. |
| Audit commitment is unavailable | Production authority is incomplete; execution denied. |
| Domain mismatch occurs | Capsule cannot be used for the request. |
| Kernel class mismatch occurs | Route denied. |
| Diagnostic path attempts production elevation | Claim denied; evidence marked diagnostic-only. |

## 16. Revocation Priority Rules

Revocation dominates expiry.

Revocation dominates prior admission unless a controlled completion rule explicitly applies.

Policy root invalidation dominates issuer validity.

Epoch revocation dominates capsule validity.

Build mismatch dominates route availability.

Capsule invalidity dominates audit availability.

| Conflict | Required Result |
|---|---|
| Capsule not expired, but issuer revoked | Deny. |
| Epoch live, but policy root changed | Deny unless migration compatibility is declared. |
| Hardware evidence valid, but build fingerprint mismatch | Deny. |
| Audit available, but capsule invalid | Deny. |
| Draining epoch, but policy disallows completion | Deny. |
| Capsule valid, but route mutated after validation | Deny. |
| Test issuer valid in diagnostic mode, but used in production | Deny. |
| Provider fact present, but not platform-bound | Deny authority elevation. |

## 17. Epoch Transition Semantics

Authority epochs are monotonic liveness domains. An epoch transition MUST be explicit and auditable.

### 17.1 Epoch States

| State | Meaning |
|---|---|
| pending | A new epoch is prepared but not yet issuing production authority. |
| live | The epoch may issue and admit capsules. |
| draining | Existing admitted decisions may complete only if policy explicitly allows. New production admission is disabled. |
| revoked | No production execution may be admitted. |
| sealed | Epoch evidence is closed for audit retention. |

### 17.2 Valid Epoch Transitions

```text
pending  -> live       allowed by policy root
live     -> draining   allowed for controlled shutdown
live     -> revoked    allowed for emergency revocation
draining -> revoked    allowed
draining -> sealed     allowed only after all admitted executions are closed
revoked  -> sealed     allowed after revocation evidence is committed
sealed   -> live       forbidden
revoked  -> live       forbidden
```

### 17.3 Draining Restrictions

A draining epoch MUST NOT admit new production decisions.

It may only complete decisions that were already admitted under the same:

```text
epoch_snapshot
capsule_id
audit_parent_root
runtime_decision_hash
requested_domain
requested_route_class
selected_kernel_class
```

A draining completion path MUST NOT change:

- route class;
- kernel class;
- domain;
- capsule scope;
- issuer;
- audit parent root;
- decision hash;
- hardware evidence binding.

A draining state MUST NOT be used as fallback admission.

## 18. Runtime Route Binding

A runtime route is bound to a decision only when the following values are identical between validation and execution:

```text
requested_domain
requested_route_class
selected_kernel_class
capsule_scope_hash
epoch_snapshot
audit_parent_root
decision_hash
hardware_evidence_hash
build_fingerprint
policy_root_hash
```

Any mutation after validation MUST invalidate the decision.

The kernel MUST NOT self-select a route from local probes. The domain MUST NOT upgrade or downgrade route class after capsule validation. The runtime MUST NOT substitute a diagnostic adapter for a production route. The executor MUST NOT modify the capsule, route class, kernel class, or audit binding after admission.

## 19. Admission Result Types

A runtime admission decision MUST produce one of the following result types.

| Result | Meaning | Production Execution Allowed? |
|---|---|---|
| ADMIT | Fully authorized execution route. | Yes |
| DENY | Request rejected because an authority predicate failed. | No |
| SAFE_HALT | Protected operation blocked due to missing or invalid evidence. | No |
| QUARANTINE | Route, component, or issuer isolated after violation. | No |
| DIAGNOSTIC_ONLY | Evidence may be recorded, but it cannot authorize production execution. | No |

A result type MUST be included in audit evidence when audit is available.

## 20. Safe Halt Contract

The following chain is normative:

```text
NO_EVIDENCE  -> NO_CAPSULE
NO_CAPSULE   -> NO_EXECUTION
NO_EXECUTION -> SAFE_HALT
```

### 20.1 Formal Meaning

| State | Meaning |
|---|---|
| NO_EVIDENCE | Required policy, build, hardware, epoch, issuer, capsule, or audit evidence is missing or unverifiable. |
| NO_CAPSULE | No sealed capsule can be validly issued or validated from the evidence set. |
| NO_EXECUTION | Runtime admission cannot produce an authorized route. |
| SAFE_HALT | Runtime blocks the protected operation without performing it. |

### 20.2 Operational Safe Halt

SAFE_HALT means:

1. the protected operation is not performed;
2. no partial domain execution is allowed;
3. no fallback kernel is selected;
4. no diagnostic adapter is substituted;
5. no production flag is elevated;
6. no route mutation is accepted;
7. no stale capsule is reused;
8. no provider fact is elevated into authority;
9. diagnostic evidence may be emitted only as diagnostic evidence.

Diagnostic evidence produced during SAFE_HALT MUST NOT become production admission evidence.

## 21. Runtime Admission Algorithm

A conforming runtime admission function MUST be equivalent to:

```text
function admit(
    request,
    capsule,
    epoch_snapshot,
    policy_root,
    build_fingerprint,
    hardware_evidence,
    audit_root
):
    if not verify_policy_root(policy_root):
        return SAFE_HALT(POLICY_ROOT_INVALID)

    if not verify_build_fingerprint(build_fingerprint, policy_root):
        return SAFE_HALT(BUILD_FINGERPRINT_INVALID)

    if not verify_hardware_evidence(hardware_evidence, policy_root):
        return SAFE_HALT(HARDWARE_EVIDENCE_INVALID)

    if not epoch_live(epoch_snapshot):
        return DENY(EPOCH_NOT_LIVE)

    if not verify_capsule(capsule):
        return DENY(CAPSULE_AUTH_INVALID)

    if not verify_issuer(capsule.issuer_id, policy_root):
        return DENY(ISSUER_NOT_AUTHORIZED)

    if issuer_revoked(capsule.issuer_id, epoch_snapshot):
        return DENY(ISSUER_REVOKED)

    if not capsule_binds(policy_root, build_fingerprint, hardware_evidence, epoch_snapshot):
        return DENY(CAPSULE_BINDING_MISMATCH)

    if capsule_expired(capsule):
        return DENY(CAPSULE_EXPIRED)

    if capsule_replayed(capsule):
        return DENY(AUTHORITY_REPLAY_REJECTED)

    if not capsule_allows(request.domain, request.route_class, request.kernel_class):
        return DENY(ROUTE_NOT_AUTHORIZED)

    if not audit_root_available(audit_root):
        return DENY(AUDIT_COMMIT_MISSING)

    decision = bind_decision(request, capsule, epoch_snapshot, audit_root)

    if not epoch_still_live(epoch_snapshot):
        return DENY(EPOCH_TOCTOU_REJECTED)

    if route_mutated_after_validation(request, decision):
        return DENY(ROUTE_MUTATION_REJECTED)

    return ADMIT(decision_hash(decision))
```

Implementations may optimize this flow but MUST preserve the same denial points and evidence semantics.

## 22. Revocation Behavior

Revocation MUST be deterministic.

1. A revoked issuer invalidates all unexpired capsules issued by that issuer unless policy explicitly narrows an allowed survival window.
2. A revoked epoch invalidates all capsules bound to that epoch for new production admission.
3. Revoked hardware evidence invalidates route classes depending on that evidence.
4. A revoked policy root invalidates all capsules bound to that policy root.
5. A revoked build fingerprint invalidates all capsules bound to that build.
6. A revoked domain contract invalidates capsules bound to that domain contract.
7. A revoked audit root prevents production admission if no valid replacement is policy-declared.

Revocation MUST generate a ledger event or revocation evidence artifact unless the system is in a state where audit is unavailable. If audit is unavailable, production admission MUST fail.

## 23. Authority Failure Codes and Required Evidence

| Failure | Required Code | Minimum Evidence |
|---|---|---|
| Missing evidence | `NO_EVIDENCE` | missing evidence class, request id, domain |
| Missing capsule | `NO_CAPSULE` | reason issuance or validation is impossible |
| Forged capsule | `CAPSULE_AUTH_INVALID` | capsule id if parseable, issuer id if parseable, validation point |
| Capsule replay | `AUTHORITY_REPLAY_REJECTED` | capsule id, epoch, parent ledger root |
| Epoch mismatch | `EPOCH_MISMATCH` | capsule epoch, live epoch snapshot |
| Epoch not live | `EPOCH_NOT_LIVE` | epoch id, epoch state |
| Epoch TOCTOU | `EPOCH_TOCTOU_REJECTED` | validation epoch, admission epoch |
| Issuer revoked | `ISSUER_REVOKED` | issuer id, revocation evidence id |
| Route mismatch | `ROUTE_NOT_AUTHORIZED` | requested route, capsule route class |
| Route mutation | `ROUTE_MUTATION_REJECTED` | validation route, execution route |
| Audit missing | `AUDIT_COMMIT_MISSING` | domain, request hash, attempted route |
| Build mismatch | `BUILD_FINGERPRINT_INVALID` | expected fingerprint, actual fingerprint |
| Hardware evidence invalid | `HARDWARE_EVIDENCE_INVALID` | evidence hash, evidence class |
| Diagnostic elevation | `DIAGNOSTIC_ELEVATION_REJECTED` | diagnostic path id, attempted claim |
| Test issuer in production | `TEST_ISSUER_REJECTED` | issuer id, requested domain |
| Provider fact unbound | `PROVIDER_FACT_UNBOUND` | provider id, fact class |

## 24. Production Claim Guard

A component MUST NOT claim production authority unless it proves:

```text
POLICY_BOUND=YES
BUILD_BOUND=YES
HARDWARE_EVIDENCE_BOUND=YES
EPOCH_LIVE=YES
ISSUER_AUTHORIZED=YES
CAPSULE_VALID=YES
ROUTE_BOUND=YES
AUDIT_COMMIT_AVAILABLE=YES
NO_DIAGNOSTIC_ELEVATION=YES
NO_TEST_ISSUER=YES
NO_LOCAL_AUTHORITY=YES
NO_IMPORT_AUTHORITY_INFERENCE=YES
```

If any of these are false, missing, unverifiable, or diagnostic-only, the production claim MUST be rejected.

### 24.1 Forbidden Production Elevation

The following MUST NOT elevate a production flag:

- test fixture success;
- diagnostic report success;
- adapter execution;
- benchmark result;
- partial materialization;
- target-only build;
- stale phase result;
- local hardware probe;
- provider statement without platform binding;
- audit log without external verification;
- route execution without capsule binding.

## 25. Domain Authority Rules

### 25.1 SIMD

SIMD MUST NOT:

- probe CPUID locally;
- use compiler macros for authority;
- read `/proc/cpuinfo`;
- use `__builtin_cpu_supports`;
- use `getauxval` for self-authorization;
- select kernels from local hardware inference;
- broaden routes after capsule admission.

SIMD MUST:

- consume route facts from hardware authority;
- validate route class against the capsule;
- bind kernel selection to the runtime decision;
- emit denial evidence for route mismatch.

### 25.2 AES

AES MUST NOT:

- select AES-NI or fallback paths through local probing;
- rely on environment variables for authority;
- expose key material through audit evidence;
- claim side-channel completeness without a domain contract.

AES MUST:

- use authority-routed capabilities;
- reject route mismatch;
- keep key material outside audit records.

### 25.3 FHE

FHE MUST NOT:

- use test bootstrap adapters in production;
- elevate fixture success to production closure;
- claim full bootstrap from diagnostic paths;
- use secret-key material in production paths unless explicitly authorized by a production contract;
- treat partial materialization as full runtime closure;
- treat legacy phase success as current authority.

FHE MUST:

- bind bootstrap admission to policy, capsule, epoch, and audit;
- reject missing production route evidence;
- distinguish fixture proof from production proof;
- reject diagnostic overclaim.

### 25.4 PQC

PQC MUST NOT:

- use debug or test private key material in production;
- treat provider facts as direct authority;
- claim broad quantum resistance without primitive-level declaration;
- bypass issuer and capsule validation.

PQC MUST:

- protect private key material from audit leakage;
- bind algorithm suites to policy;
- reject unsupported or security-none targets in production.

### 25.5 Provider Path

Provider code MUST NOT:

- issue authority directly;
- bypass platform filtering;
- export private facts to domains;
- own domain kernel routes;
- decide production admission.

Provider code MUST:

- provide facts only;
- be filtered by platform policy;
- produce evidence source class metadata;
- fail closed when facts are stale, malformed, or unverifiable.

## 26. Conformance Test Matrix

A conforming implementation MUST provide tests, scans, or formal checks covering at least the following.

| Requirement | Test | Expected Result |
|---|---|---|
| Missing policy root | admission test | SAFE_HALT |
| Invalid build fingerprint | admission test | SAFE_HALT |
| Missing hardware evidence | route test | SAFE_HALT or DENY |
| Forged capsule | capsule test | DENY |
| Expired capsule | capsule test | DENY |
| Revoked issuer | revocation test | DENY |
| Revoked epoch | epoch test | DENY |
| Epoch TOCTOU | race test | DENY |
| Route mismatch | route test | DENY |
| Route mutation after validation | mutation test | DENY |
| Audit missing | audit test | DENY |
| Draining new admission | epoch test | DENY |
| Test issuer in production | issuer test | DENY |
| Debug issuer in production | issuer test | DENY |
| Diagnostic route elevation | overclaim test | DENY |
| Import visibility authority inference | import-boundary scan | build rejection |
| Local hardware probing by domain | static/runtime test | build rejection or runtime halt |
| Provider fact without binding | provider test | no authority elevation |
| Capsule replay | replay test | DENY |
| Parent/child broadening | inheritance test | DENY |
| Production claim from fixture | claim guard test | DENY |

## 27. Conformance Statement

A component conforms to this document only if it can show:

1. compile-time contracts;
2. import-boundary enforcement;
3. runtime admission enforcement;
4. issuer validation;
5. capsule validation;
6. epoch liveness checks;
7. revocation behavior;
8. route binding;
9. audit commitment;
10. negative tests;
11. forensic evidence;
12. diagnostic/production separation.

A component that only documents intent without enforcing admission is non-conforming.

A component that passes functional tests but bypasses authority admission is non-conforming.

A component that accepts diagnostic authority as production authority is non-conforming.

A component that derives authority from imports, namespaces, local probes, or test adapters is non-conforming.

## 28. Non-Claims

This authority model does not claim that XPScerpto is unbreakable, formally verified, side-channel complete, production-certified, or immune to malicious hardware, malicious compilers, or physical attacks.

It claims only the following when implemented and tested:

1. XPScerpto defines authority as evidence lineage.
2. Authority is AND-gated.
3. Authority is policy-bound.
4. Authority is build-bound.
5. Authority is epoch-bound.
6. Authority is capsule-bound.
7. Authority is route-bound.
8. Authority is audit-bound.
9. Authority is revocable.
10. Missing evidence fails closed.
11. Diagnostic paths cannot elevate production authority.
12. Imports do not transfer authority.

## 29. Final Normative Rule

The final authority rule is:

```text
Authority must be declared,
sealed,
scoped,
narrowed,
policy-bound,
build-bound,
hardware-bound when required,
epoch-bound,
issuer-bound,
capsule-bound,
route-bound,
audit-bound,
and independently verifiable.
```

Any execution outside this chain is non-sovereign.

Any production claim without this chain is rejected.

Any fallback that bypasses this chain is non-conforming.

Any diagnostic path that elevates production authority is a violation.

Any authority that cannot be reconstructed is not authority.
