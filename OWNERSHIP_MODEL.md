# XPScerpto Ownership Model

**Status:** normative ownership policy  
**Scope:** repository-wide ownership model, protected surface ownership, reviewer roles, authority-sensitive ownership, security review routing, validation ownership, documentation ownership, release ownership, and relationship to platform-specific review-routing files such as `CODEOWNERS`

XPScerpto is a governed sovereign cryptographic infrastructure platform. Ownership in XPScerpto is not only a repository-management convenience. It is part of the project’s governance, security, authority, validation, and claim-discipline model.

This document defines the platform-neutral ownership policy for the repository. It applies regardless of whether the repository is hosted on GitHub, GitLab, Gitea, Bitbucket, an internal forge, or a private review system.

This document is not a validation report, security proof, production-readiness claim, or acceptance artifact. It defines who is responsible for review ownership and protected-surface stewardship.

## 1. Purpose

The purpose of this document is to make ownership explicit and platform-neutral.

XPScerpto requires clear ownership because the repository contains protected surfaces where ordinary review is not enough. A change may affect authority flow, cryptographic behavior, hardware evidence, routing, CI truth, documentation claims, or production-readiness language.

Ownership exists to ensure that the right maintainers review the right changes before acceptance.

Ownership answers:

- who is responsible for a surface;
- who must review changes to that surface;
- who understands the authority and security impact;
- who can judge whether evidence is sufficient;
- who can reject overclaims;
- who can require additional validation;
- who preserves long-term consistency across the repository.

Ownership does not grant permission to bypass governance.

## 2. Ownership principle

The central ownership principle is:

```text
every protected surface must have a clear owner
every high-risk change must receive the right review
ownership routes review; it does not replace evidence
```

A change is not acceptable merely because an owner approved it.

A change is not acceptable merely because CI passed.

A change is acceptable only when ownership, review, validation evidence, authority boundaries, and claim discipline all align.

## 3. Platform-neutral policy

This document is independent of GitHub or any specific hosting platform.

`OWNERSHIP_MODEL.md` defines the policy.

`CODEOWNERS` is only one platform-specific implementation of this policy for GitHub.

If XPScerpto is hosted on another platform, this ownership model should still apply through that platform’s review rules, protected branch settings, approval policies, merge request approvals, or internal review process.

The policy must survive the platform.

## 4. Ownership vocabulary

### Owner

An owner is a person or team responsible for stewarding a surface.

Ownership means responsibility for review quality, boundary preservation, authority preservation, evidence discipline, and long-term consistency.

Ownership does not mean the owner may approve unsafe changes.

### Reviewer

A reviewer evaluates a change before acceptance.

A reviewer may be an owner, domain expert, security maintainer, validation maintainer, or governance maintainer depending on the change.

### Protected surface

A protected surface is a file, directory, public API, test suite, CI rule, tool, documentation file, or artifact whose change may affect authority, security, validation, ownership, or claims.

### Domain owner

A domain owner understands a specific subsystem, such as platform, hardware, SIMD, FHE, cryptographic primitives, CI, documentation, or release evidence.

### Security owner

A security owner reviews changes that affect security posture, vulnerability handling, cryptographic behavior, secret material, memory safety, authority, or claim discipline.

### Validation owner

A validation owner reviews tests, CI gates, sanitizer runs, evidence, final flags, graph materialization, and acceptance claims.

### Governance owner

A governance owner reviews policy, claim discipline, documentation hierarchy, review rules, contribution rules, and release-status language.

## 5. Ownership is not automatic acceptance

Ownership routes review. It does not grant automatic acceptance.

An owner must still apply:

- `GOVERNANCE.md`;
- `ENGINEERING_PRINCIPLES.md`;
- `AUTHORITY_MODEL.md`;
- `ARCHITECTURE.md`;
- `MODULE_BOUNDARIES.md`;
- `THREAT_MODEL.md`;
- `SECURITY.md`;
- `CI_VALIDATION_GATES.md`;
- `REVIEW_POLICY.md`;
- `CONTRIBUTING.md`.

If a change violates governance, weakens authority, bypasses module boundaries, hides evidence gaps, weakens tests, or overclaims project status, an owner must reject it or request changes.

## 6. Default ownership

Every file should have a default owner.

Default ownership ensures that unclassified files still receive review.

