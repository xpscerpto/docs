# XPScerpto Module Boundaries

**Status:** normative module-boundary policy  
**Scope:** repository-wide import boundaries, public and internal surfaces, protected ownership, domain visibility, test-only access, tooling access, documentation discipline, CI enforcement expectations, and boundary violation handling

XPScerpto is a governed sovereign cryptographic infrastructure platform. Its module boundaries are security, authority, ownership, and review boundaries. They are not merely compilation details.

This document defines how repository modules, directories, public surfaces, internal surfaces, tests, tools, and documentation are expected to relate to one another.

This document is not a validation report, security proof, production-readiness claim, or acceptance artifact. It defines the boundary policy that implementation, tests, reviews, and validation gates must respect.

## 1. Purpose

The purpose of this document is to make XPScerpto’s module boundaries explicit and enforceable.

A module boundary exists to answer:

- who owns this surface;
- who may import or include it;
- what authority it exposes;
- what state remains internal;
- what tests protect it;
- what evidence is required when it changes;
- what behavior is forbidden across the boundary.

A boundary violation is not a minor style issue. It may weaken authority, hide ownership conflicts, bypass protected surfaces, invalidate tests, or create false acceptance claims.

## 2. Boundary principle

The central boundary principle is:

```text
public surfaces are admitted intentionally
internal surfaces remain internal
domain code consumes authority; it does not create it
tests may inspect behavior; they must not redefine production boundaries
```

A module may depend only on surfaces that are intentionally exposed for its layer and role.

An import that compiles is not automatically an allowed import.

A successful build does not prove that a boundary was respected.

A convenient shortcut is not acceptable if it bypasses ownership, authority, validation, or review.

## 3. Relationship to the authority model

Module boundaries implement the authority model in the source tree.

The expected authority flow remains:

```text
platform governance
→ platform.api
→ hardware authority
→ routing capsule
→ domain execution
→ audit and evidence
```

The module graph must not invert this flow.

Domain modules must not import private platform internals to manufacture authority. Performance modules must not import hidden hardware probes to bypass routing. Cryptographic modules must not reach into test-only helpers to simulate production behavior. Documentation must not describe a boundary as validated unless the relevant evidence exists.

## 4. Surface categories

XPScerpto uses the following conceptual surface categories.

### 4.1 Public surface

A public surface is intentionally exposed for consumption outside its implementation family.

A public surface must have:

- a clear owner;
- a narrow purpose;
- stable import expectations;
- tests or validation appropriate to its risk;
- documentation that does not overclaim its status.

A public surface is not automatically production-ready. Public means intentionally exposed, not fully accepted.

### 4.2 Protected surface

A protected surface is sensitive because it controls authority, cryptographic behavior, platform behavior, validation, evidence, CI, or public claims.

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

A protected surface may be public, internal, or test-facing. The protection level is determined by risk, not only visibility.

### 4.3 Internal surface

An internal surface is implementation detail for its family or layer.

Internal surfaces must not be imported by unrelated domains for convenience. If another family needs behavior from an internal surface, the correct repair is to expose a reviewed public or admitted surface, not to import internals directly.

### 4.4 Test-only surface

A test-only surface exists to validate behavior.

It must not become a production dependency. Test helpers, fixtures, adapters, and mock paths must remain clearly separated from production module authority.

A test-only adapter may be useful for migration or regression coverage, but it must not be presented as a production surface.

### 4.5 Tooling surface

A tooling surface supports validation, scans, evidence creation, graph analysis, reporting, or CI automation.

Tools may inspect the repository. Tools do not automatically define runtime authority or production acceptance.

### 4.6 Documentation surface

A documentation surface explains rules, architecture, validation, contribution, or claims.

Documentation can define policy, but it must not claim implementation success without evidence.

## 5. Repository boundary map

The following map describes the intended boundary posture of major repository families. It is a policy map, not a production-readiness claim.

