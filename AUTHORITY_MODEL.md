# XPScerpto Authority Model

**Status:** normative authority model  
**Scope:** repository-wide execution authority, platform admission, hardware evidence, routing decisions, domain execution, audit expectations, and fail-closed authority behavior

XPScerpto is a governed sovereign cryptographic infrastructure platform. Its execution model is based on explicit authority, not ambient capability, local probing, hidden runtime state, or convenience shortcuts.

This document defines where execution authority comes from, how it is admitted, how it flows through the system, and what behavior is forbidden when authority has not been established.

This document is not a validation report, security proof, production-readiness claim, or acceptance artifact. It defines the authority model that implementations, tests, reviews, and acceptance gates must respect.

## 1. Purpose

The purpose of this document is to make XPScerpto’s authority model explicit.

In XPScerpto, a subsystem must not decide for itself that it is allowed to use a capability merely because that capability appears to exist. Execution authority must be admitted through the governed platform and hardware evidence chain.

The authority model exists to prevent:

- implicit runtime decisions
- unauthorized hardware probing
- hidden platform assumptions
- domain-local capability inference
- unsafe fallback behavior
- unreviewed dispatch paths
- overclaims about validated execution

Authority is a first-class engineering boundary.

## 2. Core authority principle

The core authority principle is:

```text
No domain execution without admitted authority.
No admitted authority without evidence.
No evidence without reviewable boundaries.
```

A domain may execute only under authority that has been admitted through the approved platform and hardware chain.

The existence of a feature does not grant permission to use it.

A successful probe does not grant authority.

A compiler macro does not grant authority.

A local runtime observation does not grant authority.

A passing narrow test does not grant authority beyond its validated scope.

## 3. Authority chain

The expected authority flow is:

```text
platform governance
→ platform.api
→ hardware authority
→ routing capsule
→ domain execution
```

This chain defines the normal path through which execution decisions become admissible.

The chain is intentionally ordered:

1. governance defines what may be trusted;
2. `platform.api` exposes the admitted platform surface;
3. hardware authority provides evidence-backed capability truth;
4. the routing capsule carries approved execution decisions;
5. domain code executes only within the admitted decision.

Any implementation that bypasses this order must be treated as an authority violation unless explicitly reviewed, documented, and accepted by the relevant governance process.

## 4. Authority vocabulary

This document uses the following terms:

### Authority

Authority is permission to make or execute a decision within a defined scope.

Authority is not the same as capability. A machine may have a capability that a domain is not authorized to use.

### Evidence

Evidence is reviewable information that supports an authority decision.

Evidence may include platform records, hardware snapshots, routing decisions, audit ledgers, build logs, test logs, CI logs, sanitizer runtime logs, and final flags.

### Admission

Admission is the process by which evidence becomes an approved authority decision.

Unadmitted evidence must not be treated as execution permission.

### Domain

A domain is a subsystem that performs specialized work under admitted authority. Examples include cryptographic domains, SIMD execution, hardware-assisted paths, and other protected runtime units.

### Routing capsule

A routing capsule is the approved carrier of runtime decision information from the authority layer to the domain layer.

It should carry decisions that have already passed through the governed authority chain. It must not be replaced by domain-local probing.

## 5. Platform governance

Platform governance defines the rules under which authority may be created, reviewed, admitted, and rejected.

Governance includes repository policy, protected surfaces, review requirements, evidence requirements, claim discipline, and acceptance gates.

Platform governance does not merely describe the system. It constrains implementation behavior.

A runtime shortcut that violates governance is not acceptable even if it improves convenience, build success, or benchmark performance.

## 6. `platform.api` as the admitted surface

`platform.api` is the approved platform-facing surface through which platform authority is exposed to the rest of the system.

Domain code should depend on admitted platform surfaces, not on internal platform implementation details.

The following behavior is not acceptable without explicit governance approval:

- importing private platform internals from domain code
- bypassing `platform.api` to reach provider internals
- using platform runtime state that is not exposed through an admitted surface
- creating parallel authority paths outside the approved chain
- replacing platform admission with local convenience logic