Recommended default owner group:

```text
sovereign maintainers
```

Default owners should route changes to more specific owners when the change affects a protected surface.

Default ownership must not be used to avoid specialized review.

## 7. Protected ownership categories

XPScerpto uses the following ownership categories.

| Category | Responsibility |
|---|---|
| Sovereign maintainers | repository-wide integrity and final escalation |
| Governance maintainers | policy, claim discipline, review process, contributor rules |
| Security maintainers | security posture, vulnerability handling, cryptographic risk, authority risk |
| Architecture maintainers | system structure, module boundaries, public surface ownership |
| Platform maintainers | platform root, provider admission, platform API |
| Hardware maintainers | hardware evidence, topology, capability truth, routing inputs |
| SIMD maintainers | authority-routed SIMD execution, correctness, performance scope |
| FHE maintainers | FHE, BFV, CKKS, bootstrap, keys, encoders, evaluators, surface guards |
| Crypto maintainers | cryptographic primitives and domain-wide crypto review |
| PQC maintainers | post-quantum crypto surfaces and parameter review |
| Base maintainers | foundational types, checked primitives, root support |
| Memory maintainers | memory safety, spans, buffers, lifetime-sensitive utilities |
| Concurrency maintainers | atomics, synchronization, races, thread-sensitive behavior |
| CI maintainers | build graph, CI workflows, CTest integration, automation |
| Validation maintainers | tests, sanitizer evidence, final flags, validation gates |
| Audit maintainers | evidence, ledgers, reports, manifests, artifact integrity |
| Tools maintainers | repository tooling, scans, replay tools, report generators |
| Docs maintainers | documentation quality and consistency |
| Release maintainers | release artifacts, versioning, manifests, public release claims |

A real repository may combine roles when the team is small. The responsibility still exists even if one person temporarily fills multiple roles.

## 8. Root governance and policy ownership

The following documents require governance and security-aware review:

```text
START_HERE.md
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
CODEOWNERS
```

Recommended owners:

```text
governance maintainers
security maintainers
architecture maintainers, where architecture is affected
validation maintainers, where validation is affected
sovereign maintainers
```

These files define how the repository is reviewed and understood. They must not be changed casually.

## 9. Platform ownership

Platform-owned surfaces include:

```text
include/xps/crypto/platform/
include/xps/crypto/platform.api/
```

Recommended owners:

```text
platform maintainers
security maintainers
architecture maintainers
sovereign maintainers for high-risk changes
```

Platform ownership is authority-sensitive.

Platform changes must be reviewed for:

- provider admission;
- internal versus public platform surfaces;
- `platform.api` boundaries;
- platform runtime state;
- unsupported provider behavior;
- foreign-provider behavior;
- authority flow;
- fail-closed behavior;
- documentation and release claims.

Domain code must not bypass platform ownership by importing platform internals directly.

## 10. Hardware authority ownership

Hardware-owned surfaces include:

```text
include/xps/crypto/hw/
```

Recommended owners:

```text
hardware maintainers
platform maintainers
security maintainers
architecture maintainers
```

Hardware ownership protects evidence-backed hardware truth.

Hardware changes must be reviewed for:

- capability evidence;
- topology evidence;
- runtime availability;
- routing input;
- fallback behavior;
- domain-local probing risk;
- dispatch eligibility;
- auditability.

Hardware owners must reject changes that allow domains to manufacture hardware authority.

## 11. SIMD ownership

SIMD-owned surfaces include:

```text
include/xps/crypto/simd/
tests/simd/
```

Recommended owners:

```text
SIMD maintainers
hardware maintainers
security maintainers
validation maintainers
```

SIMD ownership must preserve authority-routed execution.

SIMD changes must be reviewed for:

- correctness;
- bounds behavior;
- overlap behavior;
- routing authority;
- scalar fallback behavior;
- dispatch eligibility;
- benchmark scope;
- hardware authority alignment;
- unauthorized fast paths.

A performance improvement that bypasses authority is not acceptable.

## 12. FHE ownership

FHE-owned surfaces include:

```text
include/xps/crypto/FHE/
tests/fhe/
```

Recommended owners:

```text
FHE maintainers
crypto maintainers
security maintainers
validation maintainers
architecture maintainers for public surfaces
sovereign maintainers for canonical or bootstrap-sensitive surfaces
```

