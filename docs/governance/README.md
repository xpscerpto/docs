# XPScerpto Governance Documents

**Status:** governance documentation index  
**Scope:** RFC process, decision records, architectural decisions, authority-sensitive changes, ownership-sensitive changes, and validation-sensitive changes

This directory contains governance process documents that support the repository-wide governance model.

Root-level documents such as `GOVERNANCE.md`, `REVIEW_POLICY.md`, `CI_VALIDATION_GATES.md`, `AUTHORITY_MODEL.md`, and `MODULE_BOUNDARIES.md` define repository-wide policy. The files in this directory define how long-term governance decisions are proposed, reviewed, accepted, rejected, and recorded.

This directory is not a validation report, security proof, production-readiness claim, or acceptance artifact.

## 1. Documents in this directory

| File | Role |
|---|---|
| `rfc-process.md` | Defines when an RFC is required and how major proposals are reviewed before implementation |
| `decision-record-template.md` | Template for recording accepted, rejected, superseded, or long-term architectural decisions |

## 2. Relationship to root-level governance

The root-level governance files remain the repository-wide policy sources.

Relevant root-level documents include:

```text
GOVERNANCE.md
REVIEW_POLICY.md
CI_VALIDATION_GATES.md
AUTHORITY_MODEL.md
ARCHITECTURE.md
MODULE_BOUNDARIES.md
THREAT_MODEL.md
SECURITY.md
OWNERSHIP_MODEL.md
```

The files in this directory do not override those documents. They provide the process for handling changes that are too important to be decided only inside a normal pull request.

If a document in this directory conflicts with a root-level policy file, the root-level policy remains authoritative and this directory should be updated.

## 3. When to use the RFC process

Use `rfc-process.md` when a proposed change affects long-term direction or protected surfaces, including:

- new public API surfaces;
- public API ownership changes;
- platform authority or provider model changes;
- hardware evidence or routing capsule changes;
- runtime admission logic changes;
- cryptographic subsystem architecture;
- FHE bootstrap or canonical surface changes;
- dependency policy;
- no-exception or no-RTTI policy;
- build graph materialization;
- CI validation gates;
- audit ledger schemas;
- security disclosure policy;
- documentation that changes project claims.

The RFC process is used before implementation when a decision needs architecture, security, validation, and ownership review.

## 4. When to use a decision record

Use `decision-record-template.md` when a decision has long-term architectural, security, ownership, authority, validation, or compatibility impact.

Decision records are appropriate for:

- accepted RFCs;
- rejected alternatives that should not be reopened casually;
- superseded architecture decisions;
- public API ownership decisions;
- protected-surface governance decisions;
- validation gate decisions;
- migration and compatibility decisions.

A decision record documents the decision. It does not prove implementation, production readiness, sanitizer cleanliness, full graph truth, or release acceptance.

## 5. RFC versus decision record

The relationship is:

```text
RFC = proposal and review path before implementation
Decision Record = final record of the accepted, rejected, or superseded decision
```

An RFC may lead to a decision record. A decision record may reference an RFC.

A normal pull request may not require an RFC or decision record unless it affects protected long-term policy, ownership, architecture, authority, security, validation, or public claims.

## 6. Claim boundary

Neither an accepted RFC nor an accepted decision record is a production-readiness claim.

A decision may authorize implementation work, but implementation must still pass the relevant validation gates.

The following claims require evidence from the relevant gates:

```text
PRODUCTION_READY
FULL_SANITIZER_CTEST_CLAIMED
FULL_GRAPH_ACCEPTED
FHE_PRODUCTION_READY
FHE_BOOTSTRAP_COMPLETE
```

Governance acceptance is not implementation acceptance.

## 7. Final rule

The final rule for this directory is:

```text
major changes require explicit proposal
long-term decisions require durable records
accepted decisions still require validation evidence
process documents do not replace root-level governance
```

Use these files to make important decisions reviewable, auditable, and resistant to accidental overclaim.
