# XPScerpto Sovereign Engineering Principles

**Status:** normative engineering principles  
**Scope:** repository-wide engineering discipline, authority-aware implementation, validation behavior, review expectations, and documentation truth

XPScerpto is a governed sovereign cryptographic infrastructure platform. Its engineering discipline is built on explicit authority, controlled boundaries, fail-closed behavior, and evidence-backed validation.

This document defines how XPScerpto is expected to be engineered. It complements `GOVERNANCE.md`: governance defines the rules for acceptance, while this document defines the engineering principles that should shape every implementation, test, review, and architectural decision.

This document is not a validation report, security proof, production-readiness claim, or acceptance artifact. It defines the discipline required before such claims may be considered.

## 1. Engineering thesis

The central engineering thesis of XPScerpto is:

```text
correctness without authority is incomplete
authority without evidence is untrusted
evidence without fail-closed behavior is fragile
performance without governance is not acceptable
```

Engineering work in XPScerpto is not complete when code merely builds or a narrow test passes. Work is complete only when the implementation, authority path, tests, evidence, and claims describe the same truth.

## 2. Authority must be explicit

Authority must be passed, admitted, and verified explicitly.

Domain code must not invent authority by probing the operating system, CPU, compiler environment, hidden runtime state, build flags, or local capability shortcuts.

The expected authority flow is:

```text
platform governance
→ platform.api
→ hardware authority
→ routing capsule
→ domain execution
```

Any implementation that bypasses this flow must be treated as suspect until it is explicitly reviewed and justified.

Authority-sensitive code should make its dependencies visible. Hidden authority is a design defect.

## 3. Fail closed before failing open

XPScerpto must prefer clear refusal over ambiguous success.

If an operation cannot prove that required authority, parameters, keys, ownership, memory state, or runtime conditions are valid, it must fail closed.

A fail-closed result should identify the exact blocker whenever possible. Vague failure is better than false success, but exact failure is better than both.

Acceptable incomplete outcomes include:

```text
NO_WITH_REASON
PATH_B_FAIL_CLOSED
EXACT_BLOCKER=...
```

Unacceptable outcomes include partial execution presented as full closure, silent fallback, hidden downgrade, and success claims that depend on conditions that were not verified.

## 4. Evidence is part of engineering

Evidence is not paperwork added after engineering. Evidence is part of the engineering result.

A change should be designed so that its behavior can be tested, audited, and explained. If a claim cannot be supported by live evidence, the claim must not be made.

Evidence may include build logs, CTest logs, sanitizer runtime logs, targeted test output, final flags, failure reports, evidence bundles, and reviewed merge records.

Historical evidence may help establish provenance, but it must not replace live validation when live validation is required.

## 5. Tests are authority instruments

Tests in XPScerpto are not decorative.

A test should define a boundary, preserve an invariant, reject a shortcut, or expose a blocker. Tests should not merely confirm that the easiest path works.

High-value tests include:

- positive correctness tests
- negative security tests
- fail-closed tests
- boundary and ownership tests
- authority-routing tests
- sanitizer-backed runtime tests
- regression tests for previously closed blockers
- claim-discipline tests that reject overclaims

A weakened test is not a repair. A deleted assertion is not closure. A passing test that no longer measures the original claim is a governance failure.

## 6. Security is not a patch layer

Security must be designed into the authority model, parameter validation, memory ownership, key lifecycle, randomness behavior, module boundaries, and runtime admission path.

Security-sensitive code must not depend on undocumented assumptions, ambient state, accidental initialization, hidden fallbacks, or non-binding comments.

A security fix should explain what was unsafe, what changed, why the new behavior is safer, and how the change is tested.

Security claims must remain conservative unless proven through the relevant acceptance gate.

## 7. Boundaries are engineering assets

Module boundaries, ownership boundaries, platform boundaries, and authority boundaries are part of the system design.

They must not be treated as inconvenience barriers to bypass during implementation.

A boundary may be changed only when the change is intentional, reviewed, documented, tested, and consistent with governance.

The following patterns are unacceptable unless explicitly reviewed as non-acceptance investigative work:

- aliases that hide ownership conflicts
- shims that bypass protected surfaces
- adapters that replace authority with convenience
- local probes that bypass governed routing
- metadata-only proofs that replace runtime validation
- compatibility paths that reintroduce retired surfaces

A clean boundary is a long-term engineering asset.

## 8. Performance must remain governed

Performance is a core engineering goal, but it must not bypass authority.

A faster path is not acceptable if it depends on unauthorized hardware probing, compiler macro inference, hidden runtime state, unsafe memory assumptions, or unreviewed dispatch behavior.

