# XPScerpto Sovereign Audit Ledger and Verification Specification

**Document ID:** XPS-SOV-AUD-005  
**Former ID:** XPS-SOV-LEDGER-005  
**Status:** Normative evidence and verification specification  
**Scope:** sovereign execution evidence, audit ledger entries, canonical serialization, Merkle commitments, independent verification, witness binding, retention, recovery, and production claim gating  
**Version:** 1.1  
**Document Type:** Normative Security, Evidence, and Verification Specification  
**Applies To:** platform, platform.api, hardware authority, SIMD, AES, FHE, PQC, provider-mediated evidence paths, sealed capability capsule admission, runtime execution gates, audit verification, and external checkpointing

---

## 1. Purpose

The XPScerpto audit ledger is not a logging convenience. It is the sovereign evidence layer that makes execution decisions reconstructable, tamper-evident, and independently verifiable after execution.

The ledger does not grant authority. It records whether an authority decision, denial, execution claim, quarantine action, revocation event, or diagnostic outcome can be verified. A valid ledger entry cannot make an invalid capsule valid. A valid ledger entry cannot convert a non-sovereign execution into a sovereign one. However, a missing, corrupted, unverifiable, or executor-only ledger entry can prevent a production execution claim from being considered sovereign.

The normative audit principle is:

```text
The executor must not be the sole auditor.
```

A runtime that executes a protected operation MUST NOT be the only entity capable of verifying that operation's authority lineage.

This document defines the required ledger structure, entry classes, verification algorithm, witness binding, tamper evidence, replay detection, external checkpointing, retention behavior, recovery rules, domain-specific audit requirements, and production claim semantics for XPScerpto.

## 2. Normative Language

The terms **MUST**, **MUST NOT**, **REQUIRED**, **SHALL**, **SHALL NOT**, **SHOULD**, **MAY**, and **OPTIONAL** are normative.

A component that violates a **MUST** or **MUST NOT** requirement is non-conforming even if functional tests pass.

A production implementation MUST treat this document as an evidence and verification specification, not as an advisory logging guideline.

## 3. Normative References

This specification is part of the XPScerpto sovereign specification layer.

| Document | Role |
|---|---|
| XPS-SOV-TM-001 — Sovereign Threat Model | Defines threat boundary, adversaries, protected assets, and non-claims. |
| XPS-SOV-AUTH-002 — Sovereign Authority Model | Defines authority lineage, epochs, revocation, admission, and safe halt behavior. |
| XPS-SOV-CAP-003 — Sealed Capability Capsule Specification | Defines capsule format, validation, replay prevention, lifecycle, and production admission. |
| XPS-SOV-FAIL-004 — Failure Semantics and Recovery Specification | Defines deterministic failure behavior and recovery authority. |
| XPS-SOV-AUD-005 — Sovereign Audit Ledger and Verification Specification | Defines ledger evidence, verification, witness binding, retention, and production claim gating. |

If this document conflicts with another XPScerpto sovereign specification, the stricter authority-denying or claim-denying interpretation MUST apply.

## 4. Core Audit Principle

The audit ledger is evidence, not authority.

The ledger records whether a decision can be verified. It does not authorize the decision by itself. The ledger cannot rescue a malformed capsule, an invalid epoch, a missing hardware evidence binding, a wrong build fingerprint, a provider compromise, or a route that was not admitted by policy.

The following equation is normative:

```text
sovereign_execution_claim :=
    valid_authority_lineage
AND valid_capsule_admission
AND valid_runtime_decision
AND valid_pre_execution_audit_reservation
AND valid_execution_or_denial_witness
AND valid_ledger_entry
AND valid_merkle_commitment
AND independent_verification_possible
AND no_secret_material_exposed
AND no_executor_only_truth
```

If any predicate is false, the production execution claim MUST be rejected or marked non-sovereign.

## 5. Ledger Is Evidence, Not Authority

The ledger MUST NOT be interpreted as an authority source.

The following interpretations are forbidden:

1. A ledger entry MUST NOT make an invalid capsule valid.
2. A ledger entry MUST NOT grant execution authority.
3. A ledger entry MUST NOT replace policy validation.
4. A ledger entry MUST NOT replace build fingerprint validation.
5. A ledger entry MUST NOT replace epoch liveness checks.
6. A ledger entry MUST NOT replace hardware authority.
7. A ledger entry MUST NOT convert diagnostic evidence into production proof.
8. A ledger entry MUST NOT convert executor-only logs into sovereign audit evidence.
9. A ledger entry MUST NOT allow production execution when pre-execution audit reservation was required but absent.
10. A ledger entry MUST NOT permit a claim when the entry exposes secret material.

The ledger proves only that specific evidence was committed under specific rules. It does not create the authority it records.

## 6. Ledger Entry Classes

A conforming ledger MUST distinguish entry classes.

