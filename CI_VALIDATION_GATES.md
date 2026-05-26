# XPScerpto CI Validation Gates

**Status:** normative validation-gate policy  
**Scope:** repository-wide CI gates, local replay expectations, configure/build/test separation, full graph validation, sanitizer runtime truth, module-boundary enforcement, authority validation, cryptographic validation, evidence requirements, failure reporting, and claim discipline

XPScerpto is a governed sovereign cryptographic infrastructure platform. Its CI and validation gates exist to prove claims, reject overclaims, preserve exact blockers, and prevent partial execution from being presented as full acceptance.

This document defines the validation gates that should be used to evaluate source changes, protected surfaces, security-sensitive changes, release candidates, and acceptance claims.

This document is not a validation report, security proof, production-readiness claim, or acceptance artifact. It defines what must be proven before such claims may be made.

## 1. Purpose

The purpose of this document is to make XPScerpto validation explicit, repeatable, and claim-disciplined.

A validation gate exists to answer:

- what claim is being made;
- what evidence is required to support it;
- what commands, jobs, or checks must run;
- what logs or artifacts must be preserved;
- what failure mode is acceptable;
- what result must remain blocked if validation is incomplete.

XPScerpto must not accept a claim because it sounds plausible, because a narrow target passed, because a historical log exists, or because a developer intended the change to be safe.

Claims must be proven by the relevant gate.

## 2. Validation principle

The central validation principle is:

```text
configure success is not build success
build success is not test success
test success is not sanitizer truth
target success is not full graph truth
subsystem success is not repository-wide acceptance
```

Each validation level must remain distinct.

A higher-level claim may depend on lower-level success, but lower-level success does not automatically prove the higher-level claim.

## 3. Gate vocabulary

This document uses the following terms.

### Gate

A gate is a defined validation requirement for a specific claim.

A gate may be implemented by CI, local replay, CTest, sanitizer runtime, static analysis, import scans, evidence checks, or reviewed release procedures.

### Claim

A claim is any statement that a behavior, subsystem, build, test suite, sanitizer run, security property, or release state has been accepted.

Examples include:

- build passed;
- tests passed;
- sanitizer clean;
- full graph passed;
- platform accepted;
- SIMD accepted;
- cryptographic domain accepted;
- FHE accepted;
- bootstrap complete;
- production-ready.

### Evidence

Evidence is the material that proves or rejects a claim.

Evidence may include:

- source tree identifiers;
- build logs;
- CTest logs;
- sanitizer runtime logs;
- CI logs;
- final flags;
- evidence bundles;
- failure reports;
- exact blockers;
- patch records;
- review records.

### Exact blocker

An exact blocker identifies why a gate did not pass.

A blocker should be precise enough that a reviewer knows what failed, where it failed, and what claim remains unavailable.

### Acceptance

Acceptance means the relevant gate passed within its defined scope.

Acceptance must not be expanded beyond the scope of the gate.

## 4. Claim-to-gate mapping

Every claim must map to a gate.

| Claim | Minimum required gate |
|---|---|
| Configure succeeded | Configure gate |
| Build succeeded | Configure gate + build gate |
| A target passed | Configure gate + build gate + target test gate |
| Labelled tests passed | Configure gate + build gate + labelled CTest gate |
| Full graph passed | Configure gate + full build materialization + full CTest graph gate |
| Sanitizer clean | Sanitizer build + sanitizer runtime gate + preserved sanitizer logs |
| Module boundaries respected | Boundary scan gate + review of protected surfaces |
| Authority preserved | Authority validation gate + routing/boundary tests |
| Security-sensitive change accepted | Security gate + relevant test evidence + review |
| Cryptographic subsystem accepted | Domain-specific cryptographic gate + negative tests + evidence |
| FHE accepted | FHE gate suite within exact scope |
| Bootstrap complete | Bootstrap completion gate with all required stages and evidence |
| Production-ready | Explicit production gate, full required evidence, and reviewed final flags |

No claim may be made without its gate.

## 5. Configure gate

The configure gate proves that the build system can generate the required build graph for the requested configuration.

The configure gate may include:

- CMake configure;
- generator selection;
- toolchain detection;
- module mode selection;
- feature checks;
- build option validation;
- generated configuration files.

Configure success must preserve:

- selected compiler identity;
- generator identity;
- build type;
- important feature flags;
- sanitizer flags, if configured;
- module mode, if relevant;
- configure logs.

Configure success does not prove that anything built, linked, or ran.

Forbidden overclaims:

- describing configure success as build success;
- describing configure success as CTest success;
- describing sanitizer configuration as sanitizer clean;
- describing generated targets as materialized executables.

If configure fails, the result must preserve the exact blocker.

## 6. Build gate

The build gate proves that required targets were compiled and linked for the requested configuration.

The build gate may include:

- full repository build;
- labelled target build;
- protected subsystem build;
- release build;
- debug build;
- sanitizer build;
- tool build;
- test executable build.

Build evidence should identify:

- build directory;
- configuration;
- compiler;
- generator;
- target set;
- failed target, if any;
- first compiler or linker error, if any;
- whether all required test executables were materialized.

Build success does not prove test success.

A build that stops before required targets are materialized must fail closed.

Forbidden overclaims:

- describing partial build progress as full build success;
- describing target-only build success as full graph materialization;
- describing a generated target as an executable if it was not built;
- hiding missing executables behind partial CTest output.

## 7. Test gate

The test gate proves that defined tests executed and passed within a declared scope.

The test gate may include:

- focused unit tests;
- subsystem test labels;
- authority-routing tests;
- boundary tests;
- cryptographic correctness tests;
- negative security tests;
- regression tests;
- full labelled CTest runs;
- full CTest graph runs.

Test evidence should identify:

- command executed;
- test directory;
- label or target scope;
- total tests;
- passed tests;
- failed tests;
- skipped tests;
- disabled tests;
- first failure;
- failure class;
- output logs.

A test gate must not silently drop tests.

Forbidden overclaims:

- describing focused tests as full graph tests;
- describing partial labelled tests as all tests;
- hiding skipped or missing tests;
- weakening assertions to create a pass;
- deleting failing tests to create acceptance;
- presenting test-only behavior as production behavior.

## 8. Full graph gate

The full graph gate proves that the required build and test graph was materially available and executed within the claimed scope.

A full graph claim requires more than a passing subset.

A full graph gate should establish:

- configure completed;
- required build graph materialized;
- required test executables exist;
- required CTest labels are present;
- expected tests are not missing;
- full claimed test scope executed;
- failures, skips, and omissions are recorded;
- logs are preserved.

The full graph gate must reject:

- missing executables;
- incomplete target materialization;
- label drift;
- target-only success presented as full graph success;
- tests removed without governance approval;
- partial CTest replay presented as full graph truth.

If full graph validation cannot complete, the result must identify the exact blocker.

## 9. Sanitizer runtime gate

The sanitizer runtime gate proves runtime behavior under sanitizer instrumentation.

Sanitizer configuration is not sanitizer truth.

A sanitizer runtime gate may include:

- AddressSanitizer runtime;
- UndefinedBehaviorSanitizer runtime;
- ThreadSanitizer runtime;
- memory safety focused tests;
- concurrency tests;
- subsystem sanitizer suites;
- full labelled sanitizer CTest, if required.

Sanitizer evidence should identify:

- sanitizer type;
- compiler;
- flags;
- target set;
- tests executed;
- runtime logs;
- failures;
- crashes;
- timeouts;
- first sanitizer finding;
- exact blocker if incomplete.

A sanitizer gate must not claim clean status if:

- only configure ran;
- only build ran;
- runtime did not execute;
- runtime crashed before the relevant tests;
- a sanitizer failure was observed;
- only unrelated targets ran;
- required targets were missing.

A sanitizer failure may be a valid fail-closed outcome. It is not sanitizer clean.

## 10. Module boundary gate

The module boundary gate proves that module visibility and protected surface ownership were not violated.

The gate may include:

- import graph scans;
- forbidden include scans;
- protected surface ownership checks;
- public API ownership checks;
- test-only import checks;
- tooling-only import checks;
- retired surface scans;
- alias, shim, and adapter scans;
- CODEOWNERS routing checks.

