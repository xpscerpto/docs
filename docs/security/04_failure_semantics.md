# XPScerpto Failure Semantics and Recovery Specification

**Document ID:** XPS-SOV-FAIL-004  
**Status:** Normative failure and recovery specification  
**Scope:** denial, revocation, quarantine, downgrade, safe halt, forensic escalation, audit failure, recovery authority, production admission blocking  
**Version:** 1.1  
**Document Type:** Normative Security and Runtime Failure Specification  
**Applies To:** platform, platform.api, hardware authority, SIMD, AES, FHE, PQC, provider path, sealed capability capsule admission, audit ledger, runtime execution gates

---

## 1. Purpose

This document defines deterministic failure behavior for XPScerpto. Failure is not an implementation detail. Failure is part of the sovereign contract.

If a module, runtime, provider, bridge, authority issuer, audit path, or execution kernel observes an invalid authority condition, it MUST respond according to this document.

A conforming implementation MUST NOT reinterpret failure as best-effort execution, warning-only behavior, silent fallback, opportunistic performance downgrade, local recovery, or diagnostic overclaim unless this document explicitly allows a non-production diagnostic outcome.

Production failure handling MUST be deterministic, auditable, authority-denying, and fail-closed.

## 2. Normative Language

The terms **MUST**, **MUST NOT**, **REQUIRED**, **SHALL**, **SHALL NOT**, **SHOULD**, **MAY**, and **OPTIONAL** are normative.

A component that violates a **MUST** or **MUST NOT** requirement is non-conforming even if functional tests pass.

A production implementation MUST treat this document as an execution boundary specification, not as advisory guidance.

## 3. Normative References

This specification is part of the XPScerpto sovereign specification layer.

| Document | Role |
|---|---|
| XPS-SOV-TM-001 — Sovereign Threat Model | Defines threat boundary, adversaries, protected assets, and non-claims. |
| XPS-SOV-AUTH-002 — Sovereign Authority Model | Defines authority lineage, epochs, revocation, admission, and safe halt behavior. |
| XPS-SOV-CAP-003 — Sealed Capability Capsule Specification | Defines capsule format, validation, replay prevention, lifecycle, and production admission. |
| XPS-SOV-FAIL-004 — Failure Semantics and Recovery Specification | Defines deterministic failure behavior and recovery authority. |
| XPS-SOV-AUD-005 — Audit Ledger Specification | Defines audit evidence, ledger integrity, external verification, and tamper response. |

If this document conflicts with another XPScerpto sovereign specification, the stricter authority-denying interpretation MUST apply.

If another specification permits execution after a failure and this document denies it, this document controls unless another stricter document denies earlier.

## 4. Core Failure Principle

Failure in XPScerpto MUST be explicit, deterministic, auditable, and authority-denying.

The following behavior is forbidden for production authority failures:

```text
log and continue
warn and continue
warn and fallback
best-effort execution
silent scalar fallback
silent diagnostic downgrade
executor-owned recovery
executor-only audit
local probing after missing hardware evidence
legacy path execution after capsule failure
production claim after incomplete evidence
```

A production route MUST NOT execute unless the relevant authority, capsule, epoch, policy, build, hardware evidence, route, kernel, and audit predicates are valid.

## 5. Failure Is Part of Sovereignty

XPScerpto treats failure as a security boundary.

A conforming implementation MUST prove not only that valid execution succeeds, but also that invalid execution fails in the correct way.

The following equation is normative:

```text
sovereign_failure :=
    invalid_condition_detected
AND failure_classified
AND deterministic_response_selected
AND protected_execution_blocked
AND evidence_recorded_when_available
AND recovery_restricted_to_authorized_actor
AND no_production_claim_emitted
```

If any predicate is false, failure handling is non-conforming.

## 6. Failure State Machine

A production failure affecting authority, execution, evidence, epoch, capsule validity, provider trust, or route authorization MUST follow the following state machine or one with equivalent semantics:

```text
OBSERVED_FAILURE
  -> CLASSIFIED_FAILURE
  -> AUTHORITY_IMPACT_ASSESSED
  -> RESPONSE_SELECTED
  -> EXECUTION_BLOCKED
  -> EVIDENCE_RECORDED
  -> RECOVERY_LOCKED_OR_ESCALATED
```

A failure affecting authority, audit, epoch, capsule validity, provider trust, or route authorization MUST NOT return directly to execution.

A runtime MUST NOT retry a failed authority path in a way that broadens authority, bypasses evidence, weakens validation, or changes the failure into best-effort execution.

## 7. Failure States