| Entry class | Purpose | Production claim support |
|---|---|---|
| `PRE_EXECUTION_RESERVATION` | Reserves an audit slot before protected execution begins. | required when policy demands evidence before execution |
| `ADMISSION_DECISION` | Records admit, deny, halt, quarantine, revoke, or non-sovereign decision. | supports decision reconstruction |
| `EXECUTION_WITNESS` | Records post-execution witness commitment. | supports execution claim if lineage verifies |
| `DENIAL_EVIDENCE` | Records deterministic denial before execution. | supports denial claim, not execution claim |
| `REVOCATION_EVIDENCE` | Records capsule, epoch, issuer, route, or provider revocation. | supports revocation claim |
| `QUARANTINE_EVIDENCE` | Records isolation of route, provider, capsule, evidence, or ledger segment. | blocks dependent production claims |
| `LEDGER_TAMPER_INCIDENT` | Records detected ledger mutation, fork, deletion, or replay. | blocks affected production claims |
| `EXTERNAL_CHECKPOINT` | Publishes or binds a ledger root outside executor-only control. | strengthens independent verification |
| `RECOVERY_EVIDENCE` | Records authorized repair, reconciliation, or recovery action. | supports post-incident verification only if externally verifiable |
| `DIAGNOSTIC_NON_PRODUCTION` | Records diagnostic evidence that cannot support production claims. | no production claim support |

A production verifier MUST reject any production execution claim that relies only on diagnostic, executor-local, or non-production entries.

## 7. Ledger Entry State Machine

A production ledger entry SHOULD follow the following state machine or one with equivalent semantics:

```text
ENTRY_REQUESTED
  -> ENTRY_RESERVED
  -> PRE_EXECUTION_COMMITTED
  -> EXECUTION_OBSERVED
  -> WITNESS_BOUND
  -> ENTRY_SEALED
  -> ROOT_PUBLISHED
  -> EXTERNALLY_CHECKPOINTED
  -> VERIFIED_OR_REJECTED
```

A production execution MUST NOT begin before `ENTRY_RESERVED` or an equivalent pre-execution audit commitment exists when policy requires evidence before execution.

A ledger entry MUST NOT transition from rejected, tampered, quarantined, or invalid back to verified without authorized recovery evidence.

A verifier MUST NOT treat an entry as verified until the entry, parent root, Merkle path, signature or MAC, authority lineage, and witness commitments have been checked according to this specification.

## 8. Ledger Entry Structure

A production ledger entry MUST contain fields semantically equivalent to:

```text
LedgerEntry {
    u32      schema_version;
    u32      entry_class;
    u128     entry_id;
    u64      entry_sequence;

    u128     previous_entry_id;
    hash256  previous_entry_hash;
    hash256  parent_merkle_root;

    u128     audit_reservation_id;
    u128     capsule_id;
    hash256  capsule_hash;
    u32      issuer_id;
    u32      issuer_key_id;

    u64      authority_epoch;
    hash256  policy_hash;
    hash256  build_fingerprint;
    hash256  hardware_evidence_hash;
    hash256  domain_contract_hash;
    hash256  authority_lineage_hash;

    u32      execution_domain;
    u32      route_class;
    u32      kernel_class;
    hash256  memory_contract_hash;
    hash256  execution_hash;

    hash256  pre_execution_witness_hash;
    hash256  post_execution_witness_hash;
    hash256  witness_commitment;

    u32      decision_status;
    u32      failure_code;
    u32      failure_severity;
    bool     recovery_required;
    bool     production_claim_allowed;

    u64      timestamp_or_counter;
    u32      time_or_counter_source;

    u32      redaction_policy;
    u32      retention_class;

    u32      ledger_writer_id;
    u32      ledger_writer_key_id;
    u32      verifier_id;
    hash256  verifier_policy_hash;
    bytes    verifier_commitment;

    u128     external_checkpoint_id;
    hash256  external_checkpoint_root;
    bytes    external_checkpoint_signature;

    bytes    entry_signature_or_mac;
}
```

This structure defines required semantics, not a mandatory ABI layout. Implementations MAY use a different binary layout only if canonical serialization and verification produce equivalent semantics.

A production implementation MUST NOT omit required evidence semantics by claiming storage or ABI freedom.

## 9. Field Semantics