| Family | Boundary role | Expected consumers | Boundary posture |
|---|---|---|---|
| `base` | Foundational types and checked primitives | Most lower and domain layers | Narrow, stable, non-authority-producing |
| `platform` | Platform root, providers, runtime admission | `platform.api`, platform-owned code | Protected internal authority owner |
| `platform.api` | Admitted platform-facing surface | Hardware, routing, domains, tools as appropriate | Public admitted authority surface |
| `hw` | Hardware evidence and decision support | Routing, SIMD, platform-aware domains | Authority producer; not bypassable |
| `simd` | Authority-routed acceleration domain | Approved callers through dispatch surfaces | Authority consumer; no local authority |
| `FHE` | Homomorphic encryption domain and canonical surfaces | Approved crypto API users and tests | Highly protected cryptographic core |
| `hash`, `kdf`, `mac`, `aead`, `aes` | Symmetric cryptographic domains | Approved crypto consumers | Security-sensitive domain surfaces |
| `pqc`, `ed25519`, `x25519`, `rsa` | Asymmetric and post-quantum domains | Approved crypto consumers | Security-sensitive domain surfaces |
| `number`, `atomic`, `memory`, `integrity` | Support families for arithmetic, synchronization, memory, and integrity | Approved implementation layers | Support surfaces; no authority bypass |
| `audit` | Evidence and attestation behavior | Validation, tooling, selected runtime paths | Evidence surface |
| `super` | Higher-level orchestration and integration | Approved high-level users | Must not weaken lower ownership |
| `tests` | Validation and regression | Test build only | Test-only; not production authority |
| `tools` | Scans, replay, evidence, CI support | Developers and CI | Tooling-only; not runtime authority |
| `docs` | Normative and explanatory documentation | Humans and review processes | Claim-disciplined documentation |
| `.github` | CI and repository automation | CI and repository governance | Validation and review enforcement |

## 6. Base layer boundaries

The base layer provides foundational primitives.

Allowed posture:

- may be consumed by higher layers;
- should remain stable and minimal;
- should avoid dependency on domain layers;
- should avoid hidden platform authority;
- should avoid importing cryptographic domains;
- should avoid owning policy that belongs to platform, governance, or domain layers.

The base layer must not become a hidden backdoor for domain policy, platform admission, hardware probing, or runtime shortcuts.

A base primitive should be broadly reusable because it is narrow, not because it has accumulated cross-domain authority.

## 7. Platform boundaries

The platform layer owns platform admission and provider behavior.

Platform internals must remain inside the platform family unless explicitly exposed through an admitted surface.

Forbidden patterns include:

- domain code importing platform internals for convenience;
- domain code relying on provider implementation details;
- hardware or cryptographic code bypassing `platform.api`;
- test helpers becoming hidden platform admission paths;
- provider-specific behavior leaking into unrelated domains;
- foreign or unsupported provider paths being treated as admitted runtime behavior.

The platform layer may depend on foundational layers, but it must not depend on domains in a way that reverses authority flow.

## 8. `platform.api` boundaries

`platform.api` is the admitted platform-facing surface.

It exists to keep platform authority narrow, reviewable, and consumable without exposing platform internals.

Allowed posture:

- may be consumed by hardware authority code;
- may be consumed by routing and dispatch code;
- may be consumed by domains where platform authority is required;
- may be consumed by tests and tools for validation;
- must remain narrower than the full platform implementation.

Forbidden patterns include:

- treating `platform.api` as a dumping ground for internal platform details;
- exposing provider internals through public API for convenience;
- adding broad accessors that bypass reviewable authority decisions;
- duplicating platform authority in parallel public surfaces.

`platform.api` should make the correct path easier than the shortcut.

## 9. Hardware authority boundaries

The hardware family owns hardware evidence and decision support.

Hardware authority must not be duplicated in domains.

Allowed posture:

- may consume admitted platform information;
- may produce evidence-backed hardware decisions;
- may provide routing inputs to dispatch layers;
- may expose constrained decision structures to domains;
- may be validated by hardware and routing tests.

Forbidden patterns include:

- SIMD code performing its own authority probes;
- cryptographic code performing hardware eligibility probes directly;
- domains reading operating-system hardware files for authority;
- domains using compiler CPU-support builtins as authority;
- domains inferring authority from compiler target flags;
- hidden hardware caches outside the approved authority path.

Hardware evidence is not domain authority until it is admitted and routed through the approved path.

