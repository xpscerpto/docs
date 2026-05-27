# Decision Record: <title>

**Decision ID:** DR-YYYY-NNN-short-title  
**Status:** Proposed | Accepted | Rejected | Superseded  
**Date:** YYYY-MM-DD  
**Owners:** <names or teams>  
**Reviewers:** <names or teams>  
**Related RFC:** <link or identifier>  
**Supersedes:** <decision ID, if any>  
**Superseded by:** <decision ID, if any>  
**Affected surfaces:** <paths/modules>  
**Contribution class:** A | B | C | D | E  
**Implementation status:** Not started | In progress | Implemented | Blocked | Superseded  
**Evidence status:** Required | Attached | Partial | Missing | Not applicable  

## 1. Context

Describe the problem, current behavior, affected architecture, and why a decision is required.

Include enough background for a reviewer to understand the decision without relying on private discussion history.

Identify whether this decision affects:

- architecture;
- authority;
- public APIs;
- ownership;
- cryptographic behavior;
- validation gates;
- CI behavior;
- security posture;
- documentation claims;
- release behavior.

## 2. Decision

State the decision clearly. Avoid vague language.

The decision should answer:

- what will change;
- what will not change;
- what is explicitly allowed;
- what is explicitly rejected;
- what surfaces are affected;
- what claims are not being made.

## 3. Authority impact

Describe how the decision affects:

- platform authority;
- `platform.api`;
- hardware evidence;
- runtime routing;
- domain execution;
- audit ledgers;
- merge authority;
- public surface ownership.

If authority is not affected, state that explicitly. If authority is affected, describe how the admitted authority path remains preserved.

## 4. Module boundary impact

List all boundary effects, including:

- imports;
- public surfaces;
- internal surfaces;
- protected surfaces;
- test-only surfaces;
- tooling surfaces;
- ownership changes;
- graph implications.

If a shim, alias, adapter, or forwarding surface is proposed, justify why it is not an ownership bypass and define removal criteria.

If no boundary impact exists, state that explicitly.

## 5. Security impact

Describe:

- threats considered;
- security-sensitive surfaces affected;
- key, seed, randomness, credential, or secret-material impact;
- cryptographic impact, if any;
- memory or concurrency impact, if any;
- fail-closed behavior;
- negative cases;
- residual risks.

If the decision is not security-sensitive, explain why.

## 6. Validation plan

List required evidence.

Include relevant items:

- configure scope;
- build scope;
- module graph validation;
- import boundary validation;
- tests;
- negative tests;
- sanitizer runtime evidence;
- runtime attestation evidence;
- policy scans;
- CI gates;
- documentation updates;
- final flags;
- release evidence, if relevant.

The validation plan must be specific enough that reviewers can determine whether implementation is accepted, blocked, or incomplete.

## 7. Alternatives considered

List alternatives and explain why they were rejected.

For each alternative, include:

- expected benefit;
- reason for rejection;
- authority impact;
- security impact;
- validation impact;
- compatibility impact, if relevant.

Rejected alternatives should be recorded clearly so they are not reopened casually without new evidence.

## 8. Compatibility and migration

Explain:

- migration steps;
- deprecation strategy;
- compatibility behavior;
- whether old behavior remains supported;
- whether compatibility is temporary or permanent;
- how compatibility will be tested;
- removal criteria for temporary behavior.

If no migration is required, state that explicitly.

## 9. Consequences

Describe expected benefits, tradeoffs, operational costs, new obligations, and risks introduced by the decision.

Include:

- implementation costs;
- review costs;
- testing costs;
- CI costs;
- maintenance costs;
- documentation costs;
- user or contributor impact.

## 10. Rejection and rollback criteria

Define what would cause the decision to be reverted, paused, or superseded.

Examples:

- validation gate failure;
- security review failure;
- authority violation;
- boundary violation;
- unbounded compatibility risk;
- unacceptable performance regression;
- missing evidence;
- production-claim conflict;
- implementation complexity exceeding accepted scope.

Rollback criteria should be clear enough to act on.

## 11. Evidence links

Add links to relevant evidence.

Examples:

- CI logs;
- CTest logs;
- build logs;
- module graph scans;
- import boundary scans;
- sanitizer logs;
- runtime attestation logs;
- audit evidence;
- final flags;
- review artifacts;
- related pull requests;
- related RFCs;
- related reports.

If evidence is not yet available, state:

```text
Evidence status: Missing
```

or:

```text
Evidence status: Required before implementation acceptance
```

## 12. Root policy compatibility

State whether the decision is compatible with:

```text
GOVERNANCE.md
ENGINEERING_PRINCIPLES.md
AUTHORITY_MODEL.md
ARCHITECTURE.md
MODULE_BOUNDARIES.md
THREAT_MODEL.md
SECURITY.md
CI_VALIDATION_GATES.md
REVIEW_POLICY.md
OWNERSHIP_MODEL.md
```

If the decision requires a policy change, identify the policy file and required update.

A decision record must not silently override root-level policy.

## 13. Required owner approvals

List the required owner groups or reviewers.

Examples:

```text
governance maintainers
security maintainers
architecture maintainers
platform maintainers
hardware maintainers
SIMD maintainers
FHE maintainers
crypto maintainers
CI maintainers
validation maintainers
release maintainers
sovereign maintainers
```

Only list the roles required for the decision.

If a required owner is unavailable, state the escalation path.

## 14. Implementation notes

Describe any implementation constraints that must be respected.

Examples:

- no test weakening;
- no public API aliasing;
- no compatibility shim without removal criteria;
- no local hardware probing;
- no production-readiness claim;
- no sanitizer-clean claim without sanitizer runtime;
- no target-only claim as full graph truth.

Implementation notes should be strict enough to guide future pull requests.

## 15. Production readiness statement

State explicitly whether this decision makes any production-readiness claim.

Default answer:

```text
This decision does not by itself claim production readiness. Production readiness requires the applicable validation gates to pass with evidence.
```

If the decision changes production-readiness criteria, identify the exact gate and evidence required.

## 16. Final decision boundary

End with a clear statement of what this decision does and does not authorize.

Example:

```text
This decision authorizes implementation work within the scope described above. It does not authorize production-readiness claims, sanitizer-clean claims, full graph acceptance claims, or release acceptance claims without the applicable validation evidence.
```