| Field | Required semantics |
|---|---|
| `schema_version` | Ledger schema version. Unknown production-critical schema versions MUST be rejected by verifiers. |
| `entry_class` | Declares the purpose of the entry: reservation, admission, witness, denial, revocation, quarantine, checkpoint, recovery, or diagnostic. |
| `entry_id` | Unique ledger entry identifier. Duplicate identifiers MUST be rejected or marked tamper/replay evidence. |
| `entry_sequence` | Monotonic sequence value or policy-declared ordering value. Sequence gaps MUST be detectable. |
| `previous_entry_id` | Identifier of the previous entry, when required by ordering policy. |
| `previous_entry_hash` | Hash of the previous canonical entry, when required by chaining policy. |
| `parent_merkle_root` | Previous ledger root or checkpoint root. Must match accepted ledger state. |
| `audit_reservation_id` | Reservation identifier created before protected execution when required by policy. |
| `capsule_id` | Capsule that authorized or attempted to authorize the execution. May be null only for pre-capsule denial evidence. |
| `capsule_hash` | Hash of the capsule or capsule commitment used for admission verification. |
| `issuer_id` | Capsule issuer or authority issuer identifier when parseable or applicable. |
| `issuer_key_id` | Key identifier used to issue, sign, or seal the capsule. |
| `authority_epoch` | Epoch snapshot used for admission. |
| `policy_hash` | Policy root hash used for the decision. |
| `build_fingerprint` | Build identity used for the decision. |
| `hardware_evidence_hash` | Hash of platform-authorized hardware evidence used for route decision. |
| `domain_contract_hash` | Hash of the domain execution contract in force at admission. |
| `authority_lineage_hash` | Hash of the reconstructed authority lineage. |
| `execution_domain` | Domain of execution or attempted execution. |
| `route_class` | Route class selected or denied. |
| `kernel_class` | Kernel class selected or denied. |
| `memory_contract_hash` | Hash of memory behavior, isolation, alignment, and access contract when relevant. |
| `execution_hash` | Hash of the request, decision, route, result commitment, and non-secret execution context. |
| `pre_execution_witness_hash` | Commitment to the pre-execution admission witness. |
| `post_execution_witness_hash` | Commitment to the post-execution witness. |
| `witness_commitment` | Stable commitment to independent witness material. |
| `decision_status` | Admitted, denied, halted, quarantined, revoked, non-sovereign, diagnostic, or recovered. |
| `failure_code` | Deterministic failure code if the decision was not admitted. |
| `failure_severity` | Severity classification from the failure semantics specification. |
| `recovery_required` | Whether the entry requires authorized recovery before dependent claims can proceed. |
| `production_claim_allowed` | Explicit gate indicating whether this entry may support a production claim. |
| `timestamp_or_counter` | Timestamp, monotonic counter, ledger counter, or authority counter. Source must be declared by policy. |
| `time_or_counter_source` | Declares the time or counter source used. |
| `redaction_policy` | Declares how secret or sensitive evidence has been redacted or committed. |
| `retention_class` | Retention class required for the entry. |
| `ledger_writer_id` | Identifier of the ledger writer. |
| `ledger_writer_key_id` | Key identifier used by the ledger writer. |
| `verifier_id` | Independent verifier identifier, if available. |
| `verifier_policy_hash` | Policy under which the verifier evaluated the entry. |
| `verifier_commitment` | Commitment from independent verifier or verification role. |
| `external_checkpoint_id` | Identifier for an external checkpoint event, if applicable. |
| `external_checkpoint_root` | Root published to an external checkpoint service. |
| `external_checkpoint_signature` | Signature or integrity proof from external checkpoint service. |
| `entry_signature_or_mac` | Integrity protection over canonical ledger entry body and context. |

## 10. Append-Only Guarantees

A conforming ledger MUST provide append-only semantics.

Append-only means:

1. Existing entries MUST NOT be silently modified.
2. Entry order MUST be cryptographically committed.
3. Deletion MUST be detectable by root mismatch, checkpoint mismatch, sequence gap, or missing parent.
4. Reordering MUST be detectable by parent root mismatch, previous entry hash mismatch, or sequence conflict.
5. Duplicate entry identifiers MUST be rejected or marked as replay/tamper evidence.
6. Ledger truncation MUST be detectable by external checkpoint comparison when checkpoints exist.
7. Forks MUST be detectable when the same parent root produces incompatible child roots.
8. Recovery MUST NOT rewrite history without producing explicit recovery evidence.
9. Diagnostic entries MUST NOT replace missing production entries.
10. Executor-only logs MUST NOT be substituted for missing append-only evidence.

Append-only does not require a specific storage backend. It requires verifiable ordering and tamper evidence.

## 11. Canonical Serialization

Canonical serialization MUST be deterministic.

A conforming production ledger serialization MUST satisfy:

1. Field order MUST be fixed by schema version.
2. Integer encoding MUST be fixed-width and endian-declared.
3. Hash fields MUST be fixed length.
4. Variable-length fields MUST include canonical length prefixes.
5. Duplicate fields MUST be rejected.
6. Missing required fields MUST be rejected.
7. Unknown critical fields MUST be rejected.
8. Unknown non-critical fields MAY be ignored only if policy permits and signature covers them.
9. Parsers MUST reject trailing garbage unless extension blocks are explicitly permitted.
10. Textual encodings MUST NOT be used for production verification unless canonical byte normalization is specified.
11. Non-minimal encodings MUST be rejected unless schema explicitly permits them.
12. Extension blocks MUST be covered by signature, MAC, or signed extension hash.
13. Canonical serialization MUST be identical for writer and verifier.
14. The canonical parser is part of the security boundary.

A verifier MUST reject ledger entries that cannot be parsed canonically.

## 12. Merkle Commitment Rules

Each ledger update MUST compute a new root equivalent to:

```text
entry_hash = H("XPS_LEDGER_ENTRY_V1" || canonical_entry_without_signature)
new_root   = MerkleAppend(parent_merkle_root, entry_hash)
```

Rules:

1. The hash algorithm MUST be policy-approved.
2. Domain separation strings MUST be fixed and versioned.
3. Parent root MUST match the current accepted ledger root.
4. Entry hash MUST be recomputable from canonical entry bytes.
5. Forks MUST be detected by conflicting child roots from the same parent root.
6. A verifier MUST be able to recompute roots from canonical entries and Merkle proofs.
7. The Merkle append function MUST be specified by policy or implementation profile.
8. A missing parent root in a production entry MUST be treated as insufficient or invalid evidence.
9. A root mismatch MUST produce deterministic ledger failure.

## 13. Signature and MAC Semantics

The ledger entry signature or MAC MUST bind:

```text
context_string
schema_version
entry_class
canonical_entry_body_without_signature
entry_id
entry_sequence
parent_merkle_root
entry_hash
policy_hash
build_fingerprint
authority_epoch
capsule_id
execution_domain
route_class
kernel_class
decision_status
failure_code
witness_commitment
production_claim_allowed
```

