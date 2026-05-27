# XPScerpto Formal Sovereign Specification Layer

**Status:** normative security specification package  
**Scope:** formal sovereign security specifications, authority model, sealed capability capsule model, failure semantics, audit ledger schema, runtime enforcement mapping, contribution threat notes, and phase summary

This directory contains the canonical Markdown source files for the XPScerpto Formal Sovereign Specification Layer.

The files in this directory are part of the project’s security specification layer. They do not replace the root-level repository policy documents. Root-level documents define repository-wide policy; this directory provides the detailed formal security specification companion.

This package is not a validation report, security proof, production-readiness claim, or acceptance artifact.

## 1. Relationship to root-level policy

Root-level documents remain the repository-wide canonical policy sources.

Relevant root-level policy documents include:

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
CONTRIBUTING.md
OWNERSHIP_MODEL.md
```

This directory expands selected security, authority, failure, audit, and runtime-enforcement concepts into detailed formal specification documents.

If a document in this directory appears to conflict with a root-level repository policy document, the root-level policy defines repository-wide authority and this package should be updated to align with it.

## 2. Canonical specification files

The canonical files in this package are:

```text
docs/security/01_threat_model.md
docs/security/02_authority_model.md
docs/security/03_capsule_specification.md
docs/security/04_failure_semantics.md
docs/security/05_audit_ledger_schema.md
docs/security/06_runtime_enforcement_mapping.md
docs/security/phase_summary.md
docs/security/contribution-threats.md
```

The corresponding formal specification chain is:

```text
TM-001
AUTH-002
CAP-003
FAIL-004
AUD-005
MAP-006
```

## 3. Specification index

| File | Specification ID | Role |
|---|---:|---|
| `01_threat_model.md` | XPS-SOV-TM-001 v1.1 | Formal sovereign threat model |
| `02_authority_model.md` | XPS-SOV-AUTH-002 v1.1 CLEAN | Formal sovereign authority model |
| `03_capsule_specification.md` | XPS-SOV-CAP-003 v1.1 | Sealed capability capsule specification |
| `04_failure_semantics.md` | XPS-SOV-FAIL-004 v1.1 | Failure semantics and recovery specification |
| `05_audit_ledger_schema.md` | XPS-SOV-AUD-005 v1.1 | Sovereign audit ledger and verification schema |
| `06_runtime_enforcement_mapping.md` | XPS-SOV-MAP-006 v1.1 | Runtime enforcement mapping and release-gate specification |
| `phase_summary.md` | package summary | Phase summary for the specification foundation |
| `contribution-threats.md` | security reference | Contribution and review threat reference |

## 4. Clean authority model note

The authority model included in this package is the cleaned v1.1 version.

The reference chain is aligned to:

```text
TM-001 / AUTH-002 / CAP-003 / FAIL-004 / AUD-005 / MAP-006
```

The old v1.0 authority model is not used as a final source in this package.

## 5. Package layout

Current package layout:

```text
docs/security/
  README.md
  01_threat_model.md
  02_authority_model.md
  03_capsule_specification.md
  04_failure_semantics.md
  05_audit_ledger_schema.md
  06_runtime_enforcement_mapping.md
  phase_summary.md
  contribution-threats.md
```

There is no active `deliverables/` directory in the current clean package.

There is no active `docx/`, `txt/`, `zips/`, or `manifests/` directory in the current clean package.

The Markdown files listed above are the canonical package sources.

## 6. Deliverable policy

Previous export-style package layouts may have contained files such as:

```text
deliverables/docx/
deliverables/markdown/
deliverables/txt/
deliverables/zips/
manifests/
```

Those paths are not part of the current clean canonical package.

If export artifacts are regenerated in the future, they should be treated as generated release artifacts, not as canonical edit sources.

Generated deliverables should not be edited independently from the canonical Markdown sources.

## 7. Production claim boundary

This package is a normative specification foundation.

It does not by itself claim:

- compile guard closure;
- runtime gate closure;
- audit verifier implementation;
- CI evidence bundle implementation;
- sanitizer runtime closure;
- full graph acceptance;
- production sovereign closure;
- production readiness.

Specification acceptance is not implementation acceptance.

A specification may define what must be enforced. It does not prove that enforcement has been implemented.

## 8. Security and validation boundary

This package supports the repository’s security architecture, but it does not replace:

```text
SECURITY.md
THREAT_MODEL.md
CI_VALIDATION_GATES.md
REVIEW_POLICY.md
```

Use `SECURITY.md` for vulnerability reporting.

Use root `THREAT_MODEL.md` for the repository-wide current threat model.

Use `CI_VALIDATION_GATES.md` for validation-gate requirements.

Use `REVIEW_POLICY.md` for review acceptance discipline.

## 9. Current claim boundaries

This package does not elevate project status.

Unless the relevant acceptance gates prove otherwise, conservative global boundaries remain in force:

```text
PRODUCTION_READY = NO
FULL_SANITIZER_CTEST_CLAIMED = NO
FULL_GRAPH_ACCEPTED = NO
```

Subsystem-specific boundaries remain scoped to their own gates. Examples include:

```text
FHE_PRODUCTION_READY = NO
FHE_BOOTSTRAP_COMPLETE = NO
```

## 10. Final rule

The final rule for this directory is:

```text
canonical security specifications live here
root-level policy remains repository-wide authority
generated deliverables are not canonical sources
specification text is not implementation proof
```

This package is valid only when its specification chain, root-level policy references, validation boundaries, and production-claim boundaries remain aligned.
