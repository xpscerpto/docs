# XPScerpto Governance

**Status:** normative governance charter  
**Scope:** repository-wide decision authority, claim discipline, protected surfaces, review requirements, evidence requirements, and merge policy

XPScerpto is a governed sovereign cryptographic infrastructure platform. This document defines how decisions are made, how claims are controlled, how protected surfaces are reviewed, and what evidence is required before a change may be accepted.

This document is not a validation report, security proof, production-readiness claim, or acceptance artifact. It defines the governance rules that must be satisfied by such artifacts.

## 1. Purpose

The purpose of this governance charter is to protect the integrity of XPScerpto as a sovereign cryptographic infrastructure system.

XPScerpto must not be governed as a loose collection of independent utilities. Changes must preserve the project’s authority model, module boundaries, fail-closed behavior, evidence discipline, and claim boundaries.

Governance applies to source code, tests, build logic, CI configuration, documentation, release notes, evidence bundles, and public claims.

## 2. Governance principle

The central governance rule is:

```text
authority before execution
evidence before acceptance
failure before false success
```

A change is not acceptable merely because it builds, passes a narrow test, or appears to simplify the code. A change is acceptable only if it preserves authority, maintains evidence boundaries, respects protected surfaces, and does not overclaim the project’s state.

A shortcut that makes the build pass while weakening authority, ownership, fail-closed behavior, sanitizer truth, module boundaries, or claim discipline is a governance violation.

## 3. Authority model preservation

XPScerpto relies on explicit authority flow. The expected authority chain is:

```text
platform governance
→ platform.api
→ hardware authority
→ routing capsule
→ domain execution
```

Domain code must not invent its own authority by probing the operating system, CPU, compiler environment, hidden runtime state, or local capability shortcuts.

Authority-sensitive code must receive its execution authority through the governed platform and hardware evidence chain. Local probing, implicit admission, hidden fallbacks, or unreviewed runtime shortcuts are not acceptable substitutes for governed authority.

## 4. Claim discipline

Claims must be conservative, evidence-backed, and traceable to a specific gate.

The following claims, and any equivalent or stronger claims, must not be made unless the relevant acceptance gate proves them through live evidence:

- production-ready
- production-grade
- deployment-ready
- bootstrap complete
- full graph passed
- sanitizer clean
- security hardened
- fully validated
- release accepted
- FHE accepted
- CKKS accepted
- BFV accepted
- full runtime closure
- full materialization closure

A claim is not established by intention, partial execution, historical logs, configure-only success, target-only success, or a passing result that hides an unresolved blocker.

Every significant claim must identify:

- the gate or phase that established it
- the source tree used
- the tests or CI jobs executed
- the sanitizer runtime status, if relevant
- the final flags or equivalent acceptance record
- the exact blocker if the claim is incomplete

## 5. Global claim boundaries

Global production and sanitizer claims are conservative by default.

Unless a validated acceptance gate proves otherwise, the following boundaries remain in force:

- `FHE_PRODUCTION_READY = NO`
- `FHE_BOOTSTRAP_COMPLETE = NO`
- `FULL_SANITIZER_CTEST_CLAIMED = NO`

These flags must not be elevated by documentation, local edits, isolated tests, metadata-only reports, or unreviewed assertions.

They may only be changed by reviewed evidence from the relevant acceptance gates, final flags, CI results, sanitizer runtime logs, and merge records.

## 6. Protected surfaces

