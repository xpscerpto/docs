# XPScerpto Review Policy

**Status:** normative review policy  
**Scope:** repository-wide review requirements, protected-surface review, authority-impact review, module-boundary review, security-sensitive review, cryptographic review, test review, CI validation review, evidence review, documentation-claim review, approval rules, and rejection rules

XPScerpto is a governed sovereign cryptographic infrastructure platform. Review in XPScerpto is not limited to code style, local correctness, or whether a change appears to work on one machine. Review is the process of deciding whether a change preserves authority, boundaries, evidence, security posture, validation truth, and claim discipline.

This document defines how changes should be reviewed before acceptance.

This document is not a validation report, security proof, production-readiness claim, or acceptance artifact. It defines the review discipline that must be applied before such claims may be accepted.

## 1. Purpose

The purpose of this document is to prevent shallow approval.

A change may compile and still be unacceptable.

A test may pass and still be insufficient.

A patch may look small and still weaken authority.

A documentation update may appear harmless and still overclaim the project’s state.

A review must determine whether the change is correct within XPScerpto’s governed architecture, not merely whether the diff is readable.

## 2. Review principle

The central review principle is:

```text
no approval without evidence
no approval without authority review
no approval without boundary review
no approval when claims exceed validation
```

A reviewer is not approving only code. A reviewer is approving that the change, within its stated scope, preserves:

- correctness;
- authority flow;
- module boundaries;
- protected-surface ownership;
- security posture;
- test integrity;
- CI and validation truth;
- evidence integrity;
- documentation accuracy;
- claim discipline.

If any of these dimensions is unclear, the reviewer should request changes or block acceptance until the issue is resolved.

## 3. What review means in XPScerpto

Review means evaluating whether a change is safe to accept under the repository’s governance model.

A valid review must consider:

- what changed;
- why it changed;
- which surfaces were affected;
- what authority the change depends on;
- whether module boundaries remain valid;
- whether tests still measure the intended claim;
- whether CI evidence supports the claim;
- whether sanitizer evidence is required;
- whether security impact was considered;
- whether documentation remained honest;
- whether incomplete work fails closed with an exact blocker.

A review that checks only formatting, naming, or local compilation is insufficient for protected surfaces.

## 4. Review outcomes

A reviewer should use clear outcomes.

Acceptable review outcomes include:

```text
APPROVE
REQUEST_CHANGES
COMMENT_ONLY
BLOCKED_WITH_REASON
REJECTED
```

A review should not use vague approval language when evidence is missing.

Avoid ambiguous outcomes such as:

```text
looks fine
probably okay
seems safe
works for me
ship it
close enough
```

A vague approval weakens the review record.

## 5. Required review mindset

Reviewers must assume that mistakes may occur in several forms:

- implementation mistakes;
- authority mistakes;
- boundary mistakes;
- security mistakes;
- test weakening;
- CI scope confusion;
- sanitizer overclaim;
- evidence mismatch;
- documentation overclaim;
- release claim inflation.

A reviewer should ask not only:

```text
Does this work?
```

but also:

```text
What does this prove?
What does this not prove?
What claim does this change make?
What evidence supports that claim?
What boundary could this weaken?
What would fail if this were wrong?
```

## 6. Protected-surface review

Protected surfaces require strict review.

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

A protected-surface review must identify:

- the protected surface touched;
- the owner or expected reviewer;
- the authority impact;
- the boundary impact;
- the test impact;
- the CI impact;
- the security impact;
- the evidence required for acceptance.

A protected-surface change must not be approved because it is small. Small changes may still alter authority, validation, or public claims.

## 7. Authority-impact review

Any change that affects execution authority must be reviewed against the authority model.

The expected authority flow is:

```text
platform governance
→ platform.api
→ hardware authority
→ routing capsule
→ domain execution
→ audit and evidence
```

A reviewer must reject or block a change that:

- creates domain-local authority;
- bypasses `platform.api`;
- imports platform internals for convenience;
- treats hardware capability as execution permission;
- uses compiler macros as runtime authority;
- uses local CPU probing as domain authority;
- uses hidden runtime state as permission;
- treats fallback behavior as admitted execution;
- hides missing authority behind a successful test.

Capability is not authority.

A reviewer should approve authority-sensitive changes only when the admitted authority path is explicit and supported by evidence.

## 8. Module-boundary review

A reviewer must verify that module boundaries remain valid.

The review should determine whether any change:

- imports internal surfaces from unrelated families;
- introduces a new public surface;
- changes canonical public API ownership;
- reintroduces a retired surface;
- creates an alias, shim, adapter, or compatibility wrapper;
- moves test-only behavior into production;
- moves production behavior into tests;
- widens visibility to make a failing import compile;
- hides an ownership conflict.

