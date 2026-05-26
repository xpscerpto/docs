# XPScerpto Architecture

**Status:** normative architecture overview  
**Scope:** repository-wide architecture, layer responsibilities, authority flow, protected surfaces, runtime routing, validation structure, and documentation alignment

XPScerpto is a governed sovereign cryptographic infrastructure platform. It is not organized as a loose collection of independent cryptographic utilities. Its architecture is built around explicit authority, controlled module boundaries, fail-closed behavior, evidence-backed runtime decisions, and conservative claim discipline.

This document explains how the system is structured after the governance rules, engineering principles, and authority model are understood.

This document is not a validation report, security proof, production-readiness claim, or acceptance artifact. It describes the intended and governed architecture of the repository. Acceptance, production readiness, sanitizer truth, and subsystem closure require the relevant gates and evidence.

## 1. Architectural thesis

The central architectural rule is:

```text
platform governance
→ platform.api
→ hardware authority
→ routing capsule
→ domain execution
→ audit and evidence
```

This order is architectural, not decorative.

Domain code may execute only under authority that is admitted through the platform and hardware evidence chain. A domain must not invent its own execution authority by probing the operating system, CPU, compiler environment, hidden runtime state, or local capability shortcuts.

The architecture is designed to keep four dimensions aligned:

```text
implementation
authority
evidence
claims
```

If those dimensions disagree, the architecture must preserve the stricter truth and fail closed.

## 2. Architectural position of this document

This document should be read after:

```text
START_HERE.md
GOVERNANCE.md
ENGINEERING_PRINCIPLES.md
AUTHORITY_MODEL.md
```

It should be read before:

```text
MODULE_BOUNDARIES.md
THREAT_MODEL.md
CI_VALIDATION_GATES.md
REVIEW_POLICY.md
CONTRIBUTING.md
```

`AUTHORITY_MODEL.md` defines where execution authority comes from.

`ARCHITECTURE.md` explains how the system is structured around that authority.

`MODULE_BOUNDARIES.md` should define the concrete import and ownership boundaries.

`CI_VALIDATION_GATES.md` should define how the architecture is validated.

## 3. Primary architecture layers

XPScerpto is organized into layered authority and execution responsibilities:

```text
governance and policy documents
  ↓
build and CI policy
  ↓
base layer
  ↓
platform layer
  ↓
platform.api admitted surface
  ↓
hardware authority layer
  ↓
routing and dispatch layer
  ↓
audit and evidence layer
  ↓
domain execution layers
  ↓
tests, tools, and release evidence
```

This order defines responsibility.

Lower layers do not automatically grant authority to higher layers. Higher layers must consume authority through admitted surfaces and evidence-backed decisions.

## 4. Repository-level architecture map

The repository contains several major families. The following map describes their architectural role, not a production-readiness status.

| Family | Architectural role | Authority posture |
|---|---|---|
| `base` | Root types, checked primitives, low-level policy anchors, foundational utilities | Foundational; must remain narrow and stable |
| `platform` | Platform root, provider admission, runtime policy, operating boundary | Authority owner and admission layer |
| `platform.api` | Approved platform-facing surface | Admitted boundary for platform authority |
| `hw` | Hardware evidence, topology, capability interpretation, decision support | Authority producer; not a domain shortcut |
| `simd` | Authority-routed data movement and compute acceleration | Authority consumer; no local hardware authority |
| `FHE` | Homomorphic encryption, BFV, CKKS, bootstrap, key switching, NTT/RNS, canonical surfaces | Highly restricted cryptographic core |
| `hash`, `kdf`, `mac`, `aead`, `aes` | Symmetric and primitive cryptographic domains | Domain execution under policy |
| `pqc`, `ed25519`, `x25519`, `rsa` | Asymmetric and post-quantum cryptographic domains | Cryptographic domain surfaces |
| `number`, `atomic`, `memory`, `integrity` | Support families for arithmetic, synchronization, memory, and integrity behavior | Support layers; must not bypass authority |
| `audit` | Evidence, ledgers, replay, attestation, and traceability | Evidence owner |
| `super` | Higher-level orchestration, adapters, providers, or integration surfaces | Must not weaken lower-level ownership |
| `tests` | Validation, regression, negative tests, authority tests, sanitizer execution | Validation authority only |
| `tools` | CI helpers, graph scans, evidence indexing, command-center support | Tooling authority only |
| `docs` | Normative and explanatory documentation | Documentation authority, not runtime authority |
| `.github` | Review templates, CI workflows, repository automation | Governance and validation support |