| State | Meaning | Production execution allowed |
|---|---|---|
| `DENY_EXECUTION` | Request is rejected before protected operation begins. | no |
| `REVOKE_CAPSULE` | Capsule is invalidated and cannot be reused. | no |
| `QUARANTINE_ROUTE` | Route, provider, capsule, evidence object, or ledger segment is isolated for investigation. | no for quarantined object |
| `DOWNGRADE_TO_NON_PRODUCTION` | Request may continue only in explicit non-production mode. | no production execution |
| `SAFE_HALT` | Runtime stops, blocks, or disables the affected operation in a controlled state. | no |
| `EXECUTION_NOT_SOVEREIGN` | Execution may have occurred or may be diagnostically represented without complete evidence; it cannot be claimed sovereign. | no sovereign claim |
| `TERMINATE_EXECUTION` | Runtime forcibly terminates the violating path. | no |
| `FORENSIC_ESCALATION` | Evidence is preserved and incident state is raised. | no by itself |
| `BLOCK_PRODUCTION_ADMISSION` | Future production admission is blocked for the affected authority class, route, ledger, provider, or domain. | no |
| `INVALIDATE_DEPENDENT_AUTHORITY` | Capabilities, capsules, or routes depending on invalid evidence are revoked or blocked. | no |
| `REQUIRE_REISSUANCE` | Existing authority cannot be reused; a new policy, epoch, capsule, or build admission is required. | no until reissued |
| `DIAGNOSTIC_ONLY_RECORD` | Failure may be recorded for diagnostics but cannot support production evidence. | no |
| `AUDIT_CHAIN_SUSPENDED` | A ledger segment or audit chain cannot be used for verification until repaired or rejected. | no for affected chain |
| `RECOVERY_REQUIRED` | Automatic continuation is forbidden until authorized recovery occurs. | no |

## 8. Severity Levels

Failures MUST be classified by severity when they affect authority, production admission, audit evidence, or recovery.

| Severity | Meaning | Typical response |
|---|---|---|
| `S0_DIAGNOSTIC` | Diagnostic-only condition with no production authority effect. | record diagnostic evidence |
| `S1_REQUEST_DENIAL` | Current request is invalid but wider system authority is not necessarily affected. | `DENY_EXECUTION` |
| `S2_ROUTE_QUARANTINE` | A capsule, route, provider fact, or evidence object is suspicious or invalid. | `QUARANTINE_ROUTE` |
| `S3_DOMAIN_LOCKDOWN` | A domain or bridge cannot safely continue until recovery. | domain halt or route quarantine |
| `S4_PRODUCTION_GATE_BLOCK` | Production admission depending on affected evidence, policy, ledger, or provider must stop. | `BLOCK_PRODUCTION_ADMISSION` |
| `S5_PLATFORM_SAFE_HALT` | Continuing may compromise platform-level sovereignty. | `SAFE_HALT` or process/system halt |

A conforming implementation MUST NOT treat `S4` or `S5` failures as simple request denials.

## 9. Deterministic Failure Rules

The following mappings are normative.

```text
IF authority_chain_invalid
    -> DENY_EXECUTION

IF policy_root_invalid
    -> SAFE_HALT

IF build_fingerprint_invalid
    -> SAFE_HALT

IF capsule_missing
    -> DENY_EXECUTION

IF capsule_signature_invalid
    -> DENY_EXECUTION + QUARANTINE_ROUTE

IF capsule_issuer_revoked
    -> DENY_EXECUTION + REVOKE_CAPSULE

IF capsule_epoch_mismatch
    -> REVOKE_CAPSULE

IF epoch_not_live
    -> DENY_EXECUTION

IF epoch_changes_between_validation_and_admission
    -> DENY_EXECUTION + FORENSIC_ESCALATION

IF hardware_evidence_missing AND request.production == true
    -> SAFE_HALT

IF hardware_evidence_missing AND request.production == false AND policy.allows_non_production_downgrade == true
    -> DOWNGRADE_TO_NON_PRODUCTION

IF hardware_evidence_invalid
    -> SAFE_HALT + INVALIDATE_DEPENDENT_AUTHORITY

IF audit_commit_missing AND request.production == true
    -> EXECUTION_NOT_SOVEREIGN + DENY_EXECUTION

IF audit_reservation_failed AND request.production == true
    -> DENY_EXECUTION

IF audit_ledger_tampered
    -> FORENSIC_ESCALATION + AUDIT_CHAIN_SUSPENDED + BLOCK_PRODUCTION_ADMISSION

IF runtime_attempts_local_probe
    -> TERMINATE_EXECUTION + FORENSIC_ESCALATION

IF route_not_authorized_by_capsule
    -> DENY_EXECUTION

IF provider_path_compromised
    -> QUARANTINE_ROUTE + INVALIDATE_DEPENDENT_AUTHORITY

IF provider_fact_rejected
    -> DENY_EXECUTION for dependent routes

IF diagnostic_path_attempts_production_elevation
    -> DENY_EXECUTION + FORENSIC_ESCALATION

IF legacy_path_bypasses_capsule_validation
    -> DENY_EXECUTION + BLOCK_PRODUCTION_ADMISSION for that path

IF production_failure_has_only_warning_behavior
    -> NON_CONFORMING_IMPLEMENTATION
```