The boundary gate should reject:

- domain imports of platform internals;
- bypass of `platform.api`;
- local hardware authority inside domains;
- unauthorized SIMD dispatch authority;
- test-only helper imports into production;
- duplicate public API owners;
- old public surfaces reintroduced without review;
- compatibility wrappers that hide ownership conflicts.

Boundary validation must preserve exact violations.

A boundary scan that did not run must not be described as boundary clean.

## 11. Authority validation gate

The authority validation gate proves that execution decisions follow the approved authority chain.

The expected chain is:

```text
platform governance
→ platform.api
→ hardware authority
→ routing capsule
→ domain execution
→ audit and evidence
```

Authority validation may include:

- platform admission tests;
- provider behavior tests;
- hardware evidence tests;
- routing capsule tests;
- SIMD routing tests;
- domain fail-closed tests;
- audit ledger checks;
- negative tests for unauthorized authority.

The authority gate should reject:

- local CPU probing used as domain authority;
- compiler macros used as runtime authority;
- operating-system probes used inside domains for authority;
- hidden runtime globals used as permission;
- fallback behavior presented as admitted execution;
- domain execution outside admitted scope.

Authority validation must distinguish capability from admitted capability.

## 12. Security-sensitive change gate

A security-sensitive change must pass a gate appropriate to its risk.

Security-sensitive changes include changes to:

- cryptographic behavior;
- key, seed, randomness, credential, or secret handling;
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
- sanitizer behavior;
- evidence generation;
- final flags;
- release notes or security claims.

The security gate should include:

- focused review;
- threat impact analysis;
- boundary impact analysis;
- negative tests where appropriate;
- runtime tests where appropriate;
- sanitizer runtime where memory or undefined behavior risk exists;
- evidence and claim-boundary review.

A security-sensitive change must not be accepted through convenience-only validation.

## 13. Cryptographic domain gates

Cryptographic domain gates prove claims for a specific cryptographic subsystem.

These gates may apply to domains such as:

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

A cryptographic gate should define:

- accepted parameter sets;
- key or seed lifecycle behavior;
- input and output bounds;
- correctness tests;
- negative tests;
- misuse tests where applicable;
- deterministic tests where applicable;
- randomness or domain-separation tests where applicable;
- sanitizer runtime requirements where applicable;
- exact scope of the claim.

Cryptographic gates must reject:

- invalid parameters accepted silently;
- test vectors presented as full hardening;
- metadata-only checks presented as runtime proof;
- one subsystem’s acceptance presented as another subsystem’s acceptance;
- documentation claims beyond the validated scope.

## 14. FHE gate family

FHE is one high-risk cryptographic family and may require additional gates.

This section does not make CI validation FHE-only. It records additional FHE-specific gates because FHE claims often depend on many interacting stages.

FHE gate families may include:

- parameter and context gates;
- BFV correctness gates;
- CKKS correctness gates;
- encoder and decoder gates;
- encryption and decryption gates;
- add, sub, multiply, relinearize, and rescale gates;
- NTT and RNS gates;
- key lifecycle gates;
- Galois and rotation gates;
- key-switching gates;
- bootstrap stage gates;
- public API ownership gates;
- surface guard gates;
- full FHE graph gates;
- sanitizer runtime gates.

FHE gates must preserve claim scope.

BFV acceptance does not imply CKKS acceptance.

CKKS acceptance does not imply bootstrap completion.

Bootstrap progress does not imply production readiness.

FHE subsystem success does not imply repository-wide production readiness.

## 15. Bootstrap completion gate

A bootstrap completion claim is a high-level claim and requires its own gate.

The gate must define:

- which bootstrap path is in scope;
- which stages are required;
- whether execution is encrypted-path, plaintext-reference, diagnostic, or test-only;
- scale, level, modulus, and error constraints;
- key-switching and rotation requirements;
- public API output truth requirements;
- runtime materialization requirements;
- full graph requirements;
- sanitizer requirements, if claimed;
- final flags.

The bootstrap completion gate must reject:

- partial stage closure presented as full completion;
- diagnostic path success presented as production behavior;
- plaintext reference behavior presented as encrypted-path truth;
- metadata-only proof;
- missing public API output truth;
- missing runtime materialization;
- unresolved exact blockers.

## 16. Platform and provider gates

Platform and provider gates prove platform admission behavior.

These gates may include:

- provider registration tests;
- unsupported provider fail-closed tests;
- platform API tests;
- provider isolation tests;
- platform runtime guard tests;
- platform import boundary tests;
- platform evidence tests.

The platform gate should reject:

- provider internals imported by domains;
- unsupported provider behavior treated as active support;
- foreign stubs treated as native execution;
- direct platform runtime access outside admitted surfaces;
- platform API guards disabled without disclosure.

## 17. Hardware and routing gates

Hardware and routing gates prove that hardware evidence and routing decisions remain governed.

These gates may include:

- hardware evidence collection tests;
- topology tests;
- routing decision tests;
- degraded-environment tests;
- unsupported capability tests;
- dispatch eligibility tests;
- audit evidence checks.

The hardware and routing gate should reject:

- local domain hardware authority;
- conflicting hardware evidence presented as success;
- dispatch without admitted routing;
- benchmark-based authority;
- hidden hardware caches outside the authority chain.

## 18. SIMD validation gates

SIMD gates prove correctness, authority routing, and performance claims within the admitted scope.

SIMD gates may include:

- scalar fallback correctness tests;
- vector path correctness tests;
- overlap and bounds tests;
- routing authority tests;
- unauthorized dispatch rejection tests;
- performance baselines;
- benchmark scope reports.

SIMD performance claims must identify:

- hardware authority status;
- dispatch eligibility;
- benchmark conditions;
- compared paths;
- correctness status;
- authority status.

A SIMD benchmark win does not prove authority.

A SIMD authority pass does not prove global performance dominance.

## 19. Evidence integrity gate

The evidence integrity gate proves that validation artifacts match the source and claim.

Evidence integrity may require:

- source tree identifiers;
- patch or diff records;
- build directory records;
- command records;
- logs;
- final flags;
- evidence bundles;
- checksum manifests;
- exact blocker reports.

The gate should reject:

- missing final flags when final flags are required;
- logs from a different source tree;
- stale packaged evidence presented as live replay;
- incomplete evidence bundles;
- summaries that omit failures;
- checksums that do not match artifacts;
- failure reports that hide exact blockers.

Evidence is accepted only within its scope.

## 20. Documentation claim gate

Documentation changes may require validation when they affect claims.

Documentation must distinguish:

- planned behavior;
- implemented behavior;
- validated behavior;
- blocked behavior;
- accepted behavior;
- production-ready behavior.

The documentation claim gate should reject:

- planned behavior presented as accepted;
- subsystem success presented as repository-wide success;
- sanitizer clean claims without runtime logs;
- production-ready claims without production gates;
- release notes that exceed evidence;
- documentation that hides exact blockers.

Documentation is not exempt from CI discipline when it affects project claims.

## 21. Release candidate gate

A release candidate gate must be explicit about scope.

A release candidate may require:

- clean configure;
- clean build;
- expected target materialization;
- labelled CTest truth;
- sanitizer runtime truth, if claimed;
- module boundary scans;
- authority gates;
- security-sensitive gates;
- evidence integrity checks;
- final flags;
- release notes claim review.

A release candidate must not claim more than its evidence supports.

If release validation is incomplete, the release must either remain blocked or clearly identify the exact limitation.

## 22. Production-readiness gate

Production readiness is the highest claim and must not be inferred from ordinary validation.

A production-readiness gate must define:

- product scope;
- supported platforms;
- supported configurations;
- cryptographic domains in scope;
- excluded domains;
- test scope;
- full graph requirements;
- sanitizer requirements;
- threat model coverage;
- unresolved blockers;
- final flags;
- review authority;
- release evidence.

The production gate must reject:

- local-only success;
- target-only success;
- partial subsystem success;
- historical evidence without required live replay;
- sanitizer configuration without runtime;
- documentation-only readiness;
- unresolved critical or high blockers;
- ambiguous public API ownership;
- missing claim boundaries.