A family may contain many modules, but family membership does not grant permission to bypass the architecture.

## 5. Authority-first architecture

The architecture is authority-first.

The system must not be structured around the question:

```text
What can this machine do?
```

It must be structured around the question:

```text
What has the governed authority chain admitted this domain to do?
```

This distinction is central to XPScerpto.

A CPU feature, operating-system facility, compiler option, runtime observation, or successful experiment is not execution authority by itself. These may become raw evidence only when collected, constrained, and admitted by the approved authority path.

## 6. Base layer

The base layer provides foundational types, checked primitives, low-level constraints, and policy anchors used by the rest of the system.

The base layer should remain narrow, explicit, and free of hidden domain policy.

Architectural expectations for the base layer:

- provide stable foundational abstractions;
- avoid hidden runtime authority;
- avoid domain-specific shortcuts;
- preserve checked behavior where required;
- support no-exception and no-RTTI policy where applicable;
- remain suitable for use by higher layers without importing platform or domain behavior.

The base layer must not become a hidden policy backdoor.

## 7. Platform layer

The platform layer owns platform admission and operating boundary behavior.

It may include provider registration, runtime state, provider-specific implementations, platform facts, bootstrap behavior, and platform control surfaces.

The platform layer is authority-sensitive because it is close to the root of runtime admission.

Architectural expectations for the platform layer:

- admit providers through governed rules;
- expose approved behavior through `platform.api`;
- preserve provider ownership;
- keep internal platform state internal;
- reject unauthorized provider behavior;
- fail closed on missing or invalid platform authority;
- preserve auditability for platform-sensitive decisions.

Domain code must not import platform internals to obtain convenience access.

## 8. `platform.api` layer

`platform.api` is the admitted platform-facing surface.

It should be narrow, stable, reviewable, and intentionally exposed.

The purpose of `platform.api` is to prevent domain code from reaching into platform internals. It is the approved boundary through which platform authority is consumed by the rest of the system.

Architectural expectations for `platform.api`:

- expose admitted platform facts and decisions;
- avoid leaking internal provider mechanics;
- remain stable enough for domain consumption;
- preserve the difference between public authority and private implementation;
- make authority review possible.

A domain that bypasses `platform.api` is bypassing the architecture.

## 9. Hardware authority layer

The hardware layer is responsible for evidence-backed hardware truth.

It may model topology, capability evidence, routing constraints, decision capsules, performance-relevant facts, thermal or power constraints, and domain-specific hardware eligibility.

The hardware layer does not exist so that domains can probe hardware themselves. It exists so that domains do not have to.

Architectural expectations for the hardware layer:

- collect and normalize hardware evidence through approved mechanisms;
- distinguish raw capability from admitted capability;
- provide routing information to domains through approved structures;
- preserve auditability and replayability where required;
- fail closed when capability truth is incomplete or contradictory;
- prevent domain-local hardware authority.

Hardware authority should be consumed, not duplicated.

## 10. Routing and dispatch layer

Routing and dispatch convert admitted authority into execution decisions.

This layer may produce routing capsules, dispatch plans, domain signals, or equivalent decision structures.

A routing object must not be a decorative wrapper around local probing. It must carry a decision that has passed through the governed authority chain.

Architectural expectations for routing and dispatch:

- preserve source authority;
- preserve decision scope;
- expose enough information for audit;
- reject unsupported or incomplete authority;
- keep domain execution inside admitted bounds;
- prevent unauthorized fast paths.

Routing is the architectural boundary between authority production and domain execution.

## 11. Audit and evidence layer

The audit and evidence layer records what was decided, why it was decided, what evidence supported it, and what scope the decision covered.

Audit evidence is not optional for authority-sensitive architecture. It is how XPScerpto prevents a successful path from becoming an unreviewed claim.

Architectural expectations for audit and evidence:

- record authority-sensitive decisions;
- distinguish admitted execution from fallback behavior;
- support replay or review where applicable;
- support final flags and acceptance records;
- preserve exact blockers for incomplete work;
- reject evidence that is too vague to support the claim.

An architecture that cannot explain its decisions is not sufficiently governed.

## 12. Domain execution layer

Domain execution includes cryptographic and performance-oriented modules that perform specialized work under admitted authority.

Examples include:

```text
FHE
SIMD
hash
KDF
MAC
AEAD
AES
PQC
Ed25519
X25519
RSA
VeloSeal64
number
atomic
memory
integrity
```

Domain code consumes authority. It does not create platform authority or hardware authority for itself.

Architectural expectations for domain execution:

- preserve admitted authority scope;
- validate inputs, contexts, keys, and parameters;
- fail closed when authority or validation is incomplete;
- avoid local probing where governed authority is required;
- preserve cryptographic correctness and security boundaries;
- expose precise testable behavior;
- avoid overclaiming subsystem or global acceptance.

Domain execution must be powerful but not self-authorizing.

## 13. SIMD architecture

SIMD is an authority-routed execution domain.

Its purpose is to provide high-performance data movement or compute paths without allowing the SIMD domain to become its own hardware authority.

The expected SIMD authority path is:

```text
platform admission
→ hardware evidence
→ hardware decision
→ SIMD routing signal
→ SIMD dispatch plan
→ SIMD execution
```

SIMD code must not decide eligibility by using local CPU probes, compiler CPU-support builtins, compiler ISA macros, operating-system hardware files, benchmark observations, or hidden runtime state as authority.

Architectural expectations for SIMD:

- consume hardware authority through approved routing;
- preserve correctness across scalar and accelerated paths;
- maintain explicit bounds and overlap behavior;
- keep fast paths within admitted dispatch decisions;
- fail closed when authority is missing;
- document performance claims conservatively;
- test both correctness and authority routing.

A faster SIMD path that bypasses authority is architecturally invalid.

## 14. FHE architecture

FHE is a highly restricted cryptographic core.

It includes or may include BFV, CKKS, encoders, evaluators, key generation, keys, key switching, relinearization, Galois operations, NTT/RNS arithmetic, rescale behavior, bootstrap planning, bootstrap runtime, public API surfaces, and surface guards.

FHE architecture must preserve:

- parameter validity;
- context binding;
- key lifecycle discipline;
- encrypted-path semantics;
- scale and level invariants;
- modulus and RNS correctness;
- rotation and switching material validity;
- error-budget accounting;
- canonical public surface ownership;
- fail-closed behavior;
- claim boundaries.

FHE architectural expectations:

- no plaintext shortcut may be presented as encrypted-path acceptance;
- no test-only behavior may be presented as production behavior;
- no metadata-only check may be presented as cryptographic proof;
- no partial BFV closure may be presented as CKKS or full FHE closure;
- no partial bootstrap step may be presented as bootstrap completion;
- no historical or retired public surface may be reintroduced without governance approval;
- no canonical public API ambiguity is acceptable.

FHE is not accepted as a whole merely because one subsystem passes. Claims must remain scoped to the gate that proved them.

## 15. Cryptographic domain architecture

Cryptographic domains outside FHE must also preserve conservative security behavior.

For domains such as hash, KDF, MAC, AEAD, AES, PQC, Ed25519, X25519, RSA, and related support layers, architecture must preserve:

- clear public surfaces;
- valid parameter handling;
- input and output bounds;
- key or seed lifecycle where applicable;
- deterministic test coverage where appropriate;
- negative tests for invalid or adversarial inputs;
- no hidden fallback that weakens security;
- no overclaim beyond tested scope.

A cryptographic primitive may be small, but its architecture is still security-sensitive.

## 16. Build and CI architecture

Build and CI are part of the architecture, not external decoration.

The build system defines what is materialized. CI defines what is validated. CTest and sanitizer runs define what evidence exists.

Architectural expectations for build and CI:

- preserve module graph integrity;
- materialize required targets before claiming test truth;
- distinguish configure success from build success;
- distinguish build success from test success;
- distinguish sanitizer configuration from sanitizer runtime truth;
- keep test labels meaningful;
- prevent target-only success from being presented as full graph truth;
- preserve evidence logs and final flags where required.

The architecture must make it difficult to overclaim from partial execution.

## 17. Tests architecture

The tests directory is an architectural validation surface.

Tests do not merely check implementation behavior. They preserve governance, authority, module boundaries, security assumptions, and regression history.

High-value test categories include:

- correctness tests;
- negative security tests;
- authority-routing tests;
- platform provider tests;
- hardware evidence tests;
- SIMD authority and performance tests;
- FHE cryptographic correctness tests;
- bootstrap and public surface tests;
- module boundary scans;
- command-center and evidence tests;
- sanitizer-backed runtime tests;
- regression tests for previously closed blockers.

A test may be local in code location but global in architectural importance.

Tests must not be deleted, weakened, renamed, or bypassed to create acceptance.

## 18. Tools architecture

Tools support validation, evidence, graph scanning, replay, reporting, and command-center behavior.

Tools may help enforce governance, but tools do not replace governance.

Architectural expectations for tools:

- make validation repeatable;
- expose failures clearly;
- preserve source tree identity where applicable;
- help identify exact blockers;
- avoid converting missing evidence into success;
- avoid making release or production claims by themselves.

A tool may produce evidence. It does not automatically establish acceptance unless the relevant gate accepts that evidence.

## 19. Documentation architecture

Documentation is an architectural surface.

The top-level documentation set is intentionally split by responsibility:

- `START_HERE.md` introduces the repository and read order.
- `GOVERNANCE.md` defines decision authority and claim discipline.
- `ENGINEERING_PRINCIPLES.md` defines engineering discipline.
- `AUTHORITY_MODEL.md` defines the execution authority chain.
- `ARCHITECTURE.md` defines the system structure.
- `MODULE_BOUNDARIES.md` defines import and ownership boundaries.
- `THREAT_MODEL.md` defines adversarial assumptions.
- `SECURITY.md` defines reporting and security-change policy.
- `CI_VALIDATION_GATES.md` defines validation gates and evidence requirements.
- `REVIEW_POLICY.md` defines review expectations.
- `CONTRIBUTING.md` defines contributor workflow.
- `CODEOWNERS` defines review routing.

These documents should not collapse into one generic README. Their separation mirrors the architecture: governance, authority, structure, boundaries, threats, validation, and contribution are distinct concerns.

Documentation must not claim more than evidence proves.

## 20. Public surface ownership

Public surfaces require canonical ownership.

A public API surface must have one clear owner. Aliases, forwarding surfaces, compatibility wrappers, and adapters must not be used to hide ownership ambiguity.

Architectural expectations for public surfaces:

- one canonical owner;
- explicit review path;
- stable import boundary;
- clear relationship to internal implementation;
- no duplicate public authority;
- no retired surface reintroduction without governance approval.

A public API that cannot identify its owner is not architecturally stable.

## 21. Module boundary architecture

Module boundaries define what may see what.

A module boundary is not just a compilation rule. It is an architectural security and ownership boundary.

A valid boundary should answer:

- who owns this surface?
- who may import it?
- what authority does it expose?
- what internal state remains hidden?
- what tests protect the boundary?
- what happens when the boundary is violated?

Boundary violations must not be repaired by broad imports, alias masking, or convenience adapters.

Detailed import rules belong in `MODULE_BOUNDARIES.md`.

## 22. Runtime failure architecture

Runtime failure must be explicit.

When authority, validation, context, key material, memory bounds, routing eligibility, or platform support is missing, the architecture should prefer precise refusal over ambiguous behavior.

Expected failure behavior includes:

- explicit unsupported result;
- exact blocker where possible;
- preserved diagnostic evidence;
- no silent downgrade;
- no hidden fallback;
- no success claim outside admitted scope.

Fail-closed behavior is architectural integrity.

## 23. Evidence and release architecture

Release and acceptance artifacts must preserve the relationship between source, evidence, and claims.

Evidence may include:

- source tree identifiers;
- patch records;
- build logs;
- CTest logs;
- sanitizer runtime logs;
- final flags;
- evidence bundles;
- blocker reports;
- CI records;
- review records.

Packaged evidence may establish provenance. It must not replace live validation when live validation is required.

Release notes must preserve claim boundaries. A release note must not convert partial subsystem success into global acceptance.

## 24. Forbidden architecture patterns

The following patterns are architecturally rejected unless explicitly isolated as non-acceptance investigative work:

- local hardware authority inside domains;
- platform internals imported by domain code for convenience;
- public API aliases that hide canonical ownership;
- shims or adapters that bypass protected surfaces;
- silent fallback from governed to ungoverned execution;
- performance paths outside admitted routing;
- metadata-only proof presented as runtime validation;
- configure-only, build-only, or target-only overclaims;
- historical evidence presented as live acceptance when live replay is required;
- deleted tests or weakened assertions used to create acceptance;
- partial subsystem closure presented as global closure;
- documentation that claims accepted behavior without evidence.

An architecture that depends on these patterns must be redesigned or explicitly marked as blocked.

## 25. Architecture change discipline

Architecture changes require special care.

A change is architectural if it affects:

- authority flow;
- platform admission;
- hardware evidence;
- routing or dispatch;
- public surface ownership;
- cryptographic semantics;
- module boundaries;
- build graph materialization;
- CI validation gates;
- sanitizer truth;
- evidence format;
- production claim boundaries.

Architectural changes should include:

- reason for the change;
- affected surfaces;
- authority impact;
- security impact;
- module graph impact;
- validation plan;
- migration plan, if needed;
- exact blocker if incomplete;
- documentation updates.

Long-lived architecture changes should be recorded through a decision record or equivalent reviewed process.

## 26. Relationship to validation gates

Architecture defines structure. Validation gates prove claims about that structure.

A valid architecture does not automatically prove production readiness.

The following distinctions must remain clear:

```text
architecture documented ≠ architecture validated
target built ≠ graph materialized
tests passed ≠ sanitizer clean
subsystem accepted ≠ platform production-ready
BFV accepted ≠ CKKS accepted
CKKS accepted ≠ bootstrap complete
bootstrap complete ≠ production-ready
```

Validation gates must define their scope and evidence.

## 27. Current claim discipline

This document does not elevate project status.

Unless the relevant acceptance gates prove otherwise, conservative global boundaries remain in force:

```text
FHE_PRODUCTION_READY = NO
FHE_BOOTSTRAP_COMPLETE = NO
FULL_SANITIZER_CTEST_CLAIMED = NO
```

These boundaries protect the architecture from accidental overclaim.

## 28. Final architectural rule

The final architectural rule is:

```text
structure must preserve authority
authority must preserve evidence
evidence must preserve claim discipline
claim discipline must preserve trust
```

XPScerpto should be reviewed as a governed cryptographic infrastructure system. Its architecture is successful only when the system structure, authority path, validation evidence, and public claims remain aligned.