## 10. Failure Code Registry

Every authority-affecting failure MUST map to a deterministic failure code.

A conforming implementation MUST provide codes equivalent to the following.

| Code | Trigger | Required response |
|---|---|---|
| `POLICY_ROOT_INVALID` | Active policy root missing, malformed, mismatched, or untrusted. | `SAFE_HALT` |
| `POLICY_ROOT_UNAVAILABLE` | Policy root cannot be accessed at admission. | `SAFE_HALT` |
| `BUILD_FINGERPRINT_INVALID` | Build fingerprint missing, mismatched, stale, or unverifiable. | `SAFE_HALT` |
| `BUILD_FINGERPRINT_MISMATCH` | Runtime build does not match capsule or policy binding. | `DENY_EXECUTION` |
| `HARDWARE_EVIDENCE_MISSING` | Required hardware evidence is unavailable. | production `SAFE_HALT`; non-production downgrade only if policy allows |
| `HARDWARE_EVIDENCE_INVALID` | Hardware evidence is malformed, stale, forged, or not platform-authorized. | `SAFE_HALT` + `INVALIDATE_DEPENDENT_AUTHORITY` |
| `NO_EVIDENCE` | Required evidence class is absent. | `DENY_EXECUTION` or `SAFE_HALT` according to evidence class |
| `NO_CAPSULE` | No valid sealed capsule exists. | `DENY_EXECUTION` |
| `CAPSULE_PARSE_INVALID` | Capsule cannot be parsed safely. | `DENY_EXECUTION` |
| `CAPSULE_NON_CANONICAL` | Capsule encoding is not canonical. | `DENY_EXECUTION` |
| `CAPSULE_SIGNATURE_INVALID` | Signature or MAC validation fails. | `DENY_EXECUTION` + optional quarantine |
| `CAPSULE_ISSUER_REJECTED` | Issuer is not authorized. | `DENY_EXECUTION` |
| `CAPSULE_ISSUER_REVOKED` | Issuer is revoked. | `DENY_EXECUTION` + `REVOKE_CAPSULE` |
| `CAPSULE_ISSUER_KEY_REVOKED` | Issuer key is revoked. | `DENY_EXECUTION` + `REVOKE_CAPSULE` |
| `CAPSULE_POLICY_MISMATCH` | Capsule policy hash does not match active policy. | `DENY_EXECUTION` |
| `CAPSULE_BUILD_MISMATCH` | Capsule build fingerprint does not match active build. | `DENY_EXECUTION` |
| `CAPSULE_HARDWARE_EVIDENCE_MISMATCH` | Capsule hardware evidence hash does not match platform-authorized evidence. | `DENY_EXECUTION` |
| `CAPSULE_DOMAIN_CONTRACT_MISMATCH` | Capsule domain contract does not match active domain contract. | `DENY_EXECUTION` |
| `CAPSULE_EXPIRED` | Capsule expiry has been reached. | `DENY_EXECUTION` |
| `CAPSULE_NOT_YET_VALID` | Capsule not-before condition is not satisfied. | `DENY_EXECUTION` |
| `AUTHORITY_REPLAY_REJECTED` | Capsule id, nonce, replay window, or ledger binding is reused illegally. | `DENY_EXECUTION` + possible `FORENSIC_ESCALATION` |
| `EPOCH_MISMATCH` | Capsule epoch differs from live epoch. | `REVOKE_CAPSULE` |
| `EPOCH_NOT_LIVE` | Epoch state is not live for admission. | `DENY_EXECUTION` |
| `EPOCH_REVOKED` | Epoch has been revoked. | `DENY_EXECUTION` + `REVOKE_CAPSULE` |
| `EPOCH_TOCTOU_REJECTED` | Epoch changed between validation and admission. | `DENY_EXECUTION` |
| `ROUTE_NOT_AUTHORIZED` | Requested route, kernel, domain, or execution class is not allowed. | `DENY_EXECUTION` |
| `CAPSULE_ROUTE_CLASS_REJECTED` | Requested route class is broader or different from capsule authority. | `DENY_EXECUTION` |
| `CAPSULE_KERNEL_CLASS_REJECTED` | Requested kernel class is not admitted by capsule. | `DENY_EXECUTION` |
| `CAPSULE_DOMAIN_MISMATCH` | Capsule is used by the wrong domain. | `DENY_EXECUTION` |
| `LOCAL_PROBE_ATTEMPT` | Domain attempts unauthorized local hardware or capability inference. | `TERMINATE_EXECUTION` + `FORENSIC_ESCALATION` |
| `AUDIT_COMMIT_MISSING` | Production evidence commitment unavailable. | `EXECUTION_NOT_SOVEREIGN` + `DENY_EXECUTION` |
| `AUDIT_RESERVATION_FAILED` | Audit reservation cannot be made before execution. | `DENY_EXECUTION` |
| `LEDGER_CHAIN_INVALID` | Audit ledger hash chain, Merkle chain, or checkpoint is invalid. | `FORENSIC_ESCALATION` + `AUDIT_CHAIN_SUSPENDED` |
| `LEDGER_PARENT_MISMATCH` | Audit parent root does not match expected root. | `FORENSIC_ESCALATION` + `BLOCK_PRODUCTION_ADMISSION` |
| `PROVIDER_FACT_REJECTED` | Provider fact fails platform filtering. | `DENY_EXECUTION` for dependent routes |
| `PROVIDER_PATH_COMPROMISED` | Provider path is compromised, stale, or untrusted. | `QUARANTINE_ROUTE` + `INVALIDATE_DEPENDENT_AUTHORITY` |
| `DOWNGRADE_FORBIDDEN` | Runtime attempts forbidden downgrade. | `DENY_EXECUTION` |
| `PRODUCTION_OVERCLAIM_REJECTED` | Diagnostic, fixture, or incomplete path attempts production claim. | `DENY_EXECUTION` + possible `FORENSIC_ESCALATION` |
| `SECRET_MATERIAL_EXPOSURE_RISK` | Failure path may expose key, plaintext, or secret-dependent trace. | `SAFE_HALT` |
| `RECOVERY_AUTHORITY_INVALID` | Unauthorized actor attempts recovery. | `DENY_EXECUTION` + `FORENSIC_ESCALATION` |