FHE ownership is high risk because FHE correctness depends on parameter validity, context binding, key lifecycle, arithmetic semantics, encrypted-path behavior, scale and level invariants, and public API ownership.

FHE changes must be reviewed for:

- BFV behavior;
- CKKS behavior;
- bootstrap behavior;
- parameter and context binding;
- key lifecycle;
- key switching;
- relinearization;
- Galois and rotation behavior;
- NTT and RNS arithmetic;
- scale, level, modulus, and error behavior;
- public API ownership;
- surface guards;
- test-only versus production behavior;
- claim scope.

FHE progress must not be presented as repository-wide production readiness.

## 13. General cryptographic ownership

Cryptographic domains may include:

```text
include/xps/crypto/hash/
include/xps/crypto/kdf/
include/xps/crypto/mac/
include/xps/crypto/aead/
include/xps/crypto/aes/
include/xps/crypto/pqc/
include/xps/crypto/ed25519/
include/xps/crypto/x25519/
include/xps/crypto/rsa/
include/xps/crypto/VeloSeal64/
```

Recommended owners:

```text
crypto maintainers
security maintainers
domain-specific maintainers where applicable
validation maintainers for tests
```

Cryptographic ownership must preserve:

- parameter validation;
- key and seed lifecycle;
- randomness discipline;
- domain separation;
- misuse resistance where applicable;
- known-answer tests where appropriate;
- negative tests;
- claim scope.

A cryptographic primitive may be small, but ownership remains security-sensitive.

## 14. Base, memory, number, atomic, and integrity ownership

Support surfaces may include:

```text
include/xps/crypto/base/
include/xps/crypto/memory/
include/xps/crypto/number/
include/xps/crypto/atomic/
include/xps/crypto/integrity/
```

Recommended owners:

```text
base maintainers
memory maintainers
number maintainers
concurrency maintainers
integrity maintainers
security maintainers
architecture maintainers where boundaries are affected
```

These surfaces must not become hidden authority owners.

Changes must be reviewed for:

- checked arithmetic;
- memory safety;
- lifetime behavior;
- alignment assumptions;
- concurrency behavior;
- hidden runtime state;
- boundary leakage;
- domain-specific policy creeping into support code.

## 15. Build, CI, and validation ownership

Build and validation surfaces include:

```text
CMakeLists.txt
cmake/
.github/workflows/
tests/
tools/
scripts/
CI_VALIDATION_GATES.md
```

Recommended owners:

```text
build maintainers
CI maintainers
validation maintainers
security maintainers
architecture maintainers when graph structure changes
```

Build and CI ownership protects validation truth.

Changes must be reviewed for:

- target materialization;
- CTest labels;
- sanitizer configuration;
- sanitizer runtime claims;
- module graph behavior;
- import scans;
- generated files;
- evidence creation;
- final flags;
- skipped or disabled tests;
- claim boundaries.

CI changes that make overclaim easier must be rejected.

## 16. Test ownership

Test surfaces include:

```text
tests/
```

Recommended owners:

```text
validation maintainers
domain maintainers for the affected subsystem
security maintainers for security-sensitive tests
```

Tests are protected validation surfaces.

Test ownership must prevent:

- deleted assertions without replacement;
- weakened tests;
- hidden skips;
- label drift;
- test-only behavior becoming production authority;
- target-only success becoming full graph truth;
- security tests being converted into diagnostics.

A test change must preserve the truth the original test protected unless governance explicitly accepts the change.

## 17. Tools ownership

Tooling surfaces include:

```text
tools/
scripts/
```

Recommended owners:

```text
tools maintainers
validation maintainers
CI maintainers
security maintainers where evidence or protected surfaces are affected
```

Tools may inspect, scan, report, replay, or generate evidence. Tools do not automatically establish acceptance.

Tool ownership must preserve:

- repeatability;
- evidence integrity;
- exact blocker reporting;
- no hidden rewrites of protected surfaces;
- no metadata-only acceptance;
- no generated overclaims.

## 18. Documentation ownership

Documentation surfaces include:

```text
docs/
*.md
```

Recommended owners:

```text
docs maintainers
governance maintainers
security maintainers when security claims are affected
architecture maintainers when architecture claims are affected
validation maintainers when CI or evidence claims are affected
```