The context string MUST distinguish XPScerpto ledger entries from all other signed objects.

The signature or MAC algorithm MUST be allowed by active policy.

Algorithm downgrade is forbidden.

Entry integrity is not the same as independent verification. A valid entry signature proves entry integrity under the writer's key. It does not prove that the executor was not the sole auditor unless an independent verifier, external checkpoint, or policy-approved separation exists.

## 14. Ledger Authority and Key Lifecycle

A production implementation MUST define a lifecycle for ledger writers, verifiers, and checkpoint services.

The lifecycle MUST include:

1. ledger writer identity;
2. ledger writer key identity;
3. verifier identity;
4. verifier key identity;
5. external checkpoint service identity;
6. key activation;
7. key expiry;
8. key rotation;
9. key revocation;
10. compromise response;
11. emergency checkpoint disablement;
12. audit record for key changes;
13. policy binding for each role.

### 14.1 Key Separation Rules

A runtime executor key MUST NOT be sufficient by itself to establish sovereign audit truth.

The following collapses are forbidden for production claims unless policy provides an independently verifiable separation mechanism:

1. executor as sole writer and sole verifier;
2. executor-only local log as audit ledger;
3. writer key used as capsule issuer key without policy separation;
4. verifier key controlled only by execution kernel;
5. checkpoint service controlled only by executor process;
6. debug key accepted as production ledger authority;
7. test key accepted as production verifier key.

A ledger writer may append evidence. A verifier validates evidence. A checkpoint service publishes or stores roots. These roles MUST NOT collapse into unchecked executor-only truth.

## 15. Tamper Evidence

The ledger MUST detect the following tamper classes.

| Tamper class | Detection method |
|---|---|
| entry body mutation | entry signature or MAC mismatch and entry hash mismatch |
| entry deletion | Merkle root mismatch, checkpoint mismatch, sequence gap, or missing parent |
| entry reordering | parent root mismatch, sequence conflict, or previous hash mismatch |
| fabricated entry | invalid entry signature or unknown writer |
| replayed entry | duplicate entry id, duplicate capsule id, or parent root conflict |
| forked ledger | incompatible roots for same parent or checkpoint |
| missing denial evidence | policy-defined audit gap detection |
| diagnostic substitution | entry class mismatch and production claim rejection |
| witness substitution | witness commitment mismatch |
| secret-bearing evidence | redaction policy violation and quarantine |

A tampered ledger segment MUST trigger:

```text
LEDGER_CHAIN_INVALID
FORENSIC_ESCALATION
AUDIT_CHAIN_SUSPENDED
```

where applicable under failure semantics.

## 16. Fork Detection and Resolution

A ledger fork occurs when ledger history cannot be uniquely reconciled under the accepted parent root, sequence, checkpoint, or external verification state.

| Fork condition | Required code | Required response |
|---|---|---|
| same parent root, two child roots | `LEDGER_FORK_DETECTED` | forensic escalation and suspend affected chain |
| same sequence, different entry hash | `LEDGER_SEQUENCE_CONFLICT` | reject conflicting segment |
| same entry id, different entry body | `LEDGER_DUPLICATE_ENTRY_ID` | reject or quarantine |
| same one-time capsule id, multiple admissions | `LEDGER_DUPLICATE_CAPSULE_ADMISSION` | replay rejection |
| external checkpoint root mismatch | `LEDGER_CHECKPOINT_CONFLICT` | suspend production claims |
| conflicting verifier commitments | `LEDGER_VERIFIER_CONFLICT` | forensic escalation |
| missing ancestor entry | `LEDGER_SEQUENCE_GAP` | insufficient evidence or chain invalid |

A fork MUST block dependent production claims until resolved by policy-authorized recovery.

A fork MUST NOT be resolved solely by the executor.

## 17. Replay Verification

A verifier MUST reject or mark suspicious:

1. duplicate `entry_id`;
2. duplicate one-time `capsule_id` admission;
3. entry timestamp outside capsule validity window;
4. entry epoch not live at decision time;
5. entry parent root not matching known ledger state;
6. entry policy hash inconsistent with capsule;
7. entry build fingerprint inconsistent with capsule;
8. entry hardware evidence hash inconsistent with capsule;
9. entry domain contract hash inconsistent with capsule;
10. entry authority lineage hash inconsistent with reconstructed lineage;
11. conflicting decision status for the same one-time execution;
12. duplicate audit reservation id used for more than one protected execution.

Replay verification MUST be possible without trusting the executor's memory.

## 18. External Verification Algorithm

An independent verifier SHOULD be able to verify an execution claim using:

```text
1. declared policy root
2. build fingerprint or build attestation record
3. hardware evidence hash or evidence descriptor
4. epoch state or epoch checkpoint
5. sealed capsule or capsule commitment
6. ledger entry
7. canonical ledger entry bytes
8. entry signature or MAC validation material
9. Merkle proof from entry to published root
10. external checkpoint if available
11. witness data or witness commitment
12. authority lineage reconstruction rules
```

A verifier MUST perform at least:

```text
1. parse canonical ledger entry
2. verify schema version
3. verify entry class
4. verify entry signature or MAC
5. recompute entry_hash
6. verify parent root and Merkle path
7. verify external checkpoint if required
8. reconstruct authority lineage
9. compare policy, build, hardware, epoch, capsule, domain, route, and kernel bindings
10. verify witness commitment
11. evaluate decision_status and failure_code
12. evaluate production_claim_allowed
13. produce final verification result
```