Failure codes MAY be numerically encoded, but the semantic mapping MUST remain stable and auditable.

## 11. Failure Evidence Schema

A conforming implementation MUST preserve structured failure evidence when the audit path or forensic path is available.

A failure evidence record MUST contain fields equivalent to:

```text
FailureEvidenceRecord {
    failure_id
    failure_code
    failure_state
    severity
    request_hash
    capsule_id_if_parseable
    issuer_id_if_parseable
    authority_epoch_snapshot
    policy_hash
    build_fingerprint
    hardware_evidence_hash_if_relevant
    domain_contract_hash_if_relevant
    requested_domain
    requested_route_class
    requested_kernel_class
    provider_id_if_relevant
    audit_parent_root_if_available
    ledger_segment_id_if_relevant
    monotonic_counter_or_timestamp
    recovery_required
    production_gate_blocked
    redaction_policy
    evidence_hash
}
```

Failure evidence MUST NOT expose secret material.

If evidence would reveal secrets, the implementation MUST redact it or commit to it cryptographically according to policy.

Executor-local logs alone MUST NOT be treated as sovereign forensic evidence.

## 12. Deny Behavior

`DENY_EXECUTION` MUST satisfy:

1. The protected operation MUST NOT begin.
2. No partial protected kernel execution may occur after denial.
3. The runtime MUST return a deterministic status or trigger a deterministic halt path.
4. The denial SHOULD be recorded in the audit ledger when the ledger is available.
5. The denial MUST NOT cause fallback to an ungoverned implementation.
6. The denial MUST NOT silently convert production execution into diagnostic execution.
7. The denial MUST NOT broaden authority.
8. The denial MUST NOT mint a replacement capsule.
9. The denial MUST NOT call local hardware or OS probes.
10. The denial MUST NOT emit production output.

For cryptographic operations, denial MUST NOT emit partially computed plaintext, ciphertext, key material, intermediate secrets, or route-specific side effects.

## 13. Quarantine Behavior

`QUARANTINE_ROUTE` isolates an untrusted object, route, provider, evidence object, or ledger segment.

| Object | Quarantine consequence |
|---|---|
| capsule | Capsule cannot be admitted and is retained only as evidence. |
| provider | Provider facts cannot be used for authority until cleared by platform authority. |
| kernel route | Route cannot be selected for production. |
| hardware evidence | Dependent capsules and routes become invalid. |
| ledger segment | Segment cannot be used for independent verification until repaired or rejected. |
| domain bridge | Bridge cannot admit production execution until recovery. |
| issuer key | Capsules signed by affected key are revoked or blocked according to policy. |

Quarantine MUST be sticky until a policy-authorized recovery or revocation action clears it.

Quarantine MUST NOT be cleared by the violating domain itself.

Quarantine clearance MUST be auditable.

## 14. Downgrade Rules

Downgrade is not recovery.

Downgrade is not fallback.

Downgrade is not production continuation.

Downgrade is permitted only for explicit non-production contexts.

A downgrade from production to non-production is forbidden unless all conditions below are true:

```text
request.production == false
AND policy.allows_non_production_downgrade == true
AND capsule.class != production
AND audit record marks result as non-production
AND no production secret or production operation is used
AND result cannot be counted as sovereign production evidence
```

The downgrade result MUST be visibly marked as non-production.

The downgrade result MUST NOT be counted as production evidence.

Forbidden downgrades include:

1. production capsule to non-production behavior to bypass audit;
2. production request to scalar fallback without authority;
3. missing hardware evidence to local probing;
4. missing audit ledger to executor-only log;
5. unsupported signature algorithm to weaker accepted algorithm;
6. production policy mismatch to best-effort old policy;
7. production ledger tamper to new local ledger;
8. route mismatch to nearest supported route;
9. kernel mismatch to slower ungoverned fallback;
10. FHE incomplete transform to full transform claim;
11. diagnostic capsule to production admission;
12. test fixture to production closure.

## 15. Safe Halt Semantics

`SAFE_HALT` means the runtime reaches a controlled non-execution state. It does not necessarily require process-wide abort.

The implementation MAY choose one of the following policy-declared halt modes:

| Halt mode | Meaning |
|---|---|
| request halt | Reject only the current request and return terminal status. |
| route halt | Disable the affected route until recovery. |
| domain halt | Disable the affected domain until policy recovery. |
| process halt | Terminate the process when continuing would be unsafe. |
| system halt | Stop broader service admission under platform control. |

The halt mode MUST be deterministic for the failure class and policy profile.

A component MUST NOT choose the least disruptive halt dynamically if policy requires stronger isolation.

A safe halt MUST NOT emit production output.

A safe halt MUST NOT leave a partially admitted route in an active state.

## 16. Forensic Escalation

`FORENSIC_ESCALATION` MUST preserve evidence sufficient to reconstruct:

1. request hash or request identifier;
2. capsule id if parseable;
3. issuer id if parseable;
4. authority epoch snapshot;
5. policy hash;
6. build fingerprint;
7. hardware evidence hash when relevant;
8. domain contract hash when relevant;
9. failure code;
10. requested domain;
11. requested route;
12. requested kernel class;
13. provider or evidence object involved;
14. ledger parent root if available;
15. timestamp, monotonic counter, or ledger counter;
16. recovery requirement;
17. production admission status;
18. redaction policy.

Forensic escalation MUST NOT expose secret material.

If evidence would reveal secrets, it MUST be redacted or cryptographically committed according to policy.

Forensic evidence MUST NOT be generated only by the violating executor when independent evidence is available.

## 17. Partial Execution Rules

A conforming implementation MUST distinguish failure timing.

| Timing | Required behavior |
|---|---|
| failure before protected kernel begins | no protected side effect may occur |
| failure after admission but before kernel start | deny, record admission failure, no protected execution |
| failure during protected execution | mark result non-sovereign unless policy-defined recovery proves otherwise |
| failure after partial cryptographic operation | no plaintext, key material, partial secret, or unsafe output may be emitted |
| failure after audit reservation but before execution | commit denial or cancellation witness |
| failure after execution but before audit seal | mark execution not sovereign unless recovery and witness seal are policy-authorized |

Domain-specific partial execution restrictions:

| Domain | Restriction |
|---|---|
| SIMD | No unauthorized route fallback after route failure. |
| AES | No key material, partial plaintext, or secret-dependent trace may be emitted. |
| FHE | No secret-key fallback, no fixture-as-production, no partial transform as full transform. |
| PQC | No private key material, debug key path, or unsupported security-level output may be emitted. |
| platform/provider | No provider fact may be elevated after rejection or quarantine. |

## 18. Audit Failure Semantics

Audit failure is an authority failure when production execution depends on audit evidence.

| Audit failure | Required response |
|---|---|
| audit commit missing | `EXECUTION_NOT_SOVEREIGN` + `DENY_EXECUTION` |
| audit reservation failed | `DENY_EXECUTION` |
| ledger parent mismatch | `FORENSIC_ESCALATION` + `BLOCK_PRODUCTION_ADMISSION` for affected chain |
| Merkle root mismatch | `FORENSIC_ESCALATION` + `AUDIT_CHAIN_SUSPENDED` |
| external checkpoint unavailable | policy decides deny or diagnostic-only; production claim MUST NOT proceed unless policy explicitly permits delayed checkpoint |
| executor-only log | not audit evidence |
| audit contains secret material | quarantine evidence + deny production claim |
| ledger chain invalid | suspend affected chain and block dependent production admission |
| audit witness missing after execution | `EXECUTION_NOT_SOVEREIGN` and forensic escalation |

A production runtime MUST NOT execute protected operations if the required audit commitment cannot be made before execution.

## 19. Domain-Specific Failure Semantics

### 19.1 SIMD

| Condition | Required response |
|---|---|
| `LOCAL_PROBE_ATTEMPT` | `TERMINATE_EXECUTION` + `FORENSIC_ESCALATION` |
| `ROUTE_NOT_AUTHORIZED` | `DENY_EXECUTION` |
| hardware evidence missing in production | `SAFE_HALT` |
| compiler macro used for authority | build rejection or runtime termination |
| route mismatch after capsule admission | `DENY_EXECUTION` |
| unauthorized ISA fallback | `DENY_EXECUTION` |
| local dispatch self-selection | `LOCAL_PROBE_ATTEMPT` |

