# XPScerpto Module Map

**Status:** architecture reference  
**Scope:** human-readable repository surface map, module-family orientation, protected-area guide, and contributor navigation companion

This document provides a quick map of major XPScerpto repository areas.

It does not replace the root-level architecture or boundary policy. Normative rules are defined by:

```text
ARCHITECTURE.md
MODULE_BOUNDARIES.md
AUTHORITY_MODEL.md
GOVERNANCE.md
```

This document is not a validation report, security proof, production-readiness claim, acceptance artifact, or import policy.

If this file conflicts with a root-level policy document, the root-level policy remains authoritative and this file should be updated.

## 1. Purpose

The purpose of this file is to help contributors and reviewers understand the repository layout at a high level.

This is a navigation companion.

It does not define:

- final import permissions;
- canonical public API ownership;
- production-readiness status;
- subsystem acceptance;
- validation gates;
- security proofs.

Use it to orient yourself. Use root-level policy files for decisions.

## 2. High-level source map

The source tree may include module families such as:

```text
include/xps/crypto/
  base/
  platform/
  platform.api/
  hw/
  audit/
  simd/
  FHE/
  aes/
  aead/
  hash/
  kdf/
  mac/
  pqc/
  ed25519/
  x25519/
  rsa/
  VeloSeal64/
  number/
  atomic/
  memory/
  integrity/
  internal/
  super/
```

This map describes intended responsibility, not production status.

## 3. Family roles

| Family | Role |
|---|---|
| `base/` | Foundational types, checked primitives, and low-level policy anchors |
| `platform/` | Platform governance, providers, admission, and operating boundary |
| `platform.api/` | Admitted platform-facing surface |
| `hw/` | Hardware evidence, topology, capability interpretation, and routing support |
| `audit/` | Evidence ledgers, attestation records, and audit surfaces |
| `simd/` | Authority-routed acceleration and data movement |
| `FHE/` | Homomorphic encryption domain, BFV, CKKS, bootstrap, and FHE surface guards |
| `aes/` | AES-related cryptographic surfaces |
| `aead/` | AEAD composition and authenticated encryption surfaces |
| `hash/` | Hash and XOF-related domains |
| `kdf/` | Key-derivation domains |
| `mac/` | Message-authentication domains |
| `pqc/` | Post-quantum cryptographic domains |
| `ed25519/` | Ed25519 domain |
| `x25519/` | X25519 domain |
| `rsa/` | RSA domain |
| `VeloSeal64/` | Specialized governed runtime or cryptographic domain surface |
| `number/` | Arithmetic and number-support surfaces |
| `atomic/` | Atomic and concurrency-support surfaces |
| `memory/` | Memory-safety and buffer-support surfaces |
| `integrity/` | Integrity and verification support |
| `internal/` | Internal shared components, not general public API |
| `super/` | Higher-level orchestration surfaces; must not bypass ownership |

A family role does not grant permission to bypass module boundaries.

## 4. Intended authority direction

The intended authority direction is:

```text
base
  → platform
  → platform.api
  → hardware evidence and routing
  → audit and attestation
  → domain execution
```

Domain modules consume authority. They do not create it.

Domain modules must not:

- bypass `platform.api`;
- import platform internals for convenience;
- manufacture hardware authority;
- infer runtime permission from compiler macros;
- use local probing as execution authority;
- present fallback behavior as admitted execution.

## 5. Public, protected, and internal surfaces

A module may be public, protected, internal, test-only, or tooling-only.

The canonical boundary policy is defined in:

```text
MODULE_BOUNDARIES.md
```

This file only maps common areas.

A protected surface may require additional review even if it is not a public API.

A public API surface must have one clear canonical owner.

An internal surface must not become public by accident.

A test-only surface must not become production authority.

## 6. Platform and platform.api

Platform areas are authority-sensitive.

Expected relationship:

```text
platform/
  → owns provider and platform internals

platform.api/
  → exposes admitted platform-facing authority
```

Domain code should consume admitted platform surfaces. It should not reach into platform internals directly.

