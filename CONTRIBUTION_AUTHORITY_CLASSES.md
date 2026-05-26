# XPScerpto Contribution Authority Classes

**Status:** normative contribution classification  
**Scope:** who may contribute where, what evidence is required, when review escalates

Contribution authority is classified by risk. The class is determined by affected surfaces and semantic impact, not by patch size.

## Class A — Documentation and Website Contributors

### Allowed scope

Class A contributors may propose changes to:

- explanatory documentation;
- website content;
- diagrams;
- tutorials;
- translations;
- examples that do not change production semantics;
- contributor onboarding material.

### Requirements

Class A changes must:

- avoid production-readiness claims unless evidence exists;
- keep terminology consistent with `GOVERNANCE.md`, `AUTHORITY_MODEL.md`, and `MODULE_BOUNDARIES.md`;
- clearly distinguish design intent from implemented and validated behavior;
- not weaken normative security or governance language without review.

### Escalation triggers

Escalate to governance review when a documentation change:

- changes a normative rule;
- redefines authority;
- changes security reporting expectations;
- changes module boundary language;
- describes a new public API contract.

## Class B — Validation Contributors

### Allowed scope

Class B contributors may propose:

- tests;
- CI checks;
- policy scans;
- benchmark harnesses;
- evidence tooling;
- regression coverage;
- documentation for validation commands.

### Requirements

Class B changes must:

- not alter production semantics unless explicitly escalated;
- preserve or improve coverage;
- avoid target-only overclaims;
- produce repeatable evidence;
- state whether a scan is advisory or gate-enforced;
- not hide failures by disabling tests.

### Escalation triggers

Escalate to Class C/D/E when a validation change:

- modifies production code;
- weakens a security assertion;
- changes sanitizer or graph gate semantics;
- removes tests without coverage transfer;
- changes how authority or runtime attestation is interpreted.

## Class C — Controlled Runtime Contributors

### Allowed scope

Class C contributors may propose controlled runtime changes to:

- domain execution code;
- provider code that does not own root authority;
- SIMD execution kernels under authority routing;
- performance-sensitive paths that consume existing authority;
- domain orchestrators that do not change root policy.

### Requirements

Class C changes require:

- runtime maintainer review;
- build and relevant CTest evidence;
- module graph/import evidence if imports changed;
- sanitizer runtime evidence for memory/concurrency-sensitive work;
- runtime attestation evidence when routing or provider behavior changes;
- explicit no-bypass statement.

### Hard prohibitions

Class C contributors must not:

- add local hardware probing in SIMD;
- bypass HW routing capsules;
- bypass platform/provider admission;
- introduce unapproved dependencies;
- add shims or adapters to bypass ownership;
- claim production-ready status.

## Class D — Restricted Security Contributors

### Scope

Class D includes:

- hardware authority;
- platform routing;
- capsule validation;
- audit ledgers;
- runtime admission logic;
- security policy enforcement;
- fail-closed semantics;
- security-sensitive build or CI gates.

### Requirements

Class D changes require:

- security architecture review;
- threat-model impact statement;
- negative tests for bypass or malformed input where applicable;
- module graph/import evidence;
- sanitizer runtime evidence when relevant;
- runtime attestation evidence when runtime admission changes;
- decision record when the change has long-term architectural effect.

### Escalation triggers

Escalate to Class E when the change touches:

- platform governance root semantics;
- canonical public surfaces;
- FHE bootstrap ownership;
- authority root definitions;
- public API ownership transfer.

## Class E — Sovereign Core Maintainers Only

### Scope

Class E includes:

- FHE bootstrap canonical semantics;
- FHE public API ownership;
- platform governance root;
- authority root semantics;
- canonical surface creation, retirement, or transfer;
- repository-wide policy weakening;
- dependency policy changes with security impact.

### Requirements

Class E changes are not accepted as ordinary external contributions. They require sovereign core maintainer handling and full evidence appropriate to the affected system.

A Class E change must include:

- explicit purpose and scope;
- canonical ownership proof if public surfaces are touched;
- full relevant graph evidence;
- security review;
- non-regression evidence;
- sanitizer runtime evidence when applicable;
- decision record;
- explicit statement that no adapter, shim, alias, or fallback ownership path was introduced.

## Classification examples

| Contribution | Class | Reason |
|---|---|---|
| Fix typo in website diagram | A | documentation only |
| Add a policy scan for forbidden SIMD probes | B | validation tooling |
| Add a SIMD copy kernel using existing authorized route | C | runtime execution under authority |
| Change HW routing capsule validation | D | authority admission |
| Move FHE public API ownership | E | canonical surface ownership |
| Delete a failing FHE test | Escalate/Reject | validation truth affected |
| Add CPUID inside SIMD | Reject | authority bypass |
| Add adapter to satisfy old public surface | Reject | ownership bypass |

## Reviewer rule

A reviewer may review at or below their authority class. A lower-class approval cannot authorize a higher-class change.