Boundary changes must be intentional, documented, and tested.

A reviewer should reject broad imports, alias masking, or convenience adapters that hide a real boundary problem.

## 9. Security-sensitive review

A change is security-sensitive if it affects:

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

Security-sensitive changes require careful review and evidence appropriate to their scope.

A reviewer must not approve a security-sensitive change only because the patch appears reasonable. The change must be validated.

## 10. Cryptographic review

Cryptographic changes require conservative review.

A cryptographic review should ask:

- what security property is affected;
- what input, key, seed, parameter, context, or state changed;
- what invalid behavior is rejected;
- what new failure modes exist;
- what negative tests were added;
- what regression tests protect the fix;
- whether the claim is scoped correctly;
- whether the evidence measures the claimed behavior.

Cryptographic review must reject:

- metadata-only proof presented as runtime validation;
- test vectors presented as full hardening;
- plaintext reference behavior presented as encrypted-path acceptance;
- partial subsystem success presented as global acceptance;
- parameter acceptance without validation;
- hidden fallback behavior;
- documentation that claims more than tests prove.

Cryptographic code should be reviewed for both correctness and misuse resistance.

## 11. FHE and advanced-crypto review

Some cryptographic subsystems require additional review because correctness depends on many interacting stages.

For FHE, reviewers should consider:

- BFV correctness and claim scope;
- CKKS correctness and claim scope;
- bootstrap claim scope;
- parameter and context binding;
- key lifecycle;
- key switching;
- relinearization;
- Galois and rotation behavior;
- NTT and RNS arithmetic;
- scale, level, modulus, and error behavior;
- encoder and decoder behavior;
- public API ownership;
- surface guards;
- test-only versus production behavior.

This section does not make the review policy FHE-specific. FHE is listed because it is one high-risk subsystem where overclaim risk is especially high.

A reviewer must not allow:

- BFV acceptance to imply CKKS acceptance;
- CKKS acceptance to imply bootstrap completion;
- bootstrap progress to imply production readiness;
- FHE progress to imply repository-wide production readiness.

Every cryptographic claim must remain scoped to the gate and evidence that proved it.

## 12. Platform and provider review

Platform and provider changes must preserve platform admission and provider isolation.

A reviewer should ask:

- does the change expose platform internals;
- does it alter provider admission;
- does it affect unsupported provider behavior;
- does it change platform runtime state;
- does it bypass `platform.api`;
- does it affect platform evidence;
- does it weaken fail-closed behavior;
- does it change provider ownership or registration.

A platform change must not be approved if it allows domain code to reach private platform behavior for convenience.

Unsupported provider paths must remain explicit and fail closed unless accepted by the relevant platform gate.

## 13. Hardware and routing review

Hardware and routing changes must preserve evidence-backed decision making.

A reviewer should ask:

- what hardware evidence is collected;
- how raw capability becomes admitted capability;
- what routing decision is produced;
- what scope the decision covers;
- how unsupported or conflicting evidence fails;
- whether domains are prevented from local probing;
- whether audit evidence distinguishes admitted execution from fallback.

A reviewer must reject:

- domain-local hardware authority;
- routing wrappers that hide local probing;
- benchmark-based authority;
- dispatch decisions based on hidden state;
- fast paths outside admitted routing.

## 14. SIMD review

SIMD changes must preserve both correctness and authority routing.

A reviewer should ask:

- is dispatch authority admitted or locally inferred;
- are scalar and accelerated paths both correct;
- are bounds and overlap behavior tested;
- does the change rely on compiler ISA macros as runtime permission;
- does the change use CPU probing inside the SIMD domain;
- does the performance claim identify hardware authority and dispatch eligibility;
- does the benchmark scope match the claim.

A SIMD performance improvement must not be approved if it bypasses authority.

Performance is valuable only inside the governed execution model.

## 15. Test review

Tests must be reviewed as protected validation surfaces.

A reviewer must check whether tests were:

- added;
- removed;
- renamed;
- weakened;
- narrowed;
- skipped;
- re-labelled;
- converted from assertions to diagnostics;
- changed to accept weaker behavior;
- changed to hide a blocker.

A test change is acceptable when it improves accuracy, updates stale expectations after legitimate behavior changes, adds missing coverage, or removes obsolete test-only scaffolding with documented reason.

A test change is not acceptable when it makes a phase pass by weakening the original claim.

The reviewer should ask:

```text
What truth did this test protect?
Does the new version still protect that truth?
```