## 10. Routing and dispatch boundaries

Routing and dispatch surfaces carry admitted decisions from authority layers to domain execution.

Allowed posture:

- may consume platform and hardware authority;
- may produce dispatch plans or routing capsules;
- may be consumed by domains;
- must preserve enough information for audit and review;
- must fail closed when authority is missing.

Forbidden patterns include:

- routing wrappers that merely disguise local probing;
- dispatch decisions made from hidden domain state;
- fast paths that bypass routing;
- fallback paths presented as admitted execution;
- benchmark-driven dispatch without authority admission.

Routing is a boundary, not a performance decoration.

## 11. SIMD boundaries

SIMD is an authority-routed domain.

SIMD code consumes admitted routing decisions. It must not create hardware authority.

Allowed posture:

- may consume approved SIMD routing signals;
- may consume approved dispatch plans;
- may execute scalar or vector paths inside admitted scope;
- may expose tested public dispatch APIs;
- may provide performance evidence with clear scope.

Forbidden patterns include:

- local CPUID authority in SIMD domain code;
- direct compiler CPU-support builtins as authority;
- ISA macro inference as runtime authority;
- operating-system file probing from SIMD code;
- hidden dispatch globals not tied to admitted authority;
- unreviewed fast paths selected outside routing;
- presenting unauthorized vector execution as accepted performance.

SIMD may be optimized aggressively only inside the governed authority path.

## 12. FHE boundaries

FHE is a highly protected cryptographic domain.

FHE boundaries protect cryptographic correctness, key lifecycle, parameter validity, public surface ownership, bootstrap claims, and acceptance discipline.

Allowed posture:

- expose canonical public surfaces intentionally;
- keep internal cryptographic implementation details internal;
- use tests to validate behavior without making tests production dependencies;
- preserve explicit parameter, key, context, scale, level, modulus, and rotation boundaries;
- fail closed when cryptographic authority or validation is incomplete.

Forbidden patterns include:

- importing test-only helpers into production FHE code;
- reintroducing retired public surfaces as compatibility shortcuts;
- using aliases to hide canonical surface ownership conflicts;
- presenting plaintext reference behavior as encrypted-path acceptance;
- presenting metadata-only checks as cryptographic proof;
- using secret-key material outside admitted lifecycle paths;
- claiming FHE acceptance from BFV-only or CKKS-only closure;
- claiming bootstrap completion from partial transform closure;
- bypassing surface guards through adapter layers.

FHE public API ownership must be canonical. A duplicate public owner is a boundary failure.

## 13. Cryptographic domain boundaries

Cryptographic domains outside FHE remain security-sensitive.

This includes, where present:

```text
hash
kdf
mac
aead
aes
pqc
ed25519
x25519
rsa
VeloSeal64
```

Allowed posture:

- expose narrow and intentional public APIs;
- validate inputs, keys, seeds, parameters, and output bounds;
- keep implementation details internal;
- use tests to preserve security behavior;
- avoid hidden fallback behavior;
- preserve claim scope.

Forbidden patterns include:

- using test vectors as production authority;
- accepting invalid parameters for compatibility;
- silently truncating or widening security-sensitive values;
- introducing ambiguous ownership between public surfaces;
- bypassing key or seed lifecycle rules;
- documenting hardening that was not validated.

Small cryptographic modules require the same claim discipline as large ones.

## 14. Support-family boundaries

Support families such as `number`, `atomic`, `memory`, and `integrity` provide reusable implementation support.

They must not become hidden policy or authority owners.

Allowed posture:

- provide reusable primitives;
- support domain code without importing domain policy;
- expose behavior that is clear and testable;
- preserve memory, arithmetic, and synchronization invariants.

Forbidden patterns include:

- embedding domain-specific authority decisions;
- importing protected cryptographic domains for convenience;
- hiding platform or hardware probes in support code;
- weakening memory or arithmetic checks to satisfy a caller;
- converting support utilities into undocumented global runtime state.

Support code should reduce complexity without absorbing authority that belongs elsewhere.

## 15. Audit and evidence boundaries

Audit and evidence surfaces record decisions, test outcomes, validation artifacts, and acceptance context.