Changes in this area may require governance, security, architecture, and ownership review.

## 7. Hardware authority

The `hw/` family owns hardware evidence and routing-relevant capability interpretation.

Hardware authority must be produced through approved hardware surfaces and consumed through admitted routing.

Domains must not create their own hardware truth.

Changes in `hw/` may affect:

- topology evidence;
- capability evidence;
- routing decisions;
- dispatch eligibility;
- platform constraints;
- audit evidence.

## 8. SIMD

SIMD is an authority-routed execution domain.

Allowed relationship:

```text
hardware evidence
  → routing signal
  → SIMD dispatch plan
  → SIMD execution
```

Rejected relationship:

```text
SIMD code
  → local CPU probe
  → private dispatch
```

SIMD changes should preserve:

- correctness;
- bounds behavior;
- overlap behavior;
- scalar fallback behavior;
- dispatch eligibility;
- authority routing;
- benchmark scope.

A performance improvement outside admitted authority is not acceptable.

## 9. FHE

FHE is a highly protected cryptographic domain.

It may include:

- foundation, types, parameters, and context;
- polynomial arithmetic;
- RNS and NTT;
- key material;
- key generation;
- key switching;
- relinearization;
- Galois and rotation behavior;
- BFV behavior;
- CKKS behavior;
- encoding and decoding;
- bootstrap planning and runtime;
- public API surfaces;
- surface guards.

FHE changes must preserve:

- parameter validity;
- context binding;
- key lifecycle;
- encrypted-path semantics;
- scale and level invariants;
- modulus and RNS correctness;
- public API ownership;
- fail-closed behavior;
- claim boundaries.

FHE subsystem progress does not imply repository-wide production readiness.

## 10. Cryptographic domains

Cryptographic domains such as `hash/`, `kdf/`, `mac/`, `aead/`, `aes/`, `pqc/`, `ed25519/`, `x25519/`, `rsa/`, and `VeloSeal64/` are security-sensitive by default.

Changes may require review for:

- parameter validation;
- key or seed lifecycle;
- randomness behavior;
- domain separation;
- misuse resistance;
- negative tests;
- public API ownership;
- documentation claims.

A small cryptographic patch can still be high risk.

## 11. Support families

Support families such as `base/`, `memory/`, `number/`, `atomic/`, and `integrity/` are not ordinary low-risk utilities.

They may affect:

- memory safety;
- checked arithmetic;
- lifetime behavior;
- alignment assumptions;
- concurrency behavior;
- hidden runtime state;
- cross-domain behavior.

Support code must not become a hidden authority owner.

## 12. Build, validation, and tooling surfaces

The following areas are validation-sensitive:

```text
CMakeLists.txt
cmake/
tests/
tools/
.github/
```

These surfaces control what can be proven.

A build, test, or CI change may weaken the whole repository if it:

- hides graph failures;
- skips validation;
- changes labels;
- changes sanitizer meaning;
- changes evidence generation;
- alters final flags;
- allows overclaim.

Treat validation infrastructure as protected.

## 13. Documentation surfaces

Documentation areas may include:

```text
docs/architecture/
docs/contribution/
docs/governance/
docs/security/
```

These directories are companion documentation layers. They do not replace root-level policy.

Recommended classification:

```text
root documents      = canonical repository-wide policy
docs/architecture   = explanatory architecture companions
docs/contribution   = contributor navigation companions
docs/governance     = RFC and decision-process companions
docs/security       = formal security specification companions
```

Documentation must not claim more than evidence proves.

## 14. Governance process companions

Major changes may need the governance process under:

```text
docs/governance/
```

Use:

```text
docs/governance/rfc-process.md
```

for major proposals.

Use:

```text
docs/governance/decision-record-template.md
```

for durable decisions.

A governance-process document does not prove implementation or production readiness.

## 15. Final rule

The final rule for this map is:

```text
this file helps readers navigate the repository
it does not grant import permission
it does not define ownership by itself
it does not replace root-level policy
it does not elevate project status
```

If this map becomes stale, update it. If it conflicts with root policy, root policy wins.
