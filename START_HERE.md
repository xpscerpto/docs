# XPScerpto — Start Here

XPScerpto is a governed sovereign cryptographic infrastructure platform.

It is not a loose collection of cryptographic utilities. The project is designed around explicit authority, controlled module boundaries, fail-closed behavior, and evidence-backed validation.

This document is the recommended entry point for understanding the repository. It is not a validation report, security proof, production-readiness claim, or acceptance artifact.

## Current status

XPScerpto is under active sovereign validation and hardening.

The project must not be described as production-ready unless the required validation gates prove that status through live evidence.

Current global claim boundaries are conservative by default:

- `FHE_PRODUCTION_READY = NO`
- `FHE_BOOTSTRAP_COMPLETE = NO`
- `FULL_SANITIZER_CTEST_CLAIMED = NO`

These flags may only be changed by the relevant acceptance gates when those gates are backed by live evidence, reviewed CI results, sanitizer runtime logs where applicable, reviewed merge records, and explicit claim-decision records. `final_flags` may summarize accepted results, but they must not be treated as the source of truth.

Any document, test, commit, release note, or report that claims otherwise must provide live evidence and must identify the exact gate that established the claim.

## How to read the documentation

Read the documentation in this order:

1. `GOVERNANCE.md`
2. `ENGINEERING_PRINCIPLES.md`
3. `AUTHORITY_MODEL.md`
4. `ARCHITECTURE.md`
5. `MODULE_BOUNDARIES.md`
6. `THREAT_MODEL.md`
7. `SECURITY.md`
8. `CI_VALIDATION_GATES.md`
9. `REVIEW_POLICY.md`
10. `CONTRIBUTING.md`
11. `OWNERSHIP_MODEL.md`
12. `CODEOWNERS`
13. `SUPPORT.md`
14. `LICENSE_STATUS.md`
15. `docs/hw/README.md`

For detailed formal sovereign security specifications, see:

```text
docs/security/README.md
```

For major architecture, authority, ownership, validation, or policy changes, see:

```text
docs/governance/README.md
```

For HW authority, intelligence, runtime artifacts, power/thermal authority, evidence custody, validation, integration, and traceability specifications, see:

```text
docs/hw/README.md
```

This order matters. Governance defines how claims are controlled. The engineering principles define the project discipline. The authority model explains where execution authority comes from. The architecture explains how the system is structured once those rules are understood.

## Governance process documents

Major architecture, authority, ownership, validation, security-process, dependency-policy, or public-surface changes may require the RFC process before implementation.

Use:

```text
docs/governance/rfc-process.md
```

for major proposals, and:

```text
docs/governance/decision-record-template.md
```

for accepted, rejected, superseded, or long-term decisions.

These process documents do not replace `GOVERNANCE.md` or `REVIEW_POLICY.md`. They define how important changes are proposed and recorded.

## HW subsystem documentation

The HW subsystem documentation is maintained under:

```text
docs/hw/
```

Start with:

```text
docs/hw/README.md
```

The HW documentation defines the hardware authority, platform input boundary, deterministic HW intelligence, signal admission, sensor confidence, capability authorization, pressure reasoning, stability reasoning, runtime snapshot model, decision capsule contract, routing envelope contract, power and thermal authority, evidence custody, validation matrix, integration contracts, and traceability matrix.

This directory is a subsystem specification layer. It does not replace root-level repository policy. Root-level governance, authority, architecture, module-boundary, security, and validation documents remain the repository-wide sources of authority.

## Core rule

No shortcut is acceptable if it weakens authority, evidence, fail-closed behavior, sanitizer truth, module boundaries, ownership discipline, or production claim boundaries.

A build pass without authority preservation is not success.

A test pass without evidence boundaries is not acceptance.

A sanitizer configuration without runtime execution is not sanitizer truth.

A local workaround that bypasses governance is not a fix.

A passing result that hides an unresolved blocker is not closure.

## Platform authority chain

The expected authority flow is:

```text
platform governance
→ platform.api
→ hardware authority
→ decision capsule
→ routing envelope
→ domain execution
→ audit and evidence
```

Domain code must not invent its own authority by probing the operating system, CPU, compiler environment, hidden runtime state, or local capability shortcuts.

Runtime decisions must be admitted through the governed platform and hardware evidence chain.

## Protected surfaces

Changes to protected surfaces require strict review and evidence.

Protected surfaces include, but are not limited to:

```text
include/xps/crypto/platform/
include/xps/crypto/platform.api/
include/xps/crypto/hw/
include/xps/crypto/simd/
include/xps/crypto/FHE/
include/xps/crypto/FHE/fhe.bootstrap.public_api.ixx
include/xps/crypto/FHE/fhe.surface.guard.ixx
CMakeLists.txt
tests/
tools/
docs/
docs/hw/
```

Protected surfaces must not be changed through aliases, shims, adapters, weakened tests, deleted assertions, fake fallbacks, metadata-only proofs, or production overclaims.

## Contribution rule

Every meaningful change must explain:

- affected surfaces
- authority impact
- module-boundary impact
- test evidence
- sanitizer status
- claim boundaries
- whether production flags remain unchanged
- exact blocker if the change is incomplete

Incomplete work must fail closed with a clear blocker. It must not be presented as acceptance, production readiness, bootstrap completion, or full graph truth.

## First document to review

After this file, review:

```text
GOVERNANCE.md
```

It defines the project’s decision authority, review rules, protected surfaces, claim discipline, evidence requirements, and merge expectations.

For the HW subsystem, review:

```text
docs/hw/README.md
```

after the root-level governance and authority documents are understood.

XPScerpto should not be reviewed as a conventional utility library. It should be reviewed as a governed cryptographic infrastructure system where correctness, authority, evidence, and claim discipline must remain aligned.
