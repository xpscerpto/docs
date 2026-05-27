# XPScerpto Authority Flow

**Status:** architecture reference  
**Scope:** explanatory authority-flow diagrams, contribution admission flow, runtime routing flow, SIMD authority flow, FHE ownership flow, evidence flow, and governance-process flow

This document is a companion reference for understanding how authority moves through XPScerpto.

It does not replace the root-level policy documents. The repository-wide authority rules are defined by:

```text
AUTHORITY_MODEL.md
GOVERNANCE.md
MODULE_BOUNDARIES.md
CI_VALIDATION_GATES.md
REVIEW_POLICY.md
```

This document is not a validation report, security proof, production-readiness claim, acceptance artifact, or source of new authority rules.

If this document conflicts with a root-level policy document, the root-level policy remains authoritative and this document should be updated.

## 1. Purpose

XPScerpto is built around explicit authority rather than ambient capability, local probing, hidden runtime state, or convenience shortcuts.

This file provides a readable map of the intended authority flows used across the project. It helps contributors and reviewers understand how a change should move from proposal to review, from platform authority to domain execution, and from command output to evidence-backed claims.

The diagrams in this file are explanatory. They do not create acceptance, grant execution authority, or replace validation gates.

## 2. Repository authority chain

The repository-wide authority chain is:

```text
platform governance
→ platform.api
→ hardware authority
→ routing capsule
→ domain execution
→ audit and evidence
```

This chain means:

- governance defines what may be trusted;
- `platform.api` exposes admitted platform authority;
- hardware authority provides evidence-backed capability truth;
- routing capsules carry admitted execution decisions;
- domains execute only inside admitted scope;
- audit and evidence preserve what was decided and what was run.

A domain must not manufacture authority for itself.

## 3. Contribution authority flow

A contribution moves through review authority before it can be accepted.

```text
Contributor
  |
  |  submits scoped change with evidence and claim boundary
  v
Contribution Scope Review
  |
  |  identifies affected surfaces, protected areas, and claim scope
  v
Policy Review
  |
  |  checks governance, engineering principles, license status, and claim discipline
  v
Authority Review
  |
  |  checks platform authority, hardware evidence, routing, and domain execution impact
  v
Module Boundary Review
  |
  |  checks imports, public surfaces, internal surfaces, ownership, and test-only access
  v
Security Review
  |
  |  checks threat impact, secret material, fail-closed behavior, and negative cases
  v
Validation Review
  |
  |  checks build scope, test scope, sanitizer runtime, CI gates, and evidence quality
  v
Ownership Review
  |
  |  checks required owners, CODEOWNERS routing, and escalation needs
  v
Merge Authority
```

A contribution is not accepted because it compiles locally. It is accepted only when the evidence supports the claim being made and the relevant owners approve within scope.

## 4. Runtime authority flow

Runtime authority must be admitted before domain execution.

```text
Platform Policy Root
  |
  |  defines allowed platform posture, provider admission, and failure semantics
  v
Build and Configuration Fingerprint
  |
  |  identifies materialized configuration, policy flags, and build posture
  v
Hardware Evidence
  |
  |  records capability, topology, constraints, and routing-relevant conditions
  v
Authority Epoch
  |
  |  binds authority to a validity window, revocation model, and evidence state
  v
Sealed Capability Capsule
  |
  |  packages admitted authority for controlled routing
  v
Runtime Routing Decision
  |
  |  selects an execution route from admitted authority only
  v
Domain Execution
  |
  |  executes without local authority invention
  v
Audit Ledger and Attestation Evidence
```

Domain execution must not create authority from:

- local CPU probing;
- compiler feature macros;
- compiler CPU-support builtins;
- operating-system probing;
- hidden runtime globals;
- benchmark observations;
- test-only shortcuts;
- stale evidence;
- metadata-only reports.

Capability is not authority. Capability becomes usable only when admitted through the governed path.

## 5. SIMD authority flow

SIMD is an authority-routed execution domain.

Allowed flow:

```text
hardware evidence
  → routing envelope
  → decision capsule
  → SIMD routing signal
  → SIMD dispatch plan
  → SIMD execution
```

Rejected flow:

```text
SIMD domain code
  → local CPU probe
  → private dispatch decision
  → unauthorized fast path
```

SIMD may be optimized aggressively only inside the governed authority path. A faster path that bypasses authority is not acceptable.

SIMD performance evidence must identify:

- authority state;
- dispatch eligibility;
- correctness status;
- benchmark scope;
- hardware conditions;
- claim boundaries.

A benchmark result is not execution authority.

## 6. FHE authority and ownership flow

FHE is a highly protected cryptographic domain. Its public surfaces, bootstrap path, key lifecycle, and validation claims require strict ownership and evidence discipline.

Expected flow:

```text
canonical public API owner
  → surface guard
  → internal FHE implementation
  → cryptographic validation
  → evidence record
  → scoped public claim
```

Rejected flow:

```text
retired surface or new forwarding surface
  → alias / shim / adapter
  → apparent compatibility
  → ownership ambiguity
  → overclaim risk
```

FHE changes must preserve:

- canonical public surface ownership;
- key and context binding;
- BFV and CKKS claim separation;
- bootstrap claim separation;
- encrypted-path versus reference-path distinction;
- scale, level, modulus, and error-bound discipline;
- surface guard meaning;
- exact blocker preservation.

FHE progress does not imply repository-wide production readiness.

## 7. Evidence flow

Evidence must preserve scope.

```text
command executed
  → output captured
  → source tree identified
  → configuration recorded
  → test or target scope recorded
  → failure class recorded if failed
  → exact blocker preserved
  → claim limited to evidence scope
```

Evidence must answer:

- what ran;
- where it ran;
- what source tree was used;
- what configuration was used;
- what passed;
- what failed;
- what did not run;
- what claim is supported;
- what claim is not supported.

A log without scope is weak evidence.

A summary without failures is incomplete evidence.

Evidence does not prove more than the command and runtime scope actually covered.

## 8. CI and validation flow

Validation levels must remain separate.

```text
configure
  → build
  → test
  → sanitizer runtime
  → full graph validation
  → evidence integrity
  → scoped acceptance
```

The following distinctions must remain intact:

```text
configure success ≠ build success
build success ≠ test success
test success ≠ sanitizer truth
target success ≠ full graph truth
subsystem success ≠ repository-wide acceptance
```

A precise partial result is acceptable.

A broad claim without matching evidence is not.

## 9. RFC and decision-record flow

Major or long-term changes may require governance review before implementation.

```text
Proposed major change
  → RFC
  → architecture review
  → security review
  → validation plan review
  → decision
  → implementation PR
  → validation gates
  → decision record when long-term impact exists
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

for accepted, rejected, superseded, or long-term decisions.

RFC acceptance authorizes implementation work within scope. It does not prove implementation, sanitizer truth, full graph acceptance, or production readiness.

## 10. Documentation authority flow

Documentation must not outrun evidence.

```text
implemented behavior
  → validated behavior
  → evidence record
  → scoped claim
  → documentation wording
```

Rejected flow:

```text
design goal
  → aspirational wording
  → public claim
```

Documentation that changes claims is review-sensitive. If documentation states a status, it must be supported by the relevant validation gate.

## 11. Final rule

The final rule for this document is:

```text
this file explains authority flow
it does not create authority
it does not replace root policy
it does not prove validation
it does not elevate project status
```

Use this file as a map. Use the root-level policy documents and validation gates as the authority.