Verification failure MUST NOT be converted into partial verification for production claims.

## 19. External Verification Results

Verification result MUST be one of:

| Result | Meaning | Production claim allowed |
|---|---|---|
| `VERIFIED_SOVEREIGN_EXECUTION` | Authority lineage, capsule admission, ledger commitment, witness, and verification all succeed. | yes |
| `VERIFIED_DENIAL` | Denial, halt, revocation, quarantine, or rejection evidence verifies. | no execution claim; denial claim allowed |
| `VERIFIED_REVOCATION` | Revocation evidence verifies. | no execution claim |
| `VERIFIED_QUARANTINE` | Quarantine evidence verifies. | no execution claim for affected object |
| `NON_SOVEREIGN_EXECUTION` | Execution occurred or is claimed but required evidence is missing. | no |
| `LEDGER_INVALID` | Ledger integrity check fails. | no |
| `INSUFFICIENT_EVIDENCE` | Verifier lacks required inputs. | no |
| `DIAGNOSTIC_ONLY` | Entry is diagnostic or non-production. | no |
| `EXECUTOR_ONLY_LOG` | Evidence is controlled only by executor. | no |
| `SECRET_EXPOSURE_REJECTED` | Evidence exposes secret material. | no |

A production claim is allowed only for `VERIFIED_SOVEREIGN_EXECUTION`.

## 20. Independent Audit Roles

The executor MUST NOT be the sole auditor.

A production deployment MUST define at least one independent audit role.

| Role | Capability | Must not have |
|---|---|---|
| runtime executor | performs admitted execution | sole control over audit truth |
| ledger writer | appends entries | authority to forge capsules |
| verifier | validates lineage and ledger roots | authority to execute protected operation |
| policy auditor | validates policy root and epoch transitions | ability to silently rewrite runtime decisions |
| external checkpoint service | stores or publishes ledger roots/checkpoints | access to secret execution material |
| recovery authority | authorizes repair or reconciliation | ability to rewrite history without evidence |

For small deployments, roles MAY be implemented by different processes, keys, storage roots, hardware compartments, review domains, or policy-controlled authority boundaries. They MUST NOT collapse into unchecked executor-only truth for production claims.

## 21. Witness Binding

`witness_commitment` binds execution evidence to independently checkable material.

It MAY commit to:

1. request transcript hash;
2. known-answer test vector identifier;
3. ciphertext or plaintext commitment without secret disclosure;
4. kernel route proof;
5. dispatch plan hash;
6. timing or performance envelope commitment;
7. denial reason proof;
8. external verifier result;
9. source or build evidence bundle;
10. domain contract evidence;
11. secret absence witness;
12. post-execution result commitment.

Witness data MAY be stored outside the ledger. The ledger MUST commit to it with a stable hash and declared schema.

A missing or unverifiable witness MUST NOT support a production execution claim.

## 22. Production Claim Semantics

A ledger entry may support a production execution claim only when:

```text
entry_class supports production execution
AND decision_status == admitted_or_executed
AND production_claim_allowed == true
AND authority_lineage_hash verifies
AND capsule binding verifies
AND policy/build/hardware/domain/epoch bindings verify
AND witness commitment verifies
AND Merkle path verifies
AND external checkpoint verifies when required
AND no tamper, replay, fork, or secret exposure is detected
AND executor is not sole auditor
```

The following MUST NOT support production execution claims:

1. diagnostic entries;
2. non-production entries;
3. executor-only logs;
4. entries with missing witness commitment;
5. entries with unverifiable witness;
6. entries with invalid Merkle proof;
7. entries without required external checkpoint;
8. entries exposing secret material;
9. entries from a quarantined ledger segment;
10. entries from unresolved forked history;
11. entries with production_claim_allowed set to false;
12. entries generated after a failed audit reservation.

## 23. Evidence Privacy and Redaction

Ledger evidence MUST be sufficient for verification but MUST NOT expose secret material.

A ledger entry or witness MUST NOT contain:

1. plaintext secrets;
2. AES keys;
3. PQC private keys;
4. FHE secret keys;
5. raw user data;
6. secret-dependent traces;
7. raw hardware identifiers unless policy classifies them as safe;
8. provider-private material not approved for audit;
9. debug memory dumps;
10. unredacted diagnostic secret state.

A ledger entry that exposes secret material MUST be quarantined and MUST NOT support a production claim.

Secret-bearing witness data MUST be redacted, encrypted, access-controlled, or replaced by commitments according to policy.

Redaction MUST NOT make verification impossible unless the result is explicitly marked `INSUFFICIENT_EVIDENCE` or `DIAGNOSTIC_ONLY`.

## 24. Retention Semantics

Retention policy MUST define retention behavior by entry class.

| Entry class | Minimum retention requirement |
|---|---|
| production admitted execution | retain entry, witness commitment, and Merkle proof according to policy or legal requirement |
| production denial | retain denial evidence sufficient for forensic analysis |
| revocation | retain permanently or until superseded by signed archival policy |
| quarantine | retain until cleared, revoked, or incident closed |
| tamper evidence | retain until incident closure and external checkpoint reconciliation |
| external checkpoint | retain according to checkpoint policy |
| diagnostic non-production | retain only if policy requires traceability; MUST be marked non-production |
| recovery evidence | retain with the incident or chain it repairs |