SIMD MUST NOT recover from missing hardware evidence by probing CPU features locally.

### 19.2 AES

| Condition | Required response |
|---|---|
| unauthorized AES-NI probe | `TERMINATE_EXECUTION` + `FORENSIC_ESCALATION` |
| key material exposure risk | `SAFE_HALT` |
| audit key leakage risk | `DENY_EXECUTION` + `FORENSIC_ESCALATION` |
| unsupported AES route | `DENY_EXECUTION` |
| debug key path in production | `DENY_EXECUTION` |
| route downgrade to scalar without authority | `DOWNGRADE_FORBIDDEN` |

AES failure paths MUST NOT emit key material, plaintext, partial plaintext, or secret-dependent traces.

### 19.3 FHE

| Condition | Required response |
|---|---|
| fixture used as production proof | `DENY_EXECUTION` + `PRODUCTION_OVERCLAIM_REJECTED` |
| secret-key production path | `SAFE_HALT` + `FORENSIC_ESCALATION` |
| bootstrap incomplete but claimed complete | `DENY_EXECUTION` + `PRODUCTION_OVERCLAIM_REJECTED` |
| partial transform claimed as full | `DENY_EXECUTION` |
| missing rotation corpus for production route | `DENY_EXECUTION` |
| missing audit commitment | `EXECUTION_NOT_SOVEREIGN` + `DENY_EXECUTION` |
| test adapter in production path | build rejection or `DENY_EXECUTION` |
| diagnostic proof elevated to production | `PRODUCTION_OVERCLAIM_REJECTED` |

FHE MUST NOT self-recover by using fixtures, secret-key shortcuts, metadata-only corrections, or diagnostic-only ledgers.

### 19.4 PQC

| Condition | Required response |
|---|---|
| debug key in production | `DENY_EXECUTION` |
| test key in production | `DENY_EXECUTION` |
| unsupported security level | `DENY_EXECUTION` |
| private key audit leakage | `SAFE_HALT` |
| provider fact treated as authority | `DENY_EXECUTION` |
| downgraded algorithm suite | `DENY_EXECUTION` |

PQC failure paths MUST NOT emit private key material or unredacted secret-dependent traces.

### 19.5 Provider Path

| Condition | Required response |
|---|---|
| provider fact malformed | `PROVIDER_FACT_REJECTED` |
| provider fact stale | `PROVIDER_FACT_REJECTED` |
| provider path compromised | `QUARANTINE_ROUTE` + `INVALIDATE_DEPENDENT_AUTHORITY` |
| provider attempts authority issuance | `DENY_EXECUTION` + `FORENSIC_ESCALATION` |
| provider private import outside platform | build rejection |
| unfiltered provider fact reaches domain | `DENY_EXECUTION` |

Provider facts MUST NOT become authority without platform filtering, binding, and cross-checking.

## 20. Recovery Behavior

Recovery MUST be policy-authorized.

A component MAY recover only through one of the following:

1. new valid policy root;
2. new build fingerprint;
3. new hardware evidence from authorized path;
4. epoch transition and new capsule issuance;
5. quarantine clearance by platform authority;
6. ledger repair with external verification;
7. issuer key rotation or revocation update;
8. explicit non-production diagnostic profile;
9. platform-authorized route disablement;
10. full re-admission through the authority chain.

A domain MUST NOT self-recover by broadening its own authority.

A domain MUST NOT clear its own quarantine.

A violating executor MUST NOT be the sole recovery authority.

## 21. Recovery Authority Matrix

| Failure | Who may clear it |
|---|---|
| capsule revoked | capsule issuer or platform authority |
| capsule expired | new capsule issuance only |
| issuer revoked | platform authority or issuer governance authority |
| provider quarantined | platform authority only |
| route quarantined | platform authority plus domain authority; not executor alone |
| ledger invalid | audit authority plus external verification |
| build fingerprint invalid | rebuild plus platform admission |
| hardware evidence invalid | hardware authority path only |
| local probe attempt | platform security policy; not domain |
| epoch mismatch | epoch transition authority |
| domain contract mismatch | domain governance authority plus platform admission |
| production overclaim | platform authority plus evidence review |
| audit secret leakage | audit authority plus security review |
| test adapter leakage | build policy and production import scan closure |

The violating domain MUST NOT clear its own failure.

## 22. Required Decision Table

| Condition | Production response | Non-production response when policy allows |
|---|---|---|
| valid full authority chain | admit | admit as non-production if capsule class permits |
| no policy root | safe halt | safe halt |
| no build fingerprint | safe halt | safe halt or diagnostic-only |
| no hardware evidence | safe halt | downgrade to non-production only if policy allows |
| no capsule | deny | deny or diagnostic-only |
| invalid capsule signature | deny + quarantine | deny + quarantine |
| stale epoch | revoke capsule | revoke capsule |
| epoch TOCTOU | deny + forensic escalation | deny + forensic escalation |
| audit missing | execution not sovereign + deny | diagnostic-only with explicit non-sovereign marking |
| route mismatch | deny | deny |
| local probe attempt | terminate + forensic escalation | terminate + forensic escalation |
| ledger tamper | forensic escalation + block future production | forensic escalation |
| provider fact rejected | deny dependent route | diagnostic record |
| provider compromised | quarantine + invalidate dependent authority | quarantine |
| diagnostic path overclaim | deny + production overclaim rejected | deny claim |
| test adapter in production path | build reject or deny | diagnostic-only if isolated |
| secret material exposure risk | safe halt | safe halt or redacted forensic record |