`platform.api` exists to keep platform authority reviewable, narrow, and enforceable.

## 7. Hardware authority

Hardware authority is the evidence-backed understanding of hardware capabilities, topology, constraints, and routing conditions.

Hardware authority must be obtained through the approved hardware layer. Domain code must not create its own hardware truth.

Hardware authority may include:

- capability evidence
- topology evidence
- runtime availability
- platform constraints
- dispatch eligibility
- safety restrictions
- audit records

A hardware capability is not automatically admissible for domain execution. It must be admitted through the authority chain.

## 8. Routing capsule

The routing capsule carries approved execution decisions from the authority layer to domain execution.

A routing capsule should make runtime decisions explicit, reviewable, and auditable.

A routing capsule must not be treated as a decorative wrapper around a local probe. It is the boundary object that prevents domains from inventing their own execution authority.

A valid routing capsule should preserve:

- source of authority
- admitted capability scope
- relevant platform constraints
- relevant hardware evidence
- dispatch decision
- auditability
- fail-closed behavior when authority is incomplete

## 9. Domain execution

Domain execution is the final consumer of admitted authority.

Domain code may execute within the scope granted by the authority chain. It must not expand that scope on its own.

A domain must not:

- probe for capabilities locally when governed authority is required
- infer authority from compiler macros
- infer authority from operating-system availability
- infer authority from successful dynamic execution
- bypass routing decisions for performance
- silently downgrade to an unreviewed fallback
- convert missing authority into implicit permission

A domain may reject work, fail closed, or request an admitted decision. It must not manufacture the decision itself.

## 10. Forbidden authority sources

The following sources must not be treated as domain execution authority unless they are admitted through the approved authority chain:

- direct CPU probing
- direct operating-system probing
- compiler feature macros
- compiler target flags
- environment variables
- hidden runtime globals
- cached local state outside the authority chain
- benchmark observations
- successful trial execution
- test-only shortcuts
- historical logs
- metadata-only reports
- undocumented platform assumptions

These sources may be useful as raw inputs only when the approved authority layer is responsible for collecting, validating, constraining, and admitting them.

They are not domain authority by themselves.

## 11. Build-time authority versus runtime authority

Build-time configuration and runtime authority are different concepts.

Build-time flags may affect what code is compiled, but they must not be confused with runtime permission to execute a capability.

A compiler flag such as an instruction-set option does not by itself authorize a domain to dispatch to that instruction set.

A platform define may describe a build assumption, but it does not replace runtime authority when runtime authority is required.

A valid implementation must preserve the distinction between:

```text
compiled capability
available capability
admitted capability
executed capability
```

Only admitted capability may be used for governed domain execution.

## 12. SIMD authority rules

SIMD execution is authority-routed domain execution.

SIMD code must not decide hardware eligibility by reading CPU state, compiler macros, operating-system files, compiler builtins, or hidden runtime state on its own.

The expected SIMD authority path is:

```text
hardware evidence
→ hardware decision
→ SIMD routing signal
→ SIMD dispatch plan
→ SIMD execution
```

SIMD performance work must remain inside this authority path.

The following patterns are not acceptable as SIMD authority:

- local CPUID decisions inside SIMD domain code
- compiler macro inference such as assuming an instruction set from a build flag
- direct use of compiler CPU-support builtins as domain authority
- parsing operating-system hardware files from SIMD code
- choosing a fast path because a benchmark suggests it works
- bypassing admitted routing for a synthetic performance win

SIMD may be fast only when it remains governed.

## 13. FHE authority rules

FHE execution must preserve cryptographic authority, context binding, parameter validity, key lifecycle discipline, and claim boundaries.

FHE code must not present partial cryptographic closure as full subsystem acceptance.

FHE authority includes more than the ability to run arithmetic. It includes the validity of the context, parameters, keys, encoders, evaluators, scale or modulus state, rotation or switching material, error budget, and acceptance scope.

The following patterns are not acceptable:

- using test-only behavior as production authority
- treating plaintext reference behavior as encrypted-path acceptance
- presenting metadata-only checks as cryptographic proof
- using secret material outside an admitted key lifecycle path
- claiming bootstrap completion from a partial transform
- claiming FHE production readiness from BFV-only or CKKS-only closure
- bypassing scale, level, modulus, rotation, or key-binding checks

FHE claims must remain scoped to the exact gate that established them.

## 14. Cryptographic key and context authority

Keys, contexts, parameter sets, and runtime sessions carry authority-sensitive meaning.

A key valid in one context must not be silently accepted in another context unless the implementation proves that the binding is valid.

A parameter set must not be accepted merely because it is structurally well-formed. It must satisfy the required security, correctness, and runtime constraints for the operation being performed.

Key lifecycle behavior must be explicit, testable, and auditable.

Invalid, stale, mismatched, expired, missing, or unauthorized key material must fail closed.

## 15. Evidence and audit

Authority decisions should be auditable.

An authority-sensitive path should be able to explain:

- what decision was made
- what authority admitted the decision
- what evidence supported the decision
- what scope the decision covers
- what domain consumed the decision
- what happened when required authority was missing

Audit evidence must be precise enough to reject overclaims.

A log that cannot distinguish admitted execution from fallback behavior is not sufficient for authority-sensitive acceptance.

## 16. Fail-closed authority behavior

Missing authority must not become implicit permission.

If the required authority cannot be established, the implementation must fail closed.

Fail-closed behavior may include:

- rejecting the operation
- returning an explicit unsupported result
- identifying a missing authority source
- identifying a missing routing decision
- identifying an invalid context or key binding
- identifying an incomplete evidence chain
- identifying the exact blocker

The result must not silently fall back to a less governed path and present that path as accepted execution.

## 17. Authority and performance

Performance must not weaken authority.

A high-performance path must satisfy the same authority requirements as any other path.

The following are not acceptable performance justifications:

- the fast path works on this machine
- the compiler generated the instruction
- the operating system exposes the capability
- the benchmark passed
- the fallback is slower
- the shortcut is internal
- the authority path is inconvenient

A performance improvement is valid only when it preserves correctness, authority, evidence, fail-closed behavior, and claim discipline.

## 18. Authority and documentation

Documentation must reflect the real authority model.

Documentation must not describe a path as governed, admitted, accepted, complete, production-ready, or secure unless the evidence supports that description.

If a path is planned, experimental, blocked, partially implemented, or test-only, the documentation must say so clearly.

Documentation that changes authority claims must be reviewed as an authority-sensitive change.

## 19. Authority review requirements

A review of authority-sensitive code must ask:

- What authority does this code depend on?
- Where does that authority come from?
- Is the authority admitted through the approved chain?
- What evidence supports the decision?
- What scope does the authority cover?
- What happens when authority is missing?
- Does the code introduce local probing?
- Does the code infer permission from capability?
- Does the code bypass `platform.api` or hardware authority?
- Does the code preserve fail-closed behavior?
- Does the documentation claim more than the evidence proves?

A change that cannot answer these questions is not ready for acceptance.

## 20. Relationship to other documents

This document should be read after:

```text
START_HERE.md
GOVERNANCE.md
ENGINEERING_PRINCIPLES.md
```

It should be read before:

```text
ARCHITECTURE.md
MODULE_BOUNDARIES.md
THREAT_MODEL.md
CI_VALIDATION_GATES.md
```

`GOVERNANCE.md` defines the rules for acceptance.

`ENGINEERING_PRINCIPLES.md` defines the engineering discipline.

`AUTHORITY_MODEL.md` defines the source and flow of execution authority.

`ARCHITECTURE.md` should describe the system structure in a way that preserves this authority model.

## 21. Final rule

The final authority rule is:

```text
capability is not authority
authority is not trust without evidence
evidence is not acceptance without review
execution is not valid outside admitted scope
```

XPScerpto advances only when execution authority, implementation behavior, test evidence, audit records, and public claims remain aligned.
