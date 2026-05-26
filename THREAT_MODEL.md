# XPScerpto Threat Model

**Status:** normative threat model  
**Scope:** repository-wide threat assumptions, protected assets, trust boundaries, adversary classes, authority-chain threats, platform threats, hardware-routing threats, SIMD threats, FHE and cryptographic threats, build and CI threats, evidence threats, documentation-claim threats, and fail-closed expectations

XPScerpto is a governed sovereign cryptographic infrastructure platform. Its security posture is built on explicit authority, controlled module boundaries, fail-closed behavior, conservative cryptographic engineering, and evidence-backed validation.

This document defines the threat model for XPScerpto. It describes what the project is designed to resist, what assumptions it relies on, what boundaries must be protected, what classes of attacks are in scope, and what claims must not be made without validation.

This document is not a validation report, security proof, production-readiness claim, or acceptance artifact. It defines the threats that implementation, review, testing, CI, and acceptance gates must address.

## 1. Purpose

The purpose of this threat model is to prevent XPScerpto from being reviewed or implemented as a normal cryptographic utility library.

XPScerpto has a broader security target. It must protect not only cryptographic algorithms, but also:

- execution authority;
- module boundaries;
- platform admission;
- hardware evidence;
- routing decisions;
- key and context binding;
- build and CI truth;
- sanitizer truth;
- evidence integrity;
- documentation claim discipline.

A defect that bypasses authority, weakens validation, hides a blocker, or creates a false claim may be security-relevant even when it does not immediately reveal a cryptographic secret.

## 2. Security posture

XPScerpto follows a conservative security posture:

```text
do not trust ambient capability
do not trust local probing
do not trust hidden state
do not trust partial evidence
do not convert missing evidence into success
fail closed when authority or validation is incomplete
```

The project must prefer precise refusal over ambiguous success.

A secure result is not merely one that produces the expected output. A secure result must preserve authority, boundaries, evidence, and claim scope.

## 3. Assets protected

The threat model protects the following assets.

### 3.1 Cryptographic assets

- secret keys;
- private keys;
- seed material;
- PRNG state;
- FHE secret material;
- switching and rotation material where sensitive;
- encrypted payloads;
- plaintexts before encryption or after decryption;
- authentication material;
- parameter sets that define security level;
- contexts that bind keys and operations.

### 3.2 Authority assets

- platform admission decisions;
- platform provider state;
- `platform.api` authority surfaces;
- hardware evidence;
- routing capsules;
- SIMD dispatch decisions;
- FHE context and execution authority;
- audit ledgers;
- final flags and acceptance records.

### 3.3 Boundary assets

- public API ownership;
- protected internal surfaces;
- test-only surfaces;
- tooling-only surfaces;
- module import rules;
- build graph materialization;
- CI validation gates;
- documentation claim boundaries.

### 3.4 Evidence assets

- build logs;
- CTest logs;
- sanitizer runtime logs;
- CI logs;
- evidence bundles;
- failure reports;
- exact blockers;
- final flags;
- reviewed merge records;
- release notes.

Evidence is an asset because XPScerpto uses evidence to prevent false acceptance.

## 4. Trust boundaries

XPScerpto recognizes several trust boundaries.

### 4.1 Governance boundary

The governance boundary separates accepted behavior from unreviewed or aspirational behavior.

A change that has not passed the relevant review and evidence gates must not be treated as accepted.

### 4.2 Platform boundary

The platform boundary separates platform-owned authority and provider behavior from domain execution.

Domain code must not bypass `platform.api` to reach platform internals.

### 4.3 Hardware authority boundary

The hardware boundary separates evidence-backed hardware truth from domain-local capability assumptions.

Domains must not create their own hardware authority.

### 4.4 Routing boundary

The routing boundary separates admitted decisions from execution.

Domains consume routing decisions. They must not replace routing with local probing.

### 4.5 Cryptographic boundary

The cryptographic boundary protects keys, contexts, parameters, encoded data, encrypted data, arithmetic semantics, and failure behavior.