## 23. Failure Conformance Matrix

| ID | Failure condition | Required state | Required code | Production gate | Evidence |
|---|---|---|---|---|---|
| FAIL-REQ-001 | missing capsule | `DENY_EXECUTION` | `NO_CAPSULE` | blocked | denial record |
| FAIL-REQ-002 | invalid capsule signature | `DENY_EXECUTION` + optional quarantine | `CAPSULE_SIGNATURE_INVALID` | blocked | capsule failure record |
| FAIL-REQ-003 | stale epoch | `REVOKE_CAPSULE` | `EPOCH_NOT_LIVE` | blocked | revocation record |
| FAIL-REQ-004 | epoch TOCTOU | `DENY_EXECUTION` | `EPOCH_TOCTOU_REJECTED` | blocked | epoch race record |
| FAIL-REQ-005 | local probe | `TERMINATE_EXECUTION` + `FORENSIC_ESCALATION` | `LOCAL_PROBE_ATTEMPT` | blocked | forensic record |
| FAIL-REQ-006 | audit missing | `EXECUTION_NOT_SOVEREIGN` + `DENY_EXECUTION` | `AUDIT_COMMIT_MISSING` | blocked | non-sovereign denial |
| FAIL-REQ-007 | ledger tamper | `FORENSIC_ESCALATION` + `BLOCK_PRODUCTION_ADMISSION` | `LEDGER_CHAIN_INVALID` | blocked until recovery | ledger incident |
| FAIL-REQ-008 | provider compromise | `QUARANTINE_ROUTE` + `INVALIDATE_DEPENDENT_AUTHORITY` | `PROVIDER_PATH_COMPROMISED` | dependent routes blocked | provider incident |
| FAIL-REQ-009 | hardware evidence missing in production | `SAFE_HALT` | `HARDWARE_EVIDENCE_MISSING` | blocked | evidence failure |
| FAIL-REQ-010 | policy root invalid | `SAFE_HALT` | `POLICY_ROOT_INVALID` | blocked | policy failure |
| FAIL-REQ-011 | build fingerprint invalid | `SAFE_HALT` | `BUILD_FINGERPRINT_INVALID` | blocked | build failure |
| FAIL-REQ-012 | diagnostic production overclaim | `DENY_EXECUTION` | `PRODUCTION_OVERCLAIM_REJECTED` | blocked | overclaim record |
| FAIL-REQ-013 | secret exposure risk | `SAFE_HALT` | `SECRET_MATERIAL_EXPOSURE_RISK` | blocked | redacted incident |
| FAIL-REQ-014 | unauthorized recovery attempt | `DENY_EXECUTION` + `FORENSIC_ESCALATION` | `RECOVERY_AUTHORITY_INVALID` | blocked | recovery incident |

## 24. Test Requirements

A conforming implementation MUST include tests for:

| Failure test | Expected assertion |
|---|---|
| invalid authority chain | `DENY_EXECUTION` |
| missing capsule | `NO_CAPSULE` and no execution |
| invalid capsule signature | `CAPSULE_SIGNATURE_INVALID` and production denial |
| epoch mismatch | `REVOKE_CAPSULE` |
| stale epoch | `EPOCH_NOT_LIVE` |
| epoch TOCTOU | `EPOCH_TOCTOU_REJECTED` |
| hardware evidence missing in production | `SAFE_HALT` |
| hardware evidence missing in non-production profile | `DOWNGRADE_TO_NON_PRODUCTION` only if policy allows |
| audit commitment missing | `EXECUTION_NOT_SOVEREIGN` and production denial |
| audit reservation failure | production denial |
| local probe attempt | `TERMINATE_EXECUTION` + `FORENSIC_ESCALATION` |
| tampered ledger | `LEDGER_CHAIN_INVALID` |
| ledger parent mismatch | `LEDGER_PARENT_MISMATCH` and production gate block |
| compromised provider fact | `PROVIDER_FACT_REJECTED` |
| provider path compromised | quarantine and dependent authority invalidation |
| route mismatch | `ROUTE_NOT_AUTHORIZED` |
| kernel class mismatch | `CAPSULE_KERNEL_CLASS_REJECTED` |
| forbidden downgrade | `DOWNGRADE_FORBIDDEN` |
| diagnostic path overclaim | `PRODUCTION_OVERCLAIM_REJECTED` |
| test adapter in production path | build rejection or denial |
| secret material exposure risk | safe halt and redacted evidence |
| unauthorized recovery attempt | `RECOVERY_AUTHORITY_INVALID` |