Protected surfaces require strict review and evidence.

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
.github/
```

Changes to protected surfaces must explain their authority impact, boundary impact, test impact, and claim impact.

Protected surfaces must not be changed through aliases, shims, adapters, weakened tests, deleted assertions, fake fallbacks, metadata-only proofs, or production overclaims.

## 7. Review authority

Review authority must match the risk of the change.

Security-sensitive, authority-sensitive, build-system, cryptographic, hardware-routing, SIMD-dispatch, FHE, and governance changes require review by the appropriate maintainers or owners.

Reviewers must evaluate more than style and local correctness. They must evaluate:

- whether authority boundaries are preserved
- whether module boundaries are preserved
- whether protected surfaces were changed
- whether tests were added, weakened, deleted, or bypassed
- whether evidence is live, relevant, and sufficient
- whether the change introduces hidden runtime behavior
- whether the claim made by the change is justified
- whether incomplete work fails closed with an exact blocker

A reviewer must reject a change that passes tests but weakens governance.

## 8. Evidence requirements

Every meaningful change must include enough evidence to justify acceptance.

Evidence may include:

- build logs
- CTest logs
- targeted test logs
- sanitizer runtime logs
- CI logs
- final flags
- evidence bundles
- failure reports
- blocker reports
- source tree identifiers
- patch or diff records
- reviewed merge records

Evidence must be live and relevant to the source tree under review. Packaged or historical evidence may be used as provenance, but it must not replace live validation when live validation is required.

Configure-only success is not build success.

Build success is not test success.

Sanitizer configuration is not sanitizer truth.

A partial test pass is not full graph acceptance.

## 9. Merge requirements

A meaningful change must explain:

- affected surfaces
- authority impact
- module-boundary impact
- security impact
- build impact
- test evidence
- sanitizer status
- claim boundaries
- production flag status
- exact blocker if incomplete

A merge must not rely on ambiguity. If the change is incomplete, the result must fail closed and identify the exact blocker.

A merge must not silently delete tests, weaken assertions, rename failures into success, bypass ownership, or hide unresolved blockers.

## 10. Fail-closed policy

Incomplete work must fail closed.

Acceptable incomplete results include clear outcomes such as:

```text
NO_WITH_REASON
PATH_B_FAIL_CLOSED
EXACT_BLOCKER=...
```

Unacceptable incomplete results include vague success language, hidden blockers, partial execution presented as full closure, or claims that depend on tests that did not run.

Fail-closed behavior is a governance requirement, not a stylistic preference.

## 11. Test and sanitizer truth

Tests must measure the claim being made.

A test must not be weakened to make a phase pass. Assertions must not be deleted, bypassed, or converted into non-binding diagnostics to create acceptance.

Sanitizer truth requires runtime execution under the relevant sanitizer configuration. A sanitizer build, sanitizer configure step, or sanitizer target generation is not sufficient.

If sanitizer execution is blocked, the blocker must be reported exactly. The result must not be described as sanitizer clean.

## 12. Production-readiness policy

Production readiness is a gated state, not a statement of ambition.

The project, or any protected subsystem, must not be described as production-ready unless all required validation gates pass and the claim is supported by reviewed evidence.

The following are not sufficient for production readiness:

- local success on a single machine
- a single passing unit test
- a passing smoke test
- configure-only success
- build-only success
- historical evidence without live replay when live replay is required
- sanitizer configuration without sanitizer runtime
- documentation updates without validation
- partial subsystem closure presented as global closure

Production readiness may only be claimed by an explicit gate that defines scope, evidence, final flags, and acceptance criteria.

## 13. Security-sensitive changes

Security-sensitive changes require additional discipline.

A security-sensitive change includes any change that affects:

- cryptographic behavior
- key lifecycle
- randomness
- parameter validation
- authority admission
- platform or hardware evidence
- module boundaries
- protected public surfaces
- sanitizer or test enforcement
- CI gates
- release claims
- failure behavior

Security-sensitive changes must not be rushed through convenience fixes. They must include a clear threat model impact, negative test coverage where applicable, and evidence that the change does not weaken existing guarantees.

## 14. Forbidden shortcuts

The following shortcuts are not acceptable:

- deleting tests to create a pass
- weakening assertions to create a pass
- replacing runtime validation with metadata-only proof
- presenting configure success as build success
- presenting build success as test success
- presenting sanitizer configuration as sanitizer clean
- using aliases to hide ownership conflicts
- using shims or adapters to bypass protected surfaces
- using local probes to bypass governed authority
- claiming full graph truth from target-only execution
- claiming production readiness from partial subsystem closure
- hiding an exact blocker behind vague wording
- reintroducing retired public surfaces as compatibility shortcuts without governance approval

If a shortcut is necessary for temporary investigation, it must be isolated, named as non-acceptance work, and prevented from becoming a production or acceptance claim.

## 15. RFC process and decision records

Long-lived architectural, authority, ownership, security, validation, compatibility, or governance decisions must follow an explicit governance process.

Major proposals should use the RFC process defined in:

```text
docs/governance/rfc-process.md
```

Accepted, rejected, superseded, or long-term decisions should be recorded using:

```text
docs/governance/decision-record-template.md
```

The governance process index is maintained at:

```text
docs/governance/README.md
```

An RFC is appropriate when a proposed change affects:

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

A decision record should explain:

- the decision;
- the context;
- the alternatives rejected;
- the authority impact;
- the security impact;
- the validation plan;
- the migration impact;
- the required owner approvals;
- the evidence status;
- the rollback or supersession criteria;
- the claims that are not being made.

RFC acceptance authorizes implementation work within the accepted scope. It does not merge code, prove implementation, establish sanitizer truth, prove full graph acceptance, or create production readiness.

Decision records must not be used to justify an unreviewed bypass. They document governance; they do not replace validation.

## 16. Relationship to CODEOWNERS

`CODEOWNERS` defines review routing and ownership enforcement.

This governance charter defines the rules that owners must apply.

A CODEOWNERS approval is not automatically sufficient if the change violates governance. Likewise, a passing test result is not sufficient if the required ownership review was bypassed.

Ownership and evidence must both be satisfied.

## 17. Documentation discipline

Documentation must not overclaim the state of the project.

Documentation may explain goals, architecture, design intent, and validation strategy. It must clearly distinguish between:

- implemented behavior
- validated behavior
- planned behavior
- rejected behavior
- blocked behavior
- production-ready behavior

If a document describes a future or intended state, it must not present that state as already accepted.

Documentation that changes claim boundaries must be reviewed with the same seriousness as code that changes protected behavior.

## 18. Final governance rule

XPScerpto must remain aligned across four dimensions:

```text
correctness
authority
evidence
claim discipline
```

A change that improves one dimension while weakening another is not accepted.

The project advances only when the implementation, tests, evidence, and claims all describe the same truth.