Cryptographic correctness must not be inferred from metadata-only checks or plaintext reference behavior.

### 4.6 Test boundary

The test boundary separates validation behavior from production behavior.

Tests may inspect, stress, or simulate behavior. They must not become production authority.

### 4.7 Documentation boundary

The documentation boundary separates goals, design intent, validated behavior, blocked behavior, and production-ready claims.

Documentation must not convert intent into accepted fact.

## 5. Assumptions

This threat model relies on the following assumptions.

- Reviewers apply `GOVERNANCE.md`, `ENGINEERING_PRINCIPLES.md`, `AUTHORITY_MODEL.md`, `ARCHITECTURE.md`, and `MODULE_BOUNDARIES.md`.
- Protected surfaces receive appropriate review.
- CI and local validation are not intentionally falsified by maintainers.
- The compiler, build system, operating system, and hardware may have bugs, but they are not assumed to be malicious unless stated otherwise.
- Cryptographic security requires correct implementation, correct parameters, correct key lifecycle behavior, and appropriate validation gates.
- Documentation is treated as policy only when it is normative and reviewed.
- Historical evidence may establish provenance, but live validation is required when a gate requires live validation.

These assumptions must not be used to avoid testing. They define the baseline under which the threat model is meaningful.

## 6. Non-goals and limits

XPScerpto does not claim protection against every possible threat.

Unless a dedicated acceptance gate proves otherwise, this threat model does not claim:

- production readiness;
- complete FHE bootstrap closure;
- full sanitizer CTest cleanliness;
- formal verification of all code;
- protection from a fully malicious compiler;
- protection from a fully compromised operating system;
- protection from physical attacks;
- protection from all side channels;
- protection from all microarchitectural leakage;
- protection from all supply-chain attacks;
- protection from malicious maintainers with merge authority;
- cryptographic security for parameter sets that have not been validated;
- security for test-only or experimental paths;
- security for undocumented deployment configurations.

A non-goal is not a weakness by itself. It is a claim boundary.

## 7. Adversary model

XPScerpto considers multiple adversary classes.

### 7.1 External attacker

An external attacker may attempt to exploit exposed APIs, malformed inputs, invalid cryptographic parameters, corrupted ciphertexts, boundary conditions, denial-of-service behavior, or failure-mode ambiguity.

### 7.2 Malicious or careless contributor

A contributor may intentionally or accidentally introduce shortcuts, weaken tests, bypass authority, hide blockers, create aliases, reintroduce retired surfaces, or overclaim validation status.

### 7.3 Boundary violator

A boundary violator may import internal modules, bypass `platform.api`, treat test-only helpers as production behavior, or create hidden dependency paths.

### 7.4 Authority forger

An authority forger may attempt to create execution permission from local probes, compiler macros, environment variables, hidden runtime state, benchmark observations, or metadata-only reports.

### 7.5 Evidence forger or evidence confuser

An evidence adversary may attempt to present historical logs as live validation, sanitizer configuration as sanitizer truth, target-only success as full graph truth, or partial subsystem closure as global acceptance.

### 7.6 Platform or provider confusion attacker

This attacker attempts to exploit provider selection, platform admission, unsupported platform paths, foreign provider stubs, or runtime ambiguity.

### 7.7 Cryptographic misuse attacker

This attacker attempts to exploit invalid parameters, context mismatch, key reuse, stale key material, improper randomness, incorrect encoding, incorrect scale or level handling, or unsafe fallback behavior.

### 7.8 Documentation-claim attacker

This attacker attempts to make the project appear more complete, secure, validated, or production-ready than the evidence proves.

This includes accidental overclaim by maintainers or contributors.

## 8. Authority-chain threats

The authority chain is:

```text
platform governance
→ platform.api
→ hardware authority
→ routing capsule
→ domain execution
→ audit and evidence
```

Threats against this chain include:

- domain code bypassing `platform.api`;
- hardware capability inferred inside a domain;
- routing decisions replaced by local probing;
- hidden runtime globals used as authority;
- platform internals imported for convenience;
- test-only authority used in production;
- missing evidence treated as permission;
- fallback behavior presented as admitted execution;
- audit records too vague to distinguish admitted execution from fallback.

Required defenses include:

- explicit authority surfaces;
- narrow public APIs;
- import-boundary enforcement;
- fail-closed behavior;
- exact blockers;
- authority-routing tests;
- audit evidence that identifies the admitted path.

## 9. Platform threats

Platform threats include:

- unauthorized provider admission;
- provider behavior leaking across platform boundaries;
- unsupported provider paths treated as active runtime paths;
- foreign provider stubs mistaken for native support;
- platform internals imported by domains;
- provider-specific assumptions embedded in domains;
- platform runtime state exposed through unreviewed APIs;
- platform policy bypassed by direct includes or compatibility shortcuts.

Required defenses include:

- `platform.api` as the admitted surface;
- provider ownership discipline;
- strict review of platform changes;
- fail-closed unsupported-provider behavior;
- tests that distinguish admitted provider behavior from stubs or unsupported paths;
- documentation that does not overclaim platform support.

## 10. Hardware authority threats

Hardware authority threats include:

- direct CPU probing inside domains;
- operating-system hardware file probing inside domains;
- compiler builtins treated as authority;
- compiler ISA macros treated as runtime permission;
- cached local hardware state outside the authority chain;
- benchmark results used as dispatch authority;
- topology or capability evidence used outside admitted scope;
- conflicting hardware evidence converted into success.

Required defenses include:

- approved hardware evidence collection;
- clear separation between raw capability and admitted capability;
- routing capsules or equivalent decision objects;
- fail-closed behavior on incomplete hardware truth;
- hardware authority tests;
- audit records for dispatch-relevant decisions.

## 11. Routing and dispatch threats

Routing and dispatch threats include:

- routing wrappers that merely disguise local probing;
- unauthorized fast paths;
- silent fallback from governed to ungoverned execution;
- fallback paths presented as accepted execution;
- dispatch decisions based on hidden state;
- dispatch selected by benchmark observation instead of admitted authority;
- domain code expanding the scope of a routing decision.

Required defenses include:

- explicit dispatch plans;
- scope-bound routing decisions;
- fail-closed missing-route behavior;
- negative tests for unauthorized routing;
- auditability of dispatch decisions;
- documentation that identifies the admitted routing path.

## 12. SIMD threats

SIMD threats include:

- local CPUID authority inside SIMD code;
- compiler CPU-support builtins used as domain authority;
- ISA macro inference used as runtime authority;
- direct operating-system hardware probing from SIMD code;
- unauthorized vector path execution;
- performance shortcuts outside admitted routing;
- overlap, bounds, or aliasing errors in memory movement;
- benchmark overclaims that ignore authority eligibility.

Required defenses include:

- hardware-to-SIMD routing discipline;
- correctness tests across scalar and accelerated paths;
- bounds and overlap tests;
- authority-routing tests;
- benchmark reports that distinguish performance from authorization;
- fail-closed behavior when routing is not admitted.

SIMD performance is valid only inside the governed execution model.

## 13. FHE threats

FHE threats include:

- invalid parameter acceptance;
- context mismatch;
- stale, missing, or unauthorized key material;
- secret key leakage through diagnostics or test paths;
- test-only behavior presented as production behavior;
- plaintext reference behavior presented as encrypted-path acceptance;
- metadata-only checks presented as cryptographic proof;
- incorrect NTT or RNS arithmetic;
- incorrect BFV arithmetic semantics;
- incorrect CKKS scale, level, rescale, or error behavior;
- unsafe key switching, relinearization, or Galois operations;
- incomplete bootstrap transforms presented as bootstrap completion;
- public API ownership ambiguity;
- retired public surfaces reintroduced through compatibility shortcuts;
- partial subsystem closure presented as full FHE acceptance.

Required defenses include:

- strict parameter validation;
- context and key binding;
- negative tests for invalid and adversarial inputs;
- encrypted-path tests where encrypted behavior is claimed;
- canonical public surface ownership;
- surface guard tests;
- failure reports with exact blockers;
- conservative final flags;
- no production-ready claim without the relevant gates.

FHE acceptance must remain scoped to the exact gate that established it.

## 14. BFV-specific threats

BFV-specific threats include:

- incorrect plaintext modulus handling;
- incorrect ciphertext modulus behavior;
- invalid batching assumptions;
- incorrect encode or decode behavior;
- multiply-plain or multiply semantics that diverge from BFV requirements;
- improper noise or budget interpretation;
- key lifecycle mismatch;
- parameter sets accepted outside intended constraints;
- BFV acceptance presented as full FHE or CKKS acceptance.

Required defenses include:

- BFV parameter validation;
- BFV encode/decode tests;
- encryption and decryption roundtrip tests;
- arithmetic semantic tests;
- negative tests;
- key lifecycle tests;
- stress tests across accepted parameter ranges;
- claim discipline separating BFV closure from broader FHE claims.

## 15. CKKS-specific threats

CKKS-specific threats include:

- scale mismatch;
- level mismatch;
- unsafe rescale behavior;
- incorrect encoder semantics;
- incorrect approximate arithmetic claims;
- invalid rotation or Galois key behavior;
- incorrect key switching behavior;
- inaccurate error accounting;
- partial transform closure presented as full CKKS acceptance;
- CKKS acceptance presented as bootstrap completion.

Required defenses include:

- scale and level invariant tests;
- encode/decode tests;
- encrypted arithmetic tests;
- rescale tests;
- rotation and key-switching tests;
- error-bound evidence;
- negative tests for invalid scale or level transitions;
- claim discipline separating CKKS closure from bootstrap closure.

## 16. Bootstrap threats

Bootstrap threats include:

- partial bootstrap stages presented as full bootstrap completion;
- metadata-only bootstrap proof;
- plaintext reference bootstrap presented as encrypted bootstrap;
- unsafe scale or modulus transition;
- incomplete CoeffToSlot or SlotToCoeff closure;
- incomplete EvalMod closure;
- unsafe rotation composition;
- unresolved BSGS or hoisted decomposition blockers;
- public API output truth overclaimed from smoke tests;
- runtime materialization not proven but presented as complete.

Required defenses include:

- stage-specific tests;
- encrypted-path evidence for encrypted claims;
- scale and level guards;
- rotation and key-switch tests;
- public API output truth gates;
- full graph materialization before full graph claims;
- final flags that remain conservative until all required gates pass.

## 17. Cryptographic primitive threats

Cryptographic primitives outside FHE face threats such as:

- invalid key length acceptance;
- invalid nonce or IV behavior;
- state reuse;
- weak randomness;
- incorrect domain separation;
- unsafe truncation;
- malformed input handling;
- timing-sensitive behavior where relevant;
- test vector success overclaimed as full hardening;
- documentation claims beyond validated scope.

Required defenses include:

- strict input validation;
- negative tests;
- known-answer tests where appropriate;
- misuse tests where applicable;
- lifecycle tests for keys or state;
- conservative documentation and claim boundaries.

## 18. Randomness and seed threats

Randomness and seed threats include:

- predictable seed material;
- missing domain separation;
- accidental seed reuse;
- insufficient entropy admission;
- test seeds used as production seeds;
- PRNG state leakage;
- unclear ownership of randomness sources;
- deterministic test behavior mistaken for runtime security.

Required defenses include:

- explicit randomness ownership;
- domain-separated seed derivation where applicable;
- test-only seed isolation;
- rejection of invalid seed states;
- clear distinction between deterministic tests and production randomness;
- review of randomness-sensitive code.

## 19. Key lifecycle threats

Key lifecycle threats include:

- stale key use;
- context mismatch;
- unauthorized key reuse;
- missing key retirement;
- secret material copied into diagnostics;
- test keys used as production authority;
- key switching material used outside intended scope;
- rotation keys accepted for the wrong context;
- lifecycle failure presented as valid execution.