## 16. CI and validation review

CI and validation changes affect claim discipline and must be reviewed carefully.

A reviewer should ask:

- what claim does this gate prove;
- what evidence does it produce;
- what happens on failure;
- does it preserve exact blockers;
- does it hide skipped tests;
- does it weaken labels;
- does it change sanitizer meaning;
- does it affect full graph truth;
- does it alter final flags;
- does it make overclaim easier.

A reviewer must reject CI changes that:

- turn configure success into build success;
- turn build success into test success;
- turn sanitizer configuration into sanitizer clean;
- turn target-only execution into full graph truth;
- hide missing executables;
- silently skip failing tests;
- remove required labels without governance approval.

CI is part of security and governance.

## 17. Sanitizer review

Sanitizer claims require runtime evidence.

A reviewer must reject sanitizer-clean claims when:

- only configuration ran;
- only build ran;
- runtime did not execute;
- the wrong target set ran;
- required executables were missing;
- sanitizer logs are absent;
- sanitizer failures were observed;
- sanitizer failure was reclassified without evidence.

A sanitizer failure may still be a useful result. It may identify the exact blocker. It must not be called clean.

Sanitizer truth requires runtime execution and preserved logs.

## 18. Evidence review

A reviewer must verify that evidence matches the claim.

Evidence review should check:

- source tree identity;
- patch or diff identity;
- commands executed;
- build configuration;
- target scope;
- test scope;
- sanitizer scope;
- CI job scope;
- logs;
- final flags;
- exact blockers;
- whether evidence is live or historical;
- whether historical evidence is being used only as provenance.

A reviewer must reject evidence that is stale, incomplete, from a different source tree, missing required logs, missing final flags when required, or broader in summary than in execution.

Evidence does not prove more than its scope.

## 19. Documentation and claim review

Documentation changes must be reviewed for claim discipline.

A documentation change requires careful review if it:

- describes project status;
- changes production-readiness language;
- describes security posture;
- describes sanitizer status;
- describes subsystem acceptance;
- describes bootstrap completion;
- describes full graph truth;
- describes release status;
- changes governance, authority, boundary, threat, CI, or review policy.

A reviewer must reject documentation that:

- presents planned behavior as accepted;
- presents implemented behavior as validated;
- presents subsystem success as repository-wide success;
- claims sanitizer clean without runtime logs;
- claims production readiness without the production gate;
- hides blockers behind aspirational language;
- uses marketing language to bypass evidence.

Documentation must tell the same truth as implementation and evidence.

## 20. Release and advisory review

Release notes, security advisories, and public status statements require claim review.

A reviewer should verify:

- affected scope;
- fixed scope;
- validation evidence;
- remaining limitations;
- claim boundaries;
- supported configurations;
- final flags, if applicable;
- whether unresolved blockers are disclosed accurately.

A release note must not convert partial validation into global acceptance.

A security advisory must not imply that the entire repository is secure or production-ready unless the relevant gates prove that exact claim.

## 21. Approval rules

A reviewer may approve a change when:

- the change scope is clear;
- protected surfaces are identified;
- authority impact is understood;
- module boundaries are preserved;
- security impact is reviewed;
- tests are appropriate to the claim;
- CI evidence is sufficient for the claim;
- sanitizer evidence is present when sanitizer claims are made;
- documentation matches evidence;
- incomplete work, if any, fails closed with an exact blocker;
- claims remain scoped to the gates that proved them.

Approval should be explicit.

For high-risk changes, approval should state the basis for acceptance, such as:

```text
Approved for the stated scope. Evidence supports the claimed target-level fix. No production-ready, full graph, or sanitizer-clean claim is made by this approval.
```

## 22. Request-changes rules

A reviewer should request changes when:

- scope is unclear;
- evidence is missing;
- tests do not measure the claim;
- authority impact is not explained;
- module boundaries are ambiguous;
- protected surfaces were changed without review context;
- CI output is incomplete;
- sanitizer claims lack runtime logs;
- documentation overclaims;
- failure handling is vague;
- exact blockers are missing.

Requesting changes is the correct outcome when the change may be repairable but is not ready for acceptance.

## 23. Rejection rules

A reviewer should reject a change when it:

- bypasses authority;
- hides a boundary violation;
- weakens tests to create a pass;
- deletes assertions without justified replacement;
- uses aliases, shims, or adapters to mask ownership conflicts;
- imports test-only behavior into production;
- presents metadata-only proof as runtime validation;
- presents sanitizer configuration as sanitizer clean;
- presents target-only success as full graph truth;
- presents partial subsystem success as global acceptance;
- hides exact blockers;
- changes final flags without gate evidence;
- claims production readiness without the required gate.