Retention MUST NOT preserve secret material unnecessarily.

Secret-bearing witness data MUST be redacted, encrypted, or replaced by commitments.

Deletion of retained production evidence MUST be governed by signed archival policy or legal retention policy.

## 25. Authority Lineage Tracking

Each production ledger entry MUST bind to authority lineage.

The authority lineage hash MUST be equivalent to:

```text
authority_lineage_hash = H(
    "XPS_AUTHORITY_LINEAGE_V1" ||
    policy_hash ||
    build_fingerprint ||
    hardware_evidence_hash ||
    domain_contract_hash ||
    authority_epoch ||
    capsule_id ||
    execution_domain ||
    route_class ||
    kernel_class ||
    audit_reservation_id ||
    parent_merkle_root
)
```

If lineage reconstruction fails, the ledger entry cannot establish sovereign execution.

A verifier MUST NOT rely solely on a stored `authority_lineage_hash`. It MUST be able to reconstruct or validate the lineage from declared inputs or commitments.

## 26. Audit Failure Behavior

| Failure | Required response |
|---|---|
| parent root mismatch | reject entry, mark fork or tamper |
| invalid entry signature or MAC | reject entry |
| missing production audit commitment | production execution denied or marked non-sovereign |
| audit reservation failed | production execution denied |
| unverifiable witness commitment | mark evidence insufficient |
| missing witness commitment | mark evidence insufficient or non-sovereign |
| duplicate one-time capsule entry | replay rejected |
| ledger unavailable before production admission | deny production execution |
| external checkpoint required but unavailable | deny production claim or mark insufficient according to policy |
| executor-only log | not accepted as sovereign evidence |
| entry exposes secret material | quarantine evidence and reject production claim |
| fork detected | suspend affected chain and block dependent claims |
| writer key compromised | revoke writer and invalidate affected entries according to policy |
| verifier conflict | forensic escalation and suspend dependent claims |

## 27. Ledger Failure Code Registry

A conforming implementation MUST provide failure codes equivalent to:

| Failure | Code |
|---|---|
| parent root mismatch | `LEDGER_PARENT_ROOT_MISMATCH` |
| invalid entry signature | `LEDGER_ENTRY_SIGNATURE_INVALID` |
| invalid entry MAC | `LEDGER_ENTRY_MAC_INVALID` |
| duplicate entry id | `LEDGER_DUPLICATE_ENTRY_ID` |
| duplicate one-time capsule admission | `LEDGER_DUPLICATE_CAPSULE_ADMISSION` |
| sequence gap | `LEDGER_SEQUENCE_GAP` |
| sequence conflict | `LEDGER_SEQUENCE_CONFLICT` |
| fork detected | `LEDGER_FORK_DETECTED` |
| checkpoint mismatch | `LEDGER_CHECKPOINT_MISMATCH` |
| checkpoint unavailable | `LEDGER_CHECKPOINT_UNAVAILABLE` |
| witness missing | `LEDGER_WITNESS_MISSING` |
| witness unverifiable | `LEDGER_WITNESS_UNVERIFIABLE` |
| executor-only evidence | `EXECUTOR_ONLY_EVIDENCE_REJECTED` |
| secret leakage in evidence | `LEDGER_SECRET_MATERIAL_EXPOSURE` |
| unknown schema | `LEDGER_SCHEMA_UNSUPPORTED` |
| non-canonical entry | `LEDGER_NON_CANONICAL_ENTRY` |
| trailing garbage | `LEDGER_TRAILING_GARBAGE_REJECTED` |
| unknown critical extension | `LEDGER_CRITICAL_EXTENSION_UNKNOWN` |
| writer key revoked | `LEDGER_WRITER_KEY_REVOKED` |
| verifier key revoked | `LEDGER_VERIFIER_KEY_REVOKED` |
| recovery not authorized | `LEDGER_RECOVERY_AUTHORITY_INVALID` |
| production claim not allowed | `LEDGER_PRODUCTION_CLAIM_REJECTED` |
| authority lineage mismatch | `LEDGER_AUTHORITY_LINEAGE_MISMATCH` |

Failure codes MAY be numerically encoded, but their semantics MUST be stable and auditable.

## 28. Ledger Recovery and Repair

Ledger recovery MUST be policy-authorized, externally verifiable, and auditable.

| Condition | Recovery rule |
|---|---|
| entry mutation | reject affected entry or segment |
| root mismatch | suspend affected chain |
| checkpoint mismatch | forensic escalation and reconciliation |
| missing entry | mark insufficient evidence unless recovery proof exists |
| fork | choose policy-authorized branch or reject affected branches |
| secret leakage | quarantine evidence and rotate affected access boundary |
| writer key compromise | revoke writer and invalidate affected entries according to policy |
| verifier key compromise | revoke verifier and re-verify affected commitments if possible |
| missing witness | mark insufficient evidence or non-sovereign |
| external checkpoint unavailable | follow checkpoint-unavailable policy; do not silently claim production |

Ledger repair MUST NOT be performed solely by the executor.

Ledger repair MUST produce a `RECOVERY_EVIDENCE` entry.

Ledger repair MUST NOT rewrite history silently.

Ledger repair MUST NOT convert unverifiable execution into verified sovereign execution unless all missing verification predicates are restored and independently verifiable.