Required defenses include:

- explicit key-context binding;
- lifecycle state tests;
- negative tests for stale or mismatched keys;
- secure cleanup where required;
- no secret material in logs;
- fail-closed behavior on missing or invalid key authority.

## 20. Memory and bounds threats

Memory and bounds threats include:

- out-of-bounds reads or writes;
- overlapping memory errors;
- aliasing assumptions;
- lifetime violations;
- use-after-free;
- uninitialized memory use;
- unsafe alignment assumptions;
- integer overflow in size calculations;
- sanitizer failures hidden behind partial runs.

Required defenses include:

- checked arithmetic where appropriate;
- bounds tests;
- overlap tests;
- sanitizer runtime execution;
- fail-closed invalid span behavior;
- no sanitizer-clean claim without runtime evidence.

## 21. Build and CI threats

Build and CI threats include:

- configure success presented as build success;
- build success presented as test success;
- sanitizer configuration presented as sanitizer clean;
- target-only execution presented as full graph truth;
- missing executables ignored;
- CTest labels weakened;
- failing tests removed or excluded;
- CI jobs skipped without disclosure;
- build graph materialization not completed;
- generated artifacts overclaiming acceptance.

Required defenses include:

- materialization checks;
- labelled CTest validation;
- full graph evidence where claimed;
- sanitizer runtime logs where sanitizer truth is claimed;
- exact blocker reporting;
- CI logs preserved with scope;
- no production claim from partial CI success.

## 22. Evidence threats

Evidence threats include:

- stale evidence presented as current;
- evidence from a different source tree presented as applicable;
- historical logs used instead of required live replay;
- final flags missing or inconsistent;
- evidence bundle incomplete;
- failure logs omitted;
- exact blockers removed from summaries;
- partial success summarized as full success;
- metadata-only evidence presented as runtime evidence.

Required defenses include:

- source tree identification;
- patch or diff records;
- final flags;
- complete logs for claimed gates;
- explicit failure classification;
- exact blockers;
- review of evidence scope;
- refusal to elevate claims from incomplete evidence.

## 23. Documentation and claim threats

Documentation threats include:

- planned behavior described as implemented;
- implemented behavior described as validated;
- validated subsystem behavior described as global acceptance;
- security hardening claimed without tests;
- production readiness claimed without gates;
- sanitizer clean claimed without sanitizer runtime;
- bootstrap complete claimed from partial bootstrap progress;
- FHE accepted claimed from BFV-only or CKKS-only progress;
- old claims remaining after architecture changes.

Required defenses include:

- claim discipline;
- conservative wording;
- current status boundaries;
- explicit distinction between planned, implemented, validated, blocked, and accepted states;
- review of documentation changes that affect claims;
- no release note overclaim.

Documentation is part of the threat surface because it can mislead users, reviewers, contributors, and funders.

## 24. Supply-chain and dependency threats

XPScerpto’s architecture aims to reduce external dependency risk, but the threat is not eliminated.

Supply-chain threats include:

- compromised tools;
- compiler or linker bugs;
- malicious build scripts;
- unreviewed generated code;
- stale vendored assets;
- CI environment compromise;
- artifact substitution;
- package confusion.

Required defenses may include:

- reproducible or reviewable build steps where practical;
- source tree hashing;
- patch manifests;
- evidence bundle integrity;
- review of generated files;
- minimal dependency surfaces;
- explicit artifact provenance.

This document does not claim full supply-chain immunity.

## 25. Side-channel and leakage threats

Side-channel threats may include timing leakage, memory access patterns, cache behavior, branch behavior, microarchitectural effects, diagnostic leakage, or logging leakage.

XPScerpto should treat side channels conservatively, especially in cryptographic code.

Required posture:

- do not claim side-channel resistance unless specifically validated;
- avoid secret-dependent diagnostics;
- avoid secret material in logs;
- review cryptographic code for obvious secret-dependent behavior;
- preserve constant-time or leakage-resistant behavior where it is part of a claimed primitive;
- document limits clearly.

Side-channel resistance is a claim that requires specific evidence.

## 26. Denial-of-service threats

Denial-of-service threats include:

- adversarial inputs that trigger excessive allocation;
- expensive parameter sets accepted without bounds;
- unbounded loops;
- pathological graph materialization;
- repeated failed provider admission;
- malformed cryptographic objects that trigger expensive processing;
- test or tool behavior that can consume excessive resources.

Required defenses include:

- input limits;
- parameter validation;
- resource accounting where appropriate;
- fail-closed invalid requests;
- tests for adversarial sizes;
- exact blockers for resource exhaustion.

A fail-closed denial is preferable to uncontrolled resource consumption.

## 27. Release and distribution threats

Release threats include:

- publishing artifacts without matching source evidence;
- release notes overclaiming validation;
- shipping old public surfaces as current;
- missing final flags;
- missing sanitizer truth;
- packaging test-only files as production authority;
- distributing a tree that differs from the reviewed tree;
- using acceptance language for experimental builds.

Required defenses include:

- source tree identifiers;
- patch manifests;
- evidence bundles;
- final flags;
- release claim review;
- clear status language;
- no production-ready claim without gates.

## 28. Threat-to-validation mapping

The following validation families should map to this threat model:

| Threat area | Expected validation |
|---|---|
| Authority bypass | authority-routing tests, import scans, review gates |
| Platform provider confusion | provider tests, platform admission tests |
| Hardware authority confusion | hardware evidence tests, routing tests |
| SIMD unauthorized dispatch | SIMD authority tests, dispatch tests, bounds tests |
| FHE cryptographic misuse | parameter, context, key lifecycle, encrypted-path tests |
| Bootstrap overclaim | stage gates, public API truth gates, full graph validation |
| Boundary violation | module import scans, CODEOWNERS, CI checks |
| Evidence confusion | final flags, source tree identifiers, evidence bundle review |
| Sanitizer overclaim | sanitizer runtime logs |
| Documentation overclaim | documentation review and claim discipline |

This table is not exhaustive. It defines required direction, not complete closure.

## 29. Fail-closed expectations

When a threat cannot be fully mitigated, the system must fail closed or report the limitation clearly.

Acceptable incomplete outcomes include:

```text
NO_WITH_REASON
PATH_B_FAIL_CLOSED
EXACT_BLOCKER=...
```

A failure may be acceptable when it is precise, honest, and preserves the claim boundary.

A success is unacceptable when it hides missing authority, incomplete validation, missing evidence, or unresolved blockers.

## 30. Review checklist

A security or architecture review should ask:

- What asset is affected?
- What trust boundary is crossed?
- What authority is required?
- Where does that authority come from?
- What evidence supports the claim?
- What happens when evidence is missing?
- Does the change introduce local probing?
- Does the change weaken module boundaries?
- Does the change touch cryptographic behavior?
- Does the change affect keys, seeds, parameters, or contexts?
- Does the change weaken tests or sanitizer coverage?
- Does the change alter CI or evidence behavior?
- Does documentation claim more than evidence proves?
- If incomplete, is the exact blocker preserved?

A change that cannot answer these questions is not ready for acceptance.

## 31. Current claim boundaries

This threat model does not elevate project status.

Unless the relevant acceptance gates prove otherwise, conservative global boundaries remain in force:

```text
FHE_PRODUCTION_READY = NO
FHE_BOOTSTRAP_COMPLETE = NO
FULL_SANITIZER_CTEST_CLAIMED = NO
```

Threat modeling is not validation. It defines what validation must address.

## 32. Final rule

The final threat-model rule is:

```text
a bypass is a threat
a hidden fallback is a threat
a false claim is a threat
missing evidence is not success
```

XPScerpto is secure only to the extent that its implementation, authority path, module boundaries, tests, evidence, documentation, and claims remain aligned.