A change may be rejected even if it compiles.

## 24. Handling incomplete work

Incomplete work must fail closed.

A reviewer may accept incomplete diagnostic work only if:

- it is clearly marked as non-acceptance work;
- it does not elevate claims;
- it preserves exact blockers;
- it does not weaken tests or governance;
- it does not alter production behavior unsafely;
- it is useful for isolating a real issue.

Incomplete work must not be merged as acceptance unless the governance process explicitly allows that scope.

Preferred incomplete result language:

```text
NO_WITH_REASON
PATH_B_FAIL_CLOSED
BLOCKED
NOT_CLAIMED
EXACT_BLOCKER=...
```

## 25. Review of generated or tool-produced changes

Generated or tool-produced changes are not exempt from review.

A reviewer must check:

- what generated the change;
- whether generated output is deterministic or explainable;
- whether protected surfaces changed;
- whether generated output introduced broad imports;
- whether tests were changed;
- whether evidence was changed;
- whether documentation claims changed;
- whether the generator itself needs review.

Generated code can introduce boundary, security, and claim-discipline failures.

## 26. Reviewer conflict and independence

A reviewer should avoid approving changes they cannot evaluate.

A reviewer should request additional review when:

- the change touches a domain outside their expertise;
- the change is security-sensitive;
- the change affects cryptographic behavior;
- the change affects CI gates;
- the change affects authority flow;
- the change affects protected public surfaces;
- the change alters release claims.

Approval should be based on understanding, not ownership title alone.

## 27. Relationship to CODEOWNERS

`CODEOWNERS` defines who should be requested for review.

This document defines how review should be performed.

A CODEOWNERS approval is not sufficient if the change violates governance, authority, boundary, security, CI, or evidence requirements.

Likewise, a passing test result is not sufficient if required ownership review was bypassed.

Ownership and evidence must both be satisfied.

## 28. Relationship to CONTRIBUTING

`CONTRIBUTING.md` should explain how contributors prepare changes.

`REVIEW_POLICY.md` explains how reviewers evaluate changes.

A contributor may follow the contribution process correctly and still receive requested changes if the evidence is insufficient, the boundary impact is unclear, or the claim exceeds validation.

## 29. Relationship to CI validation gates

`CI_VALIDATION_GATES.md` defines what must be proven by validation.

`REVIEW_POLICY.md` defines how reviewers judge whether that proof is sufficient for the claim being made.

A reviewer should not approve a claim that CI did not prove.

A reviewer may still request changes after CI passes if authority, boundary, documentation, or security issues remain.

## 30. Relationship to governance process

Major architecture, authority, ownership, validation, security-process, dependency-policy, public-surface, or long-term compatibility changes may require an RFC before implementation.

Reviewers may require the RFC process defined in:

```text
docs/governance/rfc-process.md
```

Accepted, rejected, superseded, or long-term decisions may require a decision record using:

```text
docs/governance/decision-record-template.md
```

The governance process index is:

```text
docs/governance/README.md
```

A reviewer should request an RFC or decision record when a change is too important to be decided only inside a normal pull request.

An accepted RFC or decision record does not replace implementation evidence, CI validation, sanitizer runtime evidence, or claim boundaries.

## 30. Reviewer checklist

Before approval, a reviewer should be able to answer:

- What changed?
- Why did it change?
- What surfaces were affected?
- Is any protected surface touched?
- What authority does the change depend on?
- Does it preserve the authority chain?
- Does it preserve module boundaries?
- Does it affect cryptographic behavior?
- Does it affect keys, seeds, randomness, parameters, contexts, or credentials?
- Does it affect memory or concurrency safety?
- Were tests added or changed?
- Were any tests weakened, skipped, deleted, or re-labelled?
- What CI evidence exists?
- Is sanitizer evidence required?
- If sanitizer is claimed, did runtime execute?
- Does documentation match evidence?
- Does the change alter final flags or release claims?
- If incomplete, is the exact blocker preserved?
- Is the approval scope clear?

If the reviewer cannot answer these questions for a protected or security-sensitive change, approval is premature.

## 32. Current claim boundaries

This review policy does not elevate project status.

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

A review policy is not validation evidence. It defines how reviewers decide whether evidence is sufficient.

## 33. Final review rule

The final review rule is:

```text
approve only what was proven
reject what weakens authority
block what hides boundaries
request evidence when claims exceed validation
preserve exact blockers when work is incomplete
```

XPScerpto review is successful only when the implementation, authority path, module boundaries, tests, CI evidence, documentation, and claims remain aligned.