## 29. Domain-Specific Ledger Requirements

### 29.1 SIMD

A SIMD ledger entry MUST bind:

```text
route_class
kernel_class
hardware_evidence_hash
dispatch_plan_hash
no_local_probe_witness
domain_contract_hash
memory_contract_hash
```

A SIMD production claim MUST NOT be accepted if the ledger does not prove that the route was authority-routed and not locally inferred.

### 29.2 AES

An AES ledger entry MUST bind:

```text
algorithm_suite
key_scope_hash
route_class
kernel_class
no_key_material_exposed
side_channel_contract_hash_if_claimed
```

An AES ledger entry MUST NOT expose key material, plaintext secrets, or secret-dependent traces.

### 29.3 FHE

An FHE ledger entry MUST bind:

```text
bootstrap_stage
transform_claim
fixture_or_production_class
rotation_corpus_hash_if_required
secret_key_absence_witness
domain_contract_hash
production_claim_allowed
```

An FHE ledger entry MUST distinguish fixture, diagnostic, partial, and production claims.

A fixture ledger entry MUST NOT support:

```text
FHE_BOOTSTRAP_COMPLETE
FHE_FULL_EVALMOD
FHE_FULL_COEFF_TO_SLOT
FHE_FULL_SLOT_TO_COEFF
FHE_PRODUCTION_READY
```

unless a separate production verification chain proves those claims.

### 29.4 PQC

A PQC ledger entry MUST bind:

```text
algorithm_id
security_level
key_usage_scope
debug_key_absence_witness
private_key_absence_witness
domain_contract_hash
```

A PQC ledger entry MUST NOT expose private-key material or debug-key evidence as production proof.

### 29.5 Provider Path

A provider-related ledger entry MUST bind:

```text
provider_id
provider_fact_hash
provider_filter_policy_hash
evidence_source_class
platform_filter_result
dependent_capsule_ids_if_any
```

Provider facts MUST NOT become authority merely because they appear in the ledger.

## 30. External Checkpoint Policy

If external checkpointing is used or required, policy MUST define:

1. checkpoint frequency;
2. checkpoint root format;
3. checkpoint signature algorithm;
4. checkpoint service identity;
5. checkpoint storage boundary;
6. checkpoint publication path;
7. checkpoint reconciliation procedure;
8. checkpoint unavailability behavior;
9. checkpoint retention policy;
10. checkpoint compromise response.

If external checkpointing is required by policy and unavailable, production claims MUST be denied or marked insufficient according to policy.

A checkpoint service MUST NOT require access to secret execution material.

A checkpoint root MUST be sufficient to detect ledger truncation, fork, or incompatible rewrite.

## 31. Required Tests

A conforming implementation MUST test:

| Test | Expected result |
|---|---|
| mutate ledger entry | verification fails |
| delete entry from chain | root mismatch or sequence gap detected |
| reorder entries | parent root mismatch detected |
| duplicate entry id | duplicate rejected or marked tamper |
| replay capsule admission | replay rejected |
| use wrong policy hash | lineage verification fails |
| use wrong build fingerprint | lineage verification fails |
| use wrong hardware evidence hash | lineage verification fails |
| use wrong domain contract hash | lineage verification fails |
| missing witness commitment | evidence insufficient or non-sovereign |
| unverifiable witness commitment | evidence insufficient |
| executor-only log without ledger root | not accepted as sovereign evidence |
| forked ledger root | fork detected |
| external checkpoint mismatch | checkpoint conflict detected |
| invalid entry signature | verification fails |
| non-canonical entry | rejected |
| trailing garbage | rejected |
| secret material in evidence | quarantined and production claim rejected |
| diagnostic entry used for production claim | production claim rejected |
| missing pre-execution reservation | production claim rejected if policy requires reservation |
| writer key revoked | affected entries rejected or reclassified according to policy |
| recovery by executor only | recovery rejected |

Tests MUST assert not only error detection, but also that production claims are blocked when required.

## 32. Ledger Conformance Matrix

| ID | Requirement | Static check | Runtime check | Verification check | Evidence |
|---|---|---|---|---|---|
| LEDGER-REQ-001 | append-only required | schema and storage profile | parent root update | Merkle proof | root commitment |
| LEDGER-REQ-002 | executor must not be sole auditor | role configuration | writer/verifier separation | independent verifier | verifier commitment |
| LEDGER-REQ-003 | canonical entry required | parser tests | reject non-canonical | recompute hash | entry hash |
| LEDGER-REQ-004 | witness must be bound | witness schema | witness hash commit | verify witness | witness commitment |
| LEDGER-REQ-005 | production claim requires verified lineage | lineage schema | lineage hash | reconstruct lineage | authority_lineage_hash |
| LEDGER-REQ-006 | tamper blocks production claim | tamper tests | chain invalid | reject root | `LEDGER_CHAIN_INVALID` |
| LEDGER-REQ-007 | external checkpoint enforced when required | checkpoint policy | publish root | verify checkpoint | checkpoint commitment |
| LEDGER-REQ-008 | diagnostic evidence cannot support production | entry class scan | block claim | verify class | production claim rejection |
| LEDGER-REQ-009 | secret material must not appear in evidence | redaction scan | quarantine entry | reject claim | redaction policy |
| LEDGER-REQ-010 | fork detection required | parent/sequence scan | detect fork | reject conflicting roots | fork incident |
| LEDGER-REQ-011 | recovery must be authorized | recovery policy | recovery entry | verify recovery chain | recovery evidence |
| LEDGER-REQ-012 | domain-specific evidence required | domain schema | domain witness commit | verify domain binding | domain contract evidence |

