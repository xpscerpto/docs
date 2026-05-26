# Contributing to XPScerpto

**Status:** normative contribution guide  
**Scope:** repository-wide contribution expectations, pull request preparation, protected-surface awareness, security-sensitive changes, testing expectations, evidence requirements, documentation discipline, and contributor conduct

XPScerpto is a governed sovereign cryptographic infrastructure platform. Contributions are welcome, but they must respect the project’s authority model, module boundaries, evidence discipline, validation gates, and claim boundaries.

This document explains how to prepare a contribution so that it can be reviewed responsibly.

This document is not a validation report, security proof, production-readiness claim, or acceptance artifact. It describes the contributor workflow and the minimum discipline expected before review.

## 1. Purpose

The purpose of this guide is to help contributors submit changes that are understandable, reviewable, testable, and aligned with XPScerpto’s governance model.

A contribution is not ready merely because the code compiles.

A contribution is not ready merely because one local test passes.

A contribution is ready for review when its scope, evidence, tests, affected boundaries, and claim limits are clear.

## 2. Read before contributing

Before contributing, read the repository documents in this order:

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
SUPPORT.md
LICENSE_STATUS.md
```

You do not need to memorize every document before making a small change, but you are responsible for understanding the documents that apply to the area you are changing.

If your change touches protected surfaces, security-sensitive behavior, cryptographic behavior, authority routing, build logic, CI, tests, tools, or documentation claims, read the relevant policy documents before opening a pull request.

## 3. Contribution principle

The central contribution principle is:

```text
clear scope
clear evidence
clear boundaries
clear claims
```

Every meaningful contribution should explain:

- what changed;
- why it changed;
- which files or surfaces were affected;
- whether protected surfaces were touched;
- what tests or checks were run;
- what evidence supports the change;
- what claims are being made;
- what claims are not being made;
- what blockers remain, if any.

A contribution that hides uncertainty is not ready.

A contribution that reports an exact blocker may still be valuable.

## 4. Acceptable contribution types

XPScerpto accepts many types of contributions when they follow project discipline.

Examples include:

- bug fixes;
- security fixes;
- test improvements;
- negative test coverage;
- documentation clarification;
- CI improvements;
- build-system repairs;
- module-boundary enforcement;
- authority-routing validation;
- cryptographic correctness fixes;
- sanitizer issue isolation;
- performance improvements that preserve authority;
- evidence and tooling improvements;
- release-process hardening.

A contribution does not need to be large to be valuable. A precise fix with clear evidence is better than a broad patch with unclear claims.

## 5. Before opening a pull request

Before opening a pull request, make sure you can answer:

- What problem does this solve?
- What is the smallest accurate scope of the change?
- Which files changed?
- Which protected surfaces are affected?
- Does the change affect authority flow?
- Does the change affect module boundaries?
- Does the change affect cryptographic behavior?
- Does the change affect tests, CI, sanitizer configuration, tools, or documentation claims?
- What commands did you run?
- What passed?
- What failed?
- What did not run?
- What remains blocked?
- What claim is supported by the evidence?

If you cannot answer these questions, narrow the change or add evidence before requesting review.

## 6. Protected surfaces

Protected surfaces require special care.

Protected surfaces include, but are not limited to:

```text
include/xps/crypto/base/
include/xps/crypto/platform/
include/xps/crypto/platform.api/
include/xps/crypto/hw/
include/xps/crypto/simd/
include/xps/crypto/FHE/
include/xps/crypto/hash/
include/xps/crypto/kdf/
include/xps/crypto/mac/
include/xps/crypto/aead/
include/xps/crypto/aes/
include/xps/crypto/pqc/
include/xps/crypto/ed25519/
include/xps/crypto/x25519/
include/xps/crypto/rsa/
include/xps/crypto/number/
include/xps/crypto/atomic/
include/xps/crypto/memory/
include/xps/crypto/integrity/
include/xps/crypto/audit/
CMakeLists.txt
tests/
tools/
docs/
.github/
```

If your contribution touches a protected surface, your pull request must explain:

- why the surface was changed;
- what authority or boundary is affected;
- what tests protect the change;
- what evidence supports the change;
- what claims remain out of scope.

Do not treat protected surfaces as ordinary implementation files.

## 7. Security-sensitive contributions

A contribution is security-sensitive if it affects:

- cryptographic behavior;
- keys, seeds, randomness, credentials, or secret material;
- parameter validation;
- authority admission;
- platform or provider behavior;
- hardware evidence;
- routing or dispatch;
- SIMD eligibility;
- memory safety;
- concurrency behavior;
- public API ownership;
- protected surfaces;
- tests that enforce security behavior;
- CI gates;
- sanitizer configuration or execution;
- evidence generation;
- final flags;
- release notes or security claims.

Security-sensitive contributions must include appropriate evidence.

If the change fixes a vulnerability, do not disclose exploit details publicly before triage. Follow `SECURITY.md`.

If the issue is sensitive and no private channel is available, request a private reporting channel without including exploit details.

## 8. Authority-sensitive contributions

XPScerpto uses an explicit authority chain:

```text
platform governance
→ platform.api
→ hardware authority
→ routing capsule
→ domain execution
→ audit and evidence
```

If your change affects execution decisions, platform admission, hardware evidence, routing, dispatch, or domain execution, explain how the authority chain remains preserved.

Do not introduce:

- domain-local hardware authority;
- direct platform internal access from domains;
- CPU probing inside domains as authority;
- compiler macro inference as runtime permission;
- hidden runtime globals as authorization;
- fallback behavior presented as admitted execution.

Capability is not authority.

## 9. Module-boundary contributions

If your change affects imports, includes, public surfaces, internal surfaces, aliases, shims, adapters, or ownership, explain the boundary impact.

Do not introduce:

- broad imports to make a failing target compile;
- aliases that hide ownership conflicts;
- shims that bypass protected surfaces;
- adapters that replace proper public surfaces;
- test-only helpers imported by production code;
- retired public surfaces reintroduced without review;
- duplicate canonical public API owners.

If a boundary is wrong, repair the boundary. Do not hide it.

## 10. Cryptographic contributions

Cryptographic contributions require conservative scope.

If your change affects a cryptographic domain, explain:

- what security or correctness property changed;
- what parameters, keys, contexts, states, or inputs are affected;
- what invalid behavior is rejected;
- what tests prove the corrected behavior;
- what negative tests were added;
- what claims remain out of scope.

Cryptographic domains may include:

```text
hash
KDF
MAC
AEAD
AES
PQC
Ed25519
X25519
RSA
FHE
VeloSeal64
```

Do not present metadata-only checks as runtime validation.

Do not present test-vector success as full hardening.

Do not present one subsystem’s acceptance as repository-wide acceptance.

## 11. FHE and advanced cryptographic contributions

FHE is one high-risk subsystem among several protected domains.

If your contribution touches FHE, explain whether it affects:

- BFV;
- CKKS;
- bootstrap;
- encoding or decoding;
- encryption or decryption;
- parameter or context binding;
- key lifecycle;
- key switching;
- relinearization;
- Galois or rotation behavior;
- NTT or RNS arithmetic;
- scale, level, modulus, or error behavior;
- public API ownership;
- surface guards;
- test-only versus production behavior.

Keep claims scoped.

BFV acceptance does not imply CKKS acceptance.

CKKS acceptance does not imply bootstrap completion.

Bootstrap progress does not imply production readiness.

FHE progress does not imply repository-wide production readiness.

## 12. Testing expectations

Run the smallest meaningful test set that proves your change, and run broader tests when your change affects broader surfaces.

Your pull request should state:

- the exact commands run;
- the build configuration;
- the test scope;
- the number of tests run;
- the number of failures;
- any skipped or disabled tests;
- the first failure, if any;
- exact blockers, if any.

Examples of useful test evidence include:

```text
cmake configure output
build output
ctest --output-on-failure output
targeted test logs
labelled CTest logs
sanitizer runtime logs
import scan output
boundary scan output
final flags
```

If you did not run a relevant test, say so. Do not imply that it passed.

## 13. CI and validation expectations

CI validation must match the claim.

Use the distinctions defined in `CI_VALIDATION_GATES.md`:

```text
configure success ≠ build success
build success ≠ test success
test success ≠ sanitizer truth
target success ≠ full graph truth
subsystem success ≠ repository-wide acceptance
```

Do not claim:

- full graph passed from a target-only run;
- sanitizer clean from sanitizer configuration;
- tests passed when only configure or build ran;
- production readiness from local testing;
- subsystem acceptance from unrelated tests.

A precise partial result is acceptable. An inflated claim is not.

## 14. Sanitizer expectations

If your contribution affects memory safety, undefined behavior, concurrency, low-level primitives, cryptographic buffers, or protected runtime behavior, sanitizer evidence may be required.

Sanitizer truth requires runtime execution.

A sanitizer build is not sanitizer clean.

A sanitizer configuration is not sanitizer clean.

If sanitizer execution fails, preserve the exact blocker.

Do not hide sanitizer crashes, timeouts, missing executables, or incomplete target materialization.

## 15. Documentation contributions

Documentation changes are welcome, but they must preserve claim discipline.

Documentation must distinguish:

- planned behavior;
- implemented behavior;
- validated behavior;
- blocked behavior;
- accepted behavior;
- production-ready behavior.

Do not write documentation that claims:

- production-ready;
- sanitizer clean;
- full graph passed;
- bootstrap complete;
- subsystem accepted;
- security hardened;
- release accepted;

unless the relevant validation gate proves that exact claim.

Documentation that changes project status, security posture, authority behavior, boundaries, CI claims, or release claims is review-sensitive.

## 16. Evidence in pull requests

A pull request should include an evidence section.

A strong evidence section includes:

```text
Commands run:
- ...