Documentation ownership must preserve claim discipline.

Documentation changes must not:

- present planned behavior as accepted;
- present implemented behavior as validated;
- present subsystem success as repository-wide success;
- claim sanitizer clean without runtime logs;
- claim production readiness without the relevant gate;
- hide blockers behind aspirational language.

Documentation is a governance surface when it changes claims.

## 19. Release and evidence ownership

Release and evidence surfaces may include:

```text
CHANGELOG.md
RELEASE_NOTES.md
VERSION
final_flags.txt
sha256_manifest.txt
patch.diff
evidence_bundle.tar.gz
source_tree_result.tar.gz
evidence/
reports/
```

Recommended owners:

```text
release maintainers
audit maintainers
validation maintainers
security maintainers
governance maintainers for claim-sensitive releases
```

Release ownership must preserve:

- source tree identity;
- artifact integrity;
- evidence bundle integrity;
- final flags;
- release notes claim scope;
- known blockers;
- sanitizer truth;
- full graph truth;
- production-readiness boundaries.

A release artifact must not claim more than its evidence proves.

## 20. Public API ownership

Every public API surface must have one clear canonical owner.

Public API ownership must identify:

- the public surface;
- its owner;
- allowed consumers;
- internal implementation boundaries;
- tests that protect it;
- evidence required for changes;
- retired or demoted legacy surfaces, if any.

Forbidden ownership patterns include:

- duplicate canonical owners;
- alias masking;
- forwarding files that hide ownership;
- compatibility wrappers that bypass governance;
- public APIs created only to make a failing import compile;
- retired surfaces reintroduced without review.

A public API is a contract.

## 21. Ownership escalation

Escalation is required when a change affects multiple high-risk areas.

Escalate to sovereign maintainers when a change affects:

- production-readiness claims;
- final flags;
- public release claims;
- canonical public surface ownership;
- authority-chain structure;
- security posture;
- FHE bootstrap completion;
- CI gates that define acceptance;
- unresolved high-risk blockers;
- cross-domain architectural changes.

Escalation does not guarantee approval. It ensures the right level of review.

## 22. Ownership conflicts

An ownership conflict occurs when:

- two surfaces claim the same canonical role;
- a public API has unclear ownership;
- a team approves a change outside its expertise;
- a test-only surface becomes production behavior;
- a domain imports another domain’s internals;
- a CI rule changes acceptance meaning without validation ownership;
- documentation claims a status that validation owners did not prove.

Ownership conflicts must be resolved before acceptance.

Do not hide ownership conflicts with aliases, shims, adapters, or broad imports.

## 23. Small team rule

If XPScerpto is maintained by a small team or a single maintainer, the ownership model still applies.

In that case, the maintainer should record the ownership role being fulfilled during review.

For example:

```text
Reviewed as: security owner, validation owner, and governance owner.
```

This preserves review discipline even when roles are temporarily held by the same person.

Small team size does not remove the responsibility to review authority, boundaries, security, validation, and claims.

## 24. Relationship to CODEOWNERS

`OWNERSHIP_MODEL.md` is the platform-neutral ownership policy.

`CODEOWNERS` is the GitHub implementation of this policy.

`CODEOWNERS` should:

- route reviews to the correct owners;
- use real GitHub users or teams;
- require owners for protected surfaces;
- keep broad rules before specific rules;
- align with this ownership model;
- avoid becoming the only description of ownership.

If `CODEOWNERS` and this document disagree, the policy should be corrected so both align. Do not rely on a platform-routing file to override the ownership model.

## 25. Relationship to review policy

`REVIEW_POLICY.md` defines how owners and reviewers should evaluate changes.

This document defines who should own and review surfaces.

Ownership without review discipline is not enough.

Review discipline without correct ownership is not enough.

Both must be satisfied.

## 26. Current claim boundaries

This ownership model does not elevate project status.

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

Ownership policy is not validation evidence. It defines who must guard the surfaces that produce evidence and claims.

## 27. Final ownership rule

The final ownership rule is:

```text
every protected surface needs an owner
every owner must preserve evidence
every review must preserve boundaries
every claim must remain within validation
```

XPScerpto remains trustworthy only when ownership, review, validation, authority, and claims remain aligned.