## 33. Production Gate Flags

A conforming production ledger gate MUST be able to report equivalent final flags:

```text
LEDGER_SCHEMA_VALIDATED=YES
LEDGER_CANONICAL_SERIALIZATION_ENFORCED=YES
LEDGER_APPEND_ONLY_ENFORCED=YES
LEDGER_PARENT_ROOT_VERIFIED=YES
LEDGER_MERKLE_PROOF_VERIFIED=YES
LEDGER_ENTRY_SIGNATURE_VERIFIED=YES
LEDGER_AUTHORITY_LINEAGE_RECONSTRUCTED=YES
LEDGER_WITNESS_BOUND=YES
LEDGER_INDEPENDENT_VERIFICATION_AVAILABLE=YES
LEDGER_EXECUTOR_NOT_SOLE_AUDITOR=YES
LEDGER_EXTERNAL_CHECKPOINT_VERIFIED_IF_REQUIRED=YES
LEDGER_SECRET_SAFE_EVIDENCE_ENFORCED=YES
LEDGER_PRODUCTION_CLAIM_GATE_ENFORCED=YES
LEDGER_TAMPER_REJECTED=YES
LEDGER_FORK_DETECTED_OR_REJECTED=YES
LEDGER_DIAGNOSTIC_NOT_PRODUCTION=YES
```

The following states MUST block production claims:

```text
LEDGER_ENTRY_SIGNATURE_INVALID=YES
LEDGER_PARENT_ROOT_MISMATCH=YES
LEDGER_MERKLE_PROOF_INVALID=YES
LEDGER_CHAIN_INVALID=YES
LEDGER_FORK_UNRESOLVED=YES
LEDGER_WITNESS_MISSING=YES
LEDGER_WITNESS_UNVERIFIABLE=YES
LEDGER_EXECUTOR_ONLY_EVIDENCE=YES
LEDGER_EXTERNAL_CHECKPOINT_REQUIRED_BUT_MISSING=YES
LEDGER_AUTHORITY_LINEAGE_MISMATCH=YES
LEDGER_SECRET_MATERIAL_EXPOSURE=YES
LEDGER_DIAGNOSTIC_ENTRY_USED_FOR_PRODUCTION=YES
LEDGER_PRODUCTION_CLAIM_ALLOWED_FALSE=YES
```

## 34. Non-Claims

This specification does not claim that the ledger makes an execution valid.

It does not claim:

1. that a valid ledger entry grants authority;
2. that executor-only logs are sovereign evidence;
3. that diagnostic entries prove production execution;
4. that a Merkle root alone proves authority lineage;
5. that a signature alone proves independent verification;
6. that a missing witness can be ignored;
7. that a corrupted ledger segment can support production claims;
8. that a repaired ledger automatically validates past executions;
9. that secret-bearing evidence is acceptable;
10. that successful logging proves production readiness.

The ledger proves only that declared evidence was committed, chained, protected, and verifiable under the rules of this specification.

## 35. Final Conformance Statement

A ledger implementation conforms only if an independent verifier can reconstruct the decision lineage and verify tamper-evident ordering without trusting the executor as the sole source of truth.

A conforming implementation MUST prove:

```text
XPS_LEDGER_SCHEMA_ENFORCED=YES
XPS_LEDGER_CANONICAL_SERIALIZATION_ENFORCED=YES
XPS_LEDGER_APPEND_ONLY_ENFORCED=YES
XPS_LEDGER_MERKLE_COMMITMENT_ENFORCED=YES
XPS_LEDGER_SIGNATURE_OR_MAC_VERIFIED=YES
XPS_LEDGER_AUTHORITY_LINEAGE_BOUND=YES
XPS_LEDGER_WITNESS_BOUND=YES
XPS_LEDGER_INDEPENDENT_AUDIT_ENFORCED=YES
XPS_LEDGER_EXECUTOR_NOT_SOLE_AUDITOR=YES
XPS_LEDGER_TAMPER_DETECTED=YES
XPS_LEDGER_FORK_DETECTED=YES
XPS_LEDGER_REPLAY_DETECTED=YES
XPS_LEDGER_EXTERNAL_CHECKPOINT_SUPPORTED_IF_REQUIRED=YES
XPS_LEDGER_SECRET_SAFE_EVIDENCE_ENFORCED=YES
XPS_LEDGER_PRODUCTION_CLAIM_GATE_ENFORCED=YES
XPS_LEDGER_DIAGNOSTIC_OVERCLAIM_REJECTED=YES
XPS_LEDGER_RECOVERY_AUTHORITY_ENFORCED=YES
```

A production execution claim that cannot be reconstructed and independently verified from ledger evidence is not sovereign.

A ledger that is controlled only by the executor is not sovereign audit evidence.

A ledger entry that exposes secrets cannot support a production claim.

A diagnostic or non-production entry cannot support a production execution claim.

The XPScerpto audit ledger is valid only when it is canonical, append-only, tamper-evident, witness-bound, lineage-bound, independently verifiable, secret-safe, and capable of denying false production claims.
