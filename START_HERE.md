# XPScerpto — Start Here

XPScerpto is a governed sovereign cryptographic infrastructure platform.

It is not a loose collection of cryptographic utilities. The project is designed around explicit authority, controlled module boundaries, fail-closed behavior, and evidence-backed validation.

This document is the recommended entry point for understanding the repository. It is not a validation report, security proof, production-readiness claim, acceptance artifact, or substitute for the project’s governance and evidence records.

## Current status

XPScerpto is under active sovereign validation and hardening.

The project must not be described as production-ready unless the required validation gates establish that status through live evidence.

Current global claim boundaries are conservative by default:

```text
FHE_PRODUCTION_READY = NO
FHE_BOOTSTRAP_COMPLETE = NO
FULL_SANITIZER_CTEST_CLAIMED = NO
```

These flags may only be changed by the relevant acceptance gates when those gates are backed by live evidence, reviewed CI results, sanitizer runtime logs where applicable, reviewed merge records, and explicit claim-decision records.

`final_flags` files may summarize accepted results, but they must not be treated as the sole source of truth.

Any document, test, commit, release note, badge, status summary, or report that claims otherwise must identify the exact acceptance gate that established the claim and must provide the supporting evidence chain.

## How to read the documentation

Read the repository documentation in the following order:

| Order | Document                                                 | Purpose                                                                                                                                         |
| ----: | -------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------- |
|     1 | [`GOVERNANCE.md`](GOVERNANCE.md)                         | Defines decision authority, review rules, protected surfaces, claim discipline, evidence requirements, and merge expectations.                  |
|     2 | [`ENGINEERING_PRINCIPLES.md`](ENGINEERING_PRINCIPLES.md) | Defines the engineering discipline of the project: fail-closed behavior, evidence boundaries, strict validation, and no shortcut-based success. |
|     3 | [`AUTHORITY_MODEL.md`](AUTHORITY_MODEL.md)               | Explains where execution authority comes from and how authority is admitted, controlled, reviewed, and rejected.                                |
|     4 | [`ARCHITECTURE.md`](ARCHITECTURE.md)                     | Describes the repository architecture after the governance and authority model are understood.                                                  |
|     5 | [`MODULE_BOUNDARIES.md`](MODULE_BOUNDARIES.md)           | Defines module separation, import boundaries, protected surfaces, and forbidden cross-boundary shortcuts.                                       |
|     6 | [`THREAT_MODEL.md`](THREAT_MODEL.md)                     | Defines threat assumptions, protected assets, attacker capabilities, abuse cases, and validation expectations.                                  |
|     7 | [`SECURITY.md`](SECURITY.md)                             | Explains the security process, reporting expectations, disclosure handling, and security discipline.                                            |
|     8 | [`CI_VALIDATION_GATES.md`](CI_VALIDATION_GATES.md)       | Defines configure, build, test, sanitizer, evidence, and acceptance gate expectations.                                                          |
|     9 | [`REVIEW_POLICY.md`](REVIEW_POLICY.md)                   | Defines review requirements for protected surfaces, governance-sensitive changes, and claim-affecting modifications.                            |
|    10 | [`CONTRIBUTING.md`](CONTRIBUTING.md)                     | Explains how contributors should prepare changes without weakening authority, validation, ownership, or claim boundaries.                       |
|    11 | [`OWNERSHIP_MODEL.md`](OWNERSHIP_MODEL.md)               | Defines ownership responsibilities and review expectations for critical project areas.                                                          |
|    12 | [`CODEOWNERS`](CODEOWNERS)                               | Defines required ownership review paths for protected files and directories.                                                                    |
|    13 | [`SUPPORT.md`](SUPPORT.md)                               | Explains support expectations and where users or contributors should seek help.                                                                 |
|    14 | [`LICENSE_STATUS.md`](LICENSE_STATUS.md)                 | Defines the current license status and any licensing boundaries or unresolved licensing decisions.                                              |
|    15 | [`docs/hw/README.md`](docs/hw/README.md)                 | Entry point for the HW subsystem authority, evidence, runtime, validation, and traceability specifications.                                     |

For detailed sovereign security specifications, see:

[`docs/security/README.md`](docs/security/README.md)