Results:
- ...

Scope:
- ...

Not run:
- ...

Known blockers:
- ...

Claims made:
- ...

Claims not made:
- ...
```

Evidence should be honest and scoped.

If only a focused test was run, say it was focused.

If full graph validation was not run, say it was not run.

If sanitizer runtime was not run, do not claim sanitizer clean.

If a blocker remains, identify it exactly.

## 17. Pull request description template

Use the following structure when appropriate:

```markdown
## Summary

Describe the change in one or two paragraphs.

## Scope

List the files, modules, or surfaces affected.

## Authority impact

Explain whether the authority chain is affected.

## Boundary impact

Explain whether module boundaries, public surfaces, or protected surfaces changed.

## Security impact

Explain whether the change is security-sensitive.

## Tests and evidence

List exact commands and results.

## Claims made

State exactly what this change proves.

## Claims not made

State what this change does not prove.

## Known blockers

List exact blockers, or write `none known` if none are known.
```

Do not use the template to hide uncertainty. Use it to make uncertainty visible.

## 18. Commit discipline

Commits should be understandable and scoped.

Prefer commits that:

- describe the reason for the change;
- keep unrelated changes separate;
- avoid mixing code, tests, CI, and documentation unless necessary;
- do not hide test weakening inside unrelated edits;
- do not rename failures into success;
- do not alter final flags without evidence.

Avoid commit messages that overclaim.

A commit message should not say `production ready`, `fully secure`, `sanitizer clean`, or `complete` unless the relevant gate proves the claim.

## 19. What not to do

Do not:

- delete failing tests to create a pass;
- weaken assertions to create acceptance;
- convert failures into diagnostics without review;
- bypass `platform.api`;
- import platform internals from domains for convenience;
- introduce local hardware authority inside domains;
- add aliases or shims to hide ownership conflicts;
- use test-only helpers in production code;
- present metadata-only proof as runtime validation;
- present configure success as build success;
- present build success as test success;
- present sanitizer configuration as sanitizer clean;
- present target-only success as full graph truth;
- present subsystem success as repository-wide acceptance;
- change documentation to claim more than evidence proves;
- disclose exploit details publicly before security triage.

These patterns may cause a pull request to be rejected even if the code compiles.

## 20. Handling incomplete work

Incomplete work may be useful if it is honest.

If a contribution isolates a blocker, reports a failing test, adds diagnostics, or documents a gap, it may still be valuable.

Use precise language:

```text
NO_WITH_REASON
PATH_B_FAIL_CLOSED
BLOCKED
NOT_CLAIMED
EXACT_BLOCKER=...
```

Do not present incomplete work as acceptance.

Do not hide the blocker.

Do not turn a blocker into a vague statement.

An exact failure is better than a false success.

## 21. Generated files and tooling output

Generated files, tool-produced reports, and automated changes must still be reviewed.

If your contribution includes generated output, explain:

- what generated it;
- whether it is deterministic;
- what source inputs were used;
- whether protected surfaces changed;
- whether tests or evidence changed;
- whether generated content affects claims.

Do not use generated output to bypass review.

## 22. Performance contributions

Performance contributions are welcome when they preserve authority and correctness.

A performance contribution should include:

- correctness evidence;
- benchmark scope;
- hardware authority status, if relevant;
- dispatch eligibility, if relevant;
- compared paths;
- configuration;
- limitations.

Do not claim performance dominance from a narrow benchmark.

Do not bypass authority for speed.

A faster path outside the governed execution model is not acceptable.

## 23. Release-related contributions

Release-related contributions require claim discipline.

If your change affects release notes, versioning, packaging, artifacts, final flags, evidence bundles, or public status, explain:

- what claim is being made;
- what evidence supports the claim;
- what remains out of scope;
- whether final flags changed;
- whether any blockers remain.

A release-related contribution must not inflate status.

## 24. Contributor conduct

Contributors are expected to be respectful, precise, and evidence-oriented.

Good contribution behavior includes:

- stating uncertainty honestly;
- preserving blockers;
- asking for review when unsure;
- keeping claims scoped;
- responding to review with evidence;
- accepting that protected surfaces require stricter review.

Do not pressure reviewers to accept changes without evidence.

Do not treat requested changes as hostility. In XPScerpto, strict review protects the project.

## 25. Large changes and RFCs

Some changes are too important to start as ordinary implementation pull requests.

A proposed change may require an RFC before implementation if it affects:

- architecture;
- authority flow;
- public API ownership;
- platform or provider behavior;
- hardware evidence;
- runtime routing;
- cryptographic subsystem architecture;
- dependency policy;
- build graph materialization;
- CI validation gates;
- sanitizer policy;
- security disclosure policy;
- release claim boundaries.

Use:

```text
docs/governance/rfc-process.md
```

for major proposals.

Use:

```text
docs/governance/decision-record-template.md
```

for accepted, rejected, superseded, or long-term decisions.

If you are unsure whether a change requires an RFC, ask before implementation. Opening an RFC is better than hiding a long-term decision inside a patch.

## 26. Relationship to review policy

`CONTRIBUTING.md` explains how contributors prepare changes.

`REVIEW_POLICY.md` explains how reviewers evaluate changes.

A contribution may follow this guide and still receive requested changes if evidence is insufficient, scope is unclear, or claim boundaries are not preserved.

The goal is not to make every contribution pass immediately. The goal is to make every contribution reviewable and honest.

## 27. Relationship to security policy

If your contribution involves a vulnerability or security-sensitive behavior, follow `SECURITY.md`.

Do not disclose exploit details publicly before triage.

Do not place sensitive proof-of-concept details in a public issue or public pull request unless maintainers have coordinated disclosure.

Security fixes must preserve authority, tests, and evidence. A security fix that weakens validation is not acceptable.

## 28. Current claim boundaries

This contribution guide does not elevate project status.

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

A contribution guide is not validation evidence. It defines how contributors should prepare changes for review.

## 29. Final contribution rule

The final contribution rule is:

```text
make the change clear
make the evidence clear
make the boundaries clear
make the claims clear
```

A contribution is ready for review only when a reviewer can understand what changed, why it changed, what was tested, what was not tested, what is claimed, and what remains blocked.