Tests MUST assert not only the error code but also that protected execution did not occur.

## 25. Production Gate Flags

A production failure gate MUST be able to report equivalent final flags:

```text
FAILURE_SEMANTICS_ENFORCED=YES
FAILURE_CODES_DETERMINISTIC=YES
WARNING_ONLY_PRODUCTION_FAILURE_REJECTED=YES
SILENT_FALLBACK_REJECTED=YES
LOCAL_PROBE_FAILURE_TERMINATES=YES
AUDIT_FAILURE_BLOCKS_PRODUCTION=YES
LEDGER_TAMPER_BLOCKS_PRODUCTION=YES
PROVIDER_COMPROMISE_INVALIDATES_DEPENDENT_AUTHORITY=YES
QUARANTINE_STICKY_UNTIL_AUTHORIZED_RECOVERY=YES
RECOVERY_AUTHORITY_ENFORCED=YES
DIAGNOSTIC_OVERCLAIM_REJECTED=YES
PARTIAL_EXECUTION_RULES_ENFORCED=YES
SECRET_SAFE_FORENSIC_EVIDENCE_ENFORCED=YES
```

The following states MUST block production claims:

```text
WARNING_ONLY_AUTHORITY_FAILURE=YES
SILENT_FALLBACK_AFTER_FAILURE=YES
LOCAL_PROBE_AFTER_HW_EVIDENCE_FAILURE=YES
EXECUTOR_ONLY_AUDIT_AFTER_PRODUCTION_FAILURE=YES
LEDGER_TAMPER_IGNORED=YES
PROVIDER_COMPROMISE_IGNORED=YES
ROUTE_QUARANTINE_CLEARED_BY_DOMAIN=YES
RECOVERY_BY_VIOLATING_EXECUTOR=YES
DIAGNOSTIC_RESULT_COUNTED_AS_PRODUCTION=YES
PARTIAL_CRYPTO_OUTPUT_AFTER_FAILURE=YES
SECRET_MATERIAL_IN_FAILURE_EVIDENCE=YES
```

## 26. Non-Claims

This specification does not claim that every failure makes the platform unsafe. It defines how failures must be classified, contained, recorded, and recovered.

This specification does not claim:

1. that all failures require process-wide abort;
2. that all forensic evidence is public;
3. that all non-production downgrades are forbidden;
4. that all diagnostic execution is forbidden;
5. that recovery is impossible;
6. that a warning can never be emitted;
7. that failure evidence may expose secrets;
8. that a local log is sufficient audit evidence;
9. that successful denial proves production readiness;
10. that a component is sovereign merely because it fails closed in one test.

It claims only that production authority failures MUST fail closed, deterministically, audibly, and without silent execution.

## 27. Final Conformance Statement

A module conforms to this document only if every failure state affecting authority, execution, audit, policy, build identity, hardware evidence, provider trust, capsule validity, epoch liveness, route authorization, or production claims is mapped to one deterministic response.

Warning-only behavior is non-conforming for production authority failures.

Silent fallback is non-conforming for production authority failures.

Executor-owned recovery is non-conforming for production authority failures.

Diagnostic overclaim is non-conforming.

A production execution path that continues after an authority failure without deterministic denial, safe halt, quarantine, revocation, or authorized recovery violates the XPScerpto sovereignty boundary.

A conforming implementation MUST prove:

```text
XPS_FAIL_FAILURE_STATES_DEFINED=YES
XPS_FAIL_STATE_MACHINE_ENFORCED=YES
XPS_FAIL_SEVERITY_CLASSIFIED=YES
XPS_FAIL_CODE_REGISTRY_STABLE=YES
XPS_FAIL_EVIDENCE_SCHEMA_ENFORCED=YES
XPS_FAIL_DENY_BLOCKS_EXECUTION=YES
XPS_FAIL_QUARANTINE_STICKY=YES
XPS_FAIL_DOWNGRADE_NON_PRODUCTION_ONLY=YES
XPS_FAIL_SAFE_HALT_DETERMINISTIC=YES
XPS_FAIL_FORENSIC_SECRET_SAFE=YES
XPS_FAIL_PARTIAL_EXECUTION_BLOCKED=YES
XPS_FAIL_AUDIT_FAILURE_BLOCKS_PRODUCTION=YES
XPS_FAIL_DOMAIN_FAILURES_ENFORCED=YES
XPS_FAIL_RECOVERY_AUTHORITY_ENFORCED=YES
XPS_FAIL_WARNING_ONLY_REJECTED=YES
XPS_FAIL_SILENT_FALLBACK_REJECTED=YES
```

Failure is sovereign only when it denies unsafe execution, preserves safe evidence, blocks false claims, and permits recovery only through declared authority.