For major architecture, authority, ownership, validation, or policy changes, see:

[`docs/governance/README.md`](docs/governance/README.md)

For HW authority, intelligence, runtime artifacts, power and thermal authority, evidence custody, validation, integration, and traceability specifications, see:

[`docs/hw/README.md`](docs/hw/README.md)

This reading order matters.

Governance defines how claims are controlled. Engineering principles define the project discipline. The authority model explains where execution authority comes from. Architecture explains how the system is structured once those rules are understood. Module boundaries then define where code may and may not cross responsibility lines.

## Governance process documents

Major architecture, authority, ownership, validation, security-process, dependency-policy, public-surface, or claim-affecting changes may require the RFC process before implementation.

Use:

[`docs/governance/rfc-process.md`](docs/governance/rfc-process.md)

for major proposals, and:

[`docs/governance/decision-record-template.md`](docs/governance/decision-record-template.md)

for accepted, rejected, superseded, or long-term decisions.

These process documents do not replace `GOVERNANCE.md` or `REVIEW_POLICY.md`. They define how important changes are proposed, reviewed, accepted, rejected, superseded, and recorded.

## HW subsystem documentation

The HW subsystem documentation is maintained under:

[`docs/hw/`](docs/hw/)

Start with:

[`docs/hw/README.md`](docs/hw/README.md)

The HW documentation defines hardware authority, platform input boundaries, deterministic HW intelligence, signal admission, sensor confidence, capability authorization, pressure reasoning, stability reasoning, runtime snapshot models, decision capsule contracts, routing envelope contracts, power and thermal authority, evidence custody, validation matrices, integration contracts, and traceability rules.

This directory is a subsystem specification layer. It does not replace root-level repository policy. Root-level governance, authority, architecture, module-boundary, security, and validation documents remain the repository-wide sources of authority.

## Core rule

No shortcut is acceptable if it weakens authority, evidence, fail-closed behavior, sanitizer truth, module boundaries, ownership discipline, or production claim boundaries.

A build pass without authority preservation is not success.

A test pass without evidence boundaries is not acceptance.

A sanitizer configuration without runtime execution is not sanitizer truth.

A local workaround that bypasses governance is not a fix.

A passing result that hides an unresolved blocker is not closure.

A metadata-only proof is not runtime proof.

A claim without an evidence chain is not an accepted claim.

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

Domain code must not invent its own authority by probing the operating system, CPU, compiler environment, hidden runtime state, local capability shortcuts, or unreviewed platform assumptions.

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

Protected surfaces must not be changed through aliases, shims, adapters, weakened tests, deleted assertions, fake fallbacks, metadata-only proofs, claim inflation, production overclaims, or hidden authority bypasses.

## Contribution rule

Every meaningful change must explain:

* affected surfaces
* authority impact
* module-boundary impact
* ownership impact
* test evidence
* sanitizer status
* claim boundaries
* whether production flags remain unchanged
* exact blocker if the change is incomplete

Incomplete work must fail closed with a clear blocker. It must not be presented as acceptance, production readiness, bootstrap completion, sanitizer truth, or full graph truth.

## Claim discipline

XPScerpto separates implementation progress from accepted validation.

The following are not equivalent:

```text
Configure success ≠ Build success
Build success ≠ Test success
Test success ≠ Runtime acceptance
Runtime acceptance ≠ Sanitizer truth
Target success ≠ Full graph truth
Documentation ≠ Implementation proof
UI status ≠ Validation authority
Claim ≠ Evidence
Evidence ≠ Independent validation
```

A precise, localized, evidence-backed partial result is acceptable.

A broad security claim without an unbroken evidence chain is not acceptable.

## First document to review

After this file, review:

[`GOVERNANCE.md`](GOVERNANCE.md)

It defines the project’s decision authority, review rules, protected surfaces, claim discipline, evidence requirements, and merge expectations.

For the HW subsystem, review:

[`docs/hw/README.md`](docs/hw/README.md)

after the root-level governance and authority documents are understood.

XPScerpto should not be reviewed as a conventional utility library.

It should be reviewed as a governed cryptographic infrastructure system where correctness, authority, evidence, ownership, validation, and claim discipline must remain aligned.
