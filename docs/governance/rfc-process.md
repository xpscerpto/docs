# XPScerpto RFC Process

**Status:** normative governance process  
**Scope:** architecture changes, authority changes, public API changes, ownership changes, dependency policy, validation gates, security-sensitive process changes, and long-term repository direction

An RFC is required when a proposed change is too important to be decided only inside a normal pull request.

The RFC process exists to make significant changes explicit before implementation. It gives maintainers time to evaluate authority impact, module-boundary impact, security impact, validation requirements, compatibility consequences, and claim boundaries.

This document is not a validation report, security proof, production-readiness claim, or acceptance artifact.

## 1. Purpose

The purpose of the RFC process is to prevent large or long-term decisions from being hidden inside implementation patches.

A normal pull request may be enough for small, local, well-scoped changes. An RFC is required when a change affects architecture, authority, public surfaces, ownership, validation gates, dependency policy, security posture, release claims, or long-term direction.

## 2. Relationship to root-level policy

The RFC process is governed by repository-wide policy documents, including:

```text
GOVERNANCE.md
ENGINEERING_PRINCIPLES.md
AUTHORITY_MODEL.md
ARCHITECTURE.md
MODULE_BOUNDARIES.md
THREAT_MODEL.md
SECURITY.md
CI_VALIDATION_GATES.md
REVIEW_POLICY.md
OWNERSHIP_MODEL.md
```

An RFC must not be used to bypass those documents. If an RFC conflicts with root-level governance, the RFC must be revised or rejected unless the governance documents are intentionally updated through the appropriate review process.

## 3. When an RFC is required

Open an RFC for changes that affect any of the following:

- new public API surfaces;
- public API ownership transfer;
- platform authority or provider model changes;
- hardware evidence model changes;
- routing capsule or dispatch model changes;
- runtime admission logic;
- SIMD authority or routing behavior;
- cryptographic subsystem architecture;
- FHE bootstrap or canonical FHE public surfaces;
- audit ledger schema;
- security disclosure policy;
- dependency additions or dependency policy changes;
- no-exception or no-RTTI policy changes;
- build graph changes that alter target materialization or coverage;
- CI gate weakening, replacement, removal, or meaning changes;
- sanitizer policy changes;
- final flag semantics;
- release evidence requirements;
- documentation that changes public project status or claim boundaries.

When in doubt, open an RFC or ask maintainers whether one is required.

## 4. When an RFC is not required

An RFC is usually not required for:

- small typo fixes;
- local documentation clarifications that do not change policy or claims;
- small bug fixes that do not affect protected surfaces;
- tests that add coverage without changing validation meaning;
- internal refactors that do not change public surfaces, authority, boundaries, CI, security posture, or claims;
- routine dependency-free cleanup with no architecture impact.

A change that looks small may still require an RFC if it affects a protected surface or changes the meaning of validation, authority, security, ownership, or public claims.

## 5. RFC identifier convention

Each RFC should have a stable identifier.

Recommended format:

```text
RFC-YYYY-NNN-short-title
```

Examples:

```text
RFC-2026-001-platform-api-admission
RFC-2026-002-simd-routing-authority
RFC-2026-003-fhe-bootstrap-public-surface
```

The identifier should remain stable even if the RFC title changes.

## 6. RFC stages

An RFC should move through the following stages:

```text
Draft
  ↓
Triage
  ↓
Architecture Review
  ↓
Security Review
  ↓
Validation Plan Review
  ↓
Decision
  ↓
Implementation PR
  ↓
Decision Record
```

A stage may be skipped only when the skipped stage is clearly not relevant and reviewers agree. Skipping a stage must not be used to avoid security, authority, ownership, or validation review.

## 7. RFC status values

Use one of the following status values:

```text
Draft
In Review
Accepted
Rejected
Withdrawn
Superseded
Implemented
```

`Accepted` authorizes implementation work within the accepted scope. It does not mean implemented. `Implemented` does not imply production readiness unless the relevant gates prove that claim.

## 8. RFC document format

Each RFC should include:

```markdown
# RFC: <title>

**RFC ID:** RFC-YYYY-NNN-short-title
**Status:** Draft | In Review | Accepted | Rejected | Withdrawn | Superseded | Implemented
**Date opened:** YYYY-MM-DD
**Owner:** <name or team>
**Reviewers:** <names or teams>
**Affected surfaces:** <paths/modules>
**Contribution class:** A | B | C | D | E
**Supersedes:** <RFC or decision ID, if any>
**Superseded by:** <RFC or decision ID, if any>

## 1. Summary
## 2. Motivation
## 3. Non-goals
## 4. Current behavior
## 5. Proposed behavior
## 6. Authority impact
## 7. Module boundary impact
## 8. Security impact
## 9. Validation plan
## 10. Compatibility and migration
## 11. Alternatives considered
## 12. Rejection criteria
## 13. Rollback plan
## 14. Open questions
## 15. Claim boundary
```

The RFC should be precise enough that reviewers can understand what is being proposed without reading implementation code.

## 9. Review requirements

An RFC review should answer:

- Does the proposal preserve authority flow?
- Does it introduce a new owner or public surface?
- Does it require a dependency?
- Does it weaken fail-closed behavior?
- Does it require new CI gates?
- Does it require sanitizer or runtime attestation evidence?
- Does it affect public documentation claims?
- Can it be implemented without shims or adapters?
- Does it change production, release, or validation claims?
- Does it identify rejection and rollback criteria?

## 10. Acceptance

An RFC is accepted only when:

- affected owners agree;
- architecture review is complete when required;
- security review is complete when required;
- validation plan is specific;
- rejection criteria are defined;
- compatibility and rollback are addressed;
- claim boundaries are explicit;
- production-readiness claims are not implied by acceptance.

RFC acceptance authorizes implementation work. It does not merge code, prove implementation, or prove production readiness.

## 11. Rejection

An RFC should be rejected when it:

- bypasses authority;
- weakens canonical ownership;
- adds dependency risk without sufficient value;
- reduces validation truth;
- relies on undocumented runtime state;
- cannot be validated;
- conflicts with sovereign engineering principles;
- hides security impact;
- weakens fail-closed behavior;
- depends on vague compatibility behavior;
- overclaims production, release, or validation status.

A rejected RFC may be revised and resubmitted if the rejection criteria are addressed.

## 12. Withdrawn and superseded RFCs

A withdrawn RFC is closed by its owner before acceptance.

A superseded RFC is replaced by a later RFC or decision record.

Withdrawn or superseded RFCs should remain available for historical context unless they contain sensitive security details.

## 13. Decision records

Accepted RFCs that have long-term architectural, authority, ownership, security, validation, or compatibility effect must create a decision record using:

```text
docs/governance/decision-record-template.md
```

## 14. Claim boundary

An RFC does not itself establish production readiness, sanitizer cleanliness, full graph acceptance, cryptographic acceptance, FHE bootstrap completion, or release acceptance.

Default RFC claim boundary:

```text
This RFC authorizes review of a proposed direction. It does not by itself claim implementation acceptance, production readiness, sanitizer cleanliness, full graph truth, or release acceptance.
```

## 15. Final rule

The final RFC rule is:

```text
large decisions require proposal before implementation
accepted proposals still require implementation evidence
implementation evidence still requires validation gates
no RFC may overclaim project status
```

Use the RFC process to make important changes reviewable before they become code.