Performance work must preserve:

- authority routing
- correctness
- memory safety
- module boundaries
- fail-closed behavior
- test evidence
- claim discipline

Optimization is accepted only when it remains inside the governed execution model.

## 9. Portability must be explicit

XPScerpto should make platform assumptions explicit.

Operating-system behavior, compiler behavior, pointer assumptions, hardware capability, module support, sanitizer behavior, and runtime availability must not be silently assumed where they affect correctness or authority.

If a platform cannot support a required behavior, the implementation should fail closed, provide a clear blocker, or route through an approved platform surface.

Portability must not be achieved by weakening security, authority, or validation requirements.

## 10. Simplicity is valuable only when it preserves sovereignty

Simple code is preferred when it preserves correctness, authority, evidence, and boundaries.

Simplicity is not an excuse for removing validation, collapsing ownership, bypassing routing, weakening tests, or hiding failure cases.

A simple implementation that makes authority implicit is not simpler in XPScerpto. It is less governed.

A simple implementation that makes evidence clearer is preferred.

## 11. Documentation must tell the same truth as the code

Documentation must not overclaim the implementation.

A document may describe goals, intended architecture, accepted behavior, rejected behavior, blockers, and future work. It must distinguish those states clearly.

Documentation must not present a planned state as an accepted state.

Documentation must not describe a subsystem as production-ready, complete, accepted, hardened, or sanitizer-clean unless the relevant acceptance evidence proves that claim.

The best documentation makes the review path clearer and the claim boundary harder to misunderstand.

## 12. Repair must preserve the reason the test exists

A repair is valid only if it preserves the purpose of the test, guard, or boundary that exposed the issue.

When a failure occurs, the first engineering question should not be:

```text
How do we make this pass?
```

It should be:

```text
What truth is this failure protecting?
```

If the failure exposes a stale API name, repair the API usage. If it exposes a missing authority path, repair the authority path. If it exposes a real security weakness, repair the security weakness.

Do not repair a failure by weakening the signal that found it.

## 13. Cryptographic engineering must remain conservative

Cryptographic code must be engineered with conservative assumptions.

Correctness, parameter validation, key lifecycle, randomness, encoding, decoding, arithmetic semantics, scale and level invariants, noise behavior, and negative security behavior must be validated explicitly.

A cryptographic path must not be accepted because it appears mathematically plausible. It must be tested against the claim being made.

Partial closure of one cryptographic subsystem must not be presented as closure of another subsystem or the entire platform.

For example, BFV acceptance does not imply CKKS acceptance. CKKS acceptance does not imply bootstrap completion. Bootstrap completion does not imply production readiness unless all required gates prove that broader claim.

## 14. Build success is not acceptance

A successful configure step, build step, or target materialization step is not the same as acceptance.

The following distinctions must remain clear:

```text
configure success ≠ build success
build success ≠ test success
test success ≠ sanitizer truth
target success ≠ full graph truth
subsystem acceptance ≠ platform production readiness
```

Engineering reports, documentation, commit messages, and release notes must preserve these distinctions.

## 15. Review must ask the hard questions

A review should ask:

- What authority does this change depend on?
- Where does that authority come from?
- What boundary does this change touch?
- What failure mode is now possible?
- What evidence proves the claim?
- What evidence is missing?
- Were tests added, weakened, deleted, bypassed, or renamed?
- Does the change preserve fail-closed behavior?
- Does the change introduce hidden runtime behavior?
- Does the documentation claim more than the evidence proves?

A review that checks only style and local correctness is not sufficient for protected surfaces.

## 16. Accepted engineering behavior

XPScerpto engineering should favor:

- explicit authority
- narrow public surfaces
- strict ownership
- clear failure
- live evidence
- negative tests
- deterministic validation where possible
- guarded runtime decisions
- conservative claims
- reviewed acceptance gates
- precise blockers
- documentation aligned with implementation

These behaviors are not optional preferences. They are part of the project’s engineering identity.

## 17. Rejected engineering behavior

XPScerpto rejects:

- silent fallback
- hidden downgrade
- authority by local probing
- security by comment
- correctness by assumption
- performance by bypass
- test weakening
- assertion deletion
- metadata-only acceptance
- configure-only overclaims
- build-only overclaims
- sanitizer-configure-only claims
- partial closure presented as global closure
- documentation that outruns evidence

A change that relies on these patterns should be rejected or isolated as non-acceptance investigative work.

## 18. Final principle

The final engineering principle is alignment.

```text
implementation
tests
evidence
documentation
claims
```

All five must describe the same state.

When they disagree, the project must trust the stricter truth, preserve the blocker, and continue engineering until the evidence and the claim align.