Production-ready must never be implied. It must be explicitly proven.

## 23. Local replay expectations

Local replay may support CI evidence when CI is not available, incomplete, or being debugged.

A local replay should record:

- source tree path or identifier;
- command sequence;
- environment summary;
- compiler and generator;
- build configuration;
- test scope;
- logs;
- failures;
- exact blocker;
- whether the result is acceptance or only diagnostic.

Local replay must not overclaim.

A local replay of one target is not full CI.

A local replay may be valuable evidence, but its scope must be stated precisely.

## 24. Failure reporting

A failed gate must fail closed.

Acceptable incomplete outcomes include:

```text
NO_WITH_REASON
PATH_B_FAIL_CLOSED
EXACT_BLOCKER=...
```

A failure report should include:

- gate name;
- source tree;
- command or job;
- expected evidence;
- observed evidence;
- first failure;
- failure class;
- exact blocker;
- claims that remain unavailable;
- whether any artifacts were produced.

Failure reporting must preserve the blocker.

A vague failure that hides the real blocker weakens validation.

## 25. Forbidden CI overclaims

The following CI overclaims are forbidden:

- configure-only success reported as build success;
- build-only success reported as test success;
- sanitizer build reported as sanitizer clean;
- target-only pass reported as full graph pass;
- partial label pass reported as full labelled pass;
- historical logs reported as live replay;
- missing executables ignored;
- skipped tests hidden;
- disabled tests hidden;
- failing tests removed to create acceptance;
- final flags edited without gate evidence;
- documentation updated to claim a gate that did not pass;
- one subsystem’s success used as repository-wide acceptance.

Any such overclaim must be treated as a validation failure.

## 26. Gate result language

Gate results should use precise language.

Preferred result language:

```text
PASS
FAIL
NO_WITH_REASON
PATH_A_ACCEPTED
PATH_B_FAIL_CLOSED
BLOCKED
NOT_RUN
NOT_CLAIMED
EXACT_BLOCKER=...
```

Avoid vague language such as:

```text
mostly done
probably fine
works locally
seems clean
nearly complete
should be accepted
safe enough
```

CI language must be precise because imprecise language becomes false confidence.

## 27. Review checklist for validation changes

A validation or CI change should be reviewed with the following questions:

- What claim does this gate prove?
- What evidence does it require?
- What evidence does it produce?
- What happens when it fails?
- Does it preserve exact blockers?
- Does it hide skipped tests?
- Does it weaken labels?
- Does it change sanitizer meaning?
- Does it alter final flags?
- Does it affect protected surfaces?
- Does it change release claims?
- Does it make overclaim easier?
- Is the gate repository-wide or subsystem-specific?
- Is the scope documented clearly?

A validation change that makes claims easier without preserving evidence is not acceptable.

## 28. Relationship to other documents

This document should be read after:

```text
START_HERE.md
GOVERNANCE.md
ENGINEERING_PRINCIPLES.md
AUTHORITY_MODEL.md
ARCHITECTURE.md
MODULE_BOUNDARIES.md
THREAT_MODEL.md
SECURITY.md
```

It should be read before:

```text
REVIEW_POLICY.md
CONTRIBUTING.md
CODEOWNERS
```

`GOVERNANCE.md` defines claim discipline.

`AUTHORITY_MODEL.md` defines the authority chain.

`MODULE_BOUNDARIES.md` defines visibility and ownership boundaries.

`THREAT_MODEL.md` defines what validation must defend against.

`SECURITY.md` defines how security reports and security-sensitive changes are handled.

`CI_VALIDATION_GATES.md` defines how claims are proven or rejected.

## 29. Current claim boundaries

This document does not elevate project status.

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

A validation-gate policy is not validation evidence. It defines what evidence must exist before claims may be accepted.

## 30. Final validation rule

The final validation rule is:

```text
no gate, no claim
no runtime, no sanitizer truth
no full graph, no full graph truth
no evidence, no acceptance
no exact blocker, no honest failure
```

XPScerpto validation succeeds only when the command executed, the evidence produced, the blocker reported, and the claim made all describe the same truth.