Allowed posture:

- collect or format evidence;
- preserve final flags or equivalent acceptance records;
- distinguish accepted execution from fallback behavior;
- identify exact blockers;
- support replay or review where applicable.

Forbidden patterns include:

- turning evidence formatting into acceptance logic;
- treating historical evidence as live replay when live replay is required;
- hiding missing logs behind successful summaries;
- converting partial evidence into full closure;
- omitting blockers that change the meaning of a claim.

Evidence surfaces support truth. They must not manufacture it.

## 16. Tests boundaries

Tests are validation surfaces, not production authority surfaces.

Allowed posture:

- inspect protected behavior under controlled conditions;
- use fixtures, mocks, adapters, or helpers where clearly test-only;
- validate negative behavior;
- preserve regression coverage;
- assert claim boundaries;
- detect overclaims.

Forbidden patterns include:

- importing test-only helpers into production code;
- weakening tests to make a phase pass;
- deleting assertions that protect boundaries;
- replacing production behavior with a test adapter and calling it closure;
- using test-only behavior as production evidence;
- hiding a real blocker through a test rename or filter;
- relying on target-only execution for full graph claims.

A test may be adjusted when the product behavior legitimately changes, but the adjustment must preserve the original security or boundary intent unless the governance process explicitly accepts the change.

## 17. Tools boundaries

Tools support validation, analysis, evidence generation, and CI workflows.

Allowed posture:

- scan imports and module graphs;
- inspect repository structure;
- generate reports;
- replay tests or summarize logs;
- help identify exact blockers;
- support CI enforcement.

Forbidden patterns include:

- tools making production-readiness claims without acceptance gates;
- tools treating missing evidence as success;
- tools rewriting protected files without review;
- tools masking boundary violations;
- tools replacing runtime validation with static metadata when runtime validation is required.

Tools may support gates. They are not gates by themselves unless the governance policy defines them as such.

## 18. Documentation boundaries

Documentation is a governance and review surface.

Allowed posture:

- define policy;
- explain architecture;
- describe accepted behavior with evidence;
- describe planned behavior as planned;
- describe blocked behavior as blocked;
- preserve claim boundaries.

Forbidden patterns include:

- documenting a planned state as accepted;
- claiming production readiness without acceptance evidence;
- claiming sanitizer clean without sanitizer runtime;
- claiming full graph truth from partial execution;
- presenting examples as implemented behavior when they are not;
- hiding exact blockers behind aspirational language.

Documentation must tell the same truth as implementation and evidence.

## 19. Build and CI boundaries

Build and CI files are protected validation surfaces.

Allowed posture:

- materialize required targets;
- enforce module and import policy;
- define validation gates;
- run tests and sanitizer jobs;
- preserve logs and failure information;
- prevent accidental overclaim.

Forbidden patterns include:

- excluding failing tests without governance approval;
- weakening labels to reduce required coverage;
- generating targets without testing them and claiming acceptance;
- treating configure success as build success;
- treating build success as test success;
- treating sanitizer configuration as sanitizer clean;
- hiding materialization failures behind partial CTest output.

Build and CI changes must be reviewed as architecture-sensitive changes.

## 20. Public API ownership rules

Every public API surface must have a clear canonical owner.

A public API ownership record should answer:

- what surface is public;
- who owns it;
- what internal implementation it may reach;
- what consumers may import it;
- what tests protect it;
- what evidence is required for changes;
- whether older surfaces are retired, demoted, or still active.

Forbidden ownership patterns include:

- duplicate canonical owners;
- alias masking;
- forwarding files that hide ownership;
- compatibility wrappers that bypass governance;
- old public surfaces reintroduced without review;
- public APIs created only to make a failing import compile.

A public API is a contract, not a convenience patch.

## 21. Internal implementation rules

Internal implementation modules may be reorganized, but their visibility must remain controlled.

Allowed posture:

- internal modules may depend on their family’s private helpers;
- internal modules may use lower foundational layers;
- internal modules may expose behavior through a reviewed public surface;
- internal modules may be tested through approved test surfaces.

Forbidden patterns include:

- importing internals across families without an admitted boundary;
- making an internal module public by accident;
- exposing private state to satisfy a single caller;
- using internal reach-through to avoid designing a real API;
- relying on build-system visibility gaps as permission.

Internal means internal even when the compiler can find it.

## 22. Test-only adapter rules

Test-only adapters are allowed only when they are explicit, contained, and unable to become production authority.

A valid test-only adapter must:

- live in test scope or be clearly marked as test-only;
- not be imported by production code;
- not be presented as a production surface;
- preserve the behavior being tested;
- identify any production blocker it is working around;
- be removed or retired when no longer needed, if applicable.

A test-only adapter is invalid if it hides a missing production surface, bypasses authority, or converts incomplete work into acceptance.

## 23. Boundary violation handling

When a boundary violation is found, the response must preserve truth.

The correct response is to identify:

- the importing module;
- the imported surface;
- the boundary that was crossed;
- whether the surface is public, protected, internal, test-only, or tooling-only;
- the authority impact;
- the security impact;
- the build or test impact;
- the exact blocker;
- the proper repair path.

Acceptable repair paths include:

- move the dependency to an admitted public surface;
- narrow the interface;
- add a reviewed boundary surface;
- relocate test-only behavior into tests;
- add a CI guard;
- fail closed with an exact blocker.

Unacceptable repair paths include:

- broad imports;
- alias masking;
- shims that hide ownership;
- deleting tests;
- weakening assertions;
- moving production code into tests;
- moving test-only code into production;
- documenting the violation as acceptable without governance approval.

## 24. Enforcement expectations

Boundary rules should be enforceable through review, tests, scans, CI, or a combination of these.

Potential enforcement mechanisms include:

- import graph scans;
- forbidden include scans;
- protected surface ownership checks;
- public API ownership checks;
- test-only import checks;
- documentation claim checks;
- CMake target materialization checks;
- CTest label coverage checks;
- sanitizer runtime gates;
- CODEOWNERS review routing.

This document defines the policy. Passing enforcement requires the relevant implementation and evidence.

## 25. Boundary review checklist

A boundary-sensitive review should ask:

- What boundary does this change touch?
- Is the imported surface public, protected, internal, test-only, or tooling-only?
- Is the dependency direction allowed?
- Does the change preserve authority flow?
- Does the change expose internal state?
- Does the change create duplicate public ownership?
- Does the change rely on a test-only helper?
- Does the change weaken tests or validation?
- Does the change alter build or CI coverage?
- Does the documentation claim more than the evidence proves?
- If incomplete, is the exact blocker preserved?

A change that cannot answer these questions is not ready for acceptance.

## 26. Relationship to other documents

This document should be read after:

```text
START_HERE.md
GOVERNANCE.md
ENGINEERING_PRINCIPLES.md
AUTHORITY_MODEL.md
ARCHITECTURE.md
```

It should be read before:

```text
THREAT_MODEL.md
SECURITY.md
CI_VALIDATION_GATES.md
REVIEW_POLICY.md
CONTRIBUTING.md
CODEOWNERS
```

`ARCHITECTURE.md` explains how the system is structured.

`MODULE_BOUNDARIES.md` defines how that structure is protected.

`THREAT_MODEL.md` should describe what happens when attackers, mistakes, or shortcuts attempt to cross these boundaries.

`CI_VALIDATION_GATES.md` should describe how boundary claims are validated.

## 27. Current claim discipline

This document does not elevate project status.

Unless the relevant acceptance gates prove otherwise, conservative global boundaries remain in force:

```text
FHE_PRODUCTION_READY = NO
FHE_BOOTSTRAP_COMPLETE = NO
FULL_SANITIZER_CTEST_CLAIMED = NO
```

Module-boundary documentation is not a substitute for validation. It defines the rules that validation must enforce.

## 28. Final boundary rule

The final boundary rule is:

```text
a module may see only what it is allowed to see
a domain may execute only what it is authorized to execute
a public surface must have one clear owner
a test may validate production behavior, but it must not redefine it
```

XPScerpto remains trustworthy only when module visibility, authority flow, implementation behavior, validation evidence, and public claims remain aligned.
