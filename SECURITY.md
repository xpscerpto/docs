# XPScerpto Security Policy

**Status:** normative security reporting and security-change policy  
**Scope:** repository-wide vulnerability reporting, private disclosure, security triage, security-sensitive changes, evidence requirements, release discipline, and security claim boundaries

XPScerpto is a governed sovereign cryptographic infrastructure platform. Security reports and security-sensitive changes must be handled with evidence, restraint, and clear authority boundaries.

This document defines how vulnerabilities should be reported, how security-sensitive changes should be reviewed, and what evidence is required before a security claim may be accepted.

This document applies to the entire repository. It is not limited to any single subsystem. It covers platform, platform APIs, hardware authority, routing, SIMD, cryptographic domains, memory behavior, build and CI, tools, tests, evidence, documentation, and release claims.

This document is not a validation report, security proof, production-readiness claim, or acceptance artifact. It does not claim that XPScerpto is production-ready.

## 1. Security posture

XPScerpto treats security as a repository-wide discipline.

Security includes more than cryptographic algorithms. It also includes:

- execution authority;
- module boundaries;
- platform admission;
- provider behavior;
- hardware evidence;
- routing decisions;
- dispatch eligibility;
- public API ownership;
- key and context binding;
- parameter validation;
- memory safety;
- concurrency behavior;
- sanitizer truth;
- build and CI truth;
- evidence integrity;
- release discipline;
- documentation claim discipline.

A defect may be security-relevant if it weakens authority, bypasses a protected surface, hides a blocker, invalidates evidence, weakens a test, or creates a false claim about the project’s state.

## 2. Supported status and claim boundaries

XPScerpto is under active sovereign validation and hardening.

Unless the relevant acceptance gates prove otherwise, conservative global boundaries remain in force:

```text
PRODUCTION_READY = NO
FULL_SANITIZER_CTEST_CLAIMED = NO
FULL_GRAPH_ACCEPTED = NO
```

Subsystem-specific flags may also remain conservative unless their own gates prove otherwise. Examples include:

```text
FHE_PRODUCTION_READY = NO
FHE_BOOTSTRAP_COMPLETE = NO
```

These examples do not limit the policy to FHE. They illustrate that security and production claims must remain scoped to the exact subsystem and gate that proved them.

This means:

- do not describe the project as production-ready without the required evidence;
- do not describe a subsystem as accepted without its required evidence;
- do not describe sanitizer status as clean without runtime sanitizer evidence;
- do not convert partial subsystem success into global security acceptance;
- do not use one domain’s acceptance as proof for another domain.

Security reports are welcome even when a subsystem is not production-ready. Lack of production-ready status does not make a security issue irrelevant.

## 3. Reporting a vulnerability

Report vulnerabilities through a private channel.

Preferred reporting paths, in order:

1. the repository host’s private security advisory feature, if available;
2. a private security contact or security reporting form provided through the official XPScerpto website, `xpscerpto.eu`, if available;
3. a private maintainer security contact listed by the project;
4. a private message to a maintainer requesting a secure reporting channel, without including exploit details in the first public message.

Do not open a public issue, public pull request, discussion thread, public website form, public comment, or social media post containing exploit details, secret material, reproduction steps for an active vulnerability, or instructions that would enable misuse.

If no private channel is immediately available, send only a minimal public request such as:

```text
I would like to report a security issue privately. Please provide a secure contact channel.
```

If the official website provides a public contact form but not a clearly private security channel, use it only to request a secure reporting channel. Do not include exploit details until a private channel is established.

## 4. What to include in a report

A useful security report should include as much of the following as possible:

- affected component or file path;
- affected commit, branch, tag, package, or source tree identifier;
- vulnerability class;
- impact description;
- prerequisites or required configuration;
- minimal reproduction steps;
- expected behavior;
- observed behavior;
- logs, traces, sanitizer output, or test output;
- whether the issue affects authority, boundaries, cryptography, memory safety, CI, evidence, documentation, or release claims;
- whether a workaround exists;
- whether the issue appears exploitable remotely, locally, through build inputs, through runtime inputs, through tests, through tools, or through documentation claims.

Do not include real user secrets, production keys, private data, tokens, credentials, or sensitive third-party information in the report. Use synthetic examples whenever possible.

## 5. Examples of security-relevant reports

The following are examples of issues that should be reported privately when they expose sensitive details or active exploit paths:

- secret key, private key, seed, token, or credential exposure;
- invalid authentication or verification behavior;
- cryptographic parameter validation failure;
- invalid key, context, session, provider, or authority binding;
- unsafe platform admission behavior;
- bypass of `platform.api`;
- unauthorized access to platform internals;
- unauthorized hardware authority or routing behavior;
- unauthorized SIMD dispatch;
- memory safety errors;
- out-of-bounds access;
- use-after-free;
- data races affecting protected behavior;
- incorrect cryptographic behavior in any domain;
- unsafe fallback from governed to ungoverned execution;
- import of protected internals by unauthorized modules;
- CI, test, sanitizer, or evidence manipulation that creates false acceptance;
- stale or forged final flags;
- documentation or release notes that claim security, production readiness, sanitizer truth, or subsystem acceptance without evidence.

This list is not exhaustive. Report any issue that could weaken XPScerpto’s security, authority model, evidence chain, protected boundaries, or claim discipline.

## 6. Public issues and pull requests

Do not include vulnerability details in public issues or public pull requests until the issue has been triaged and disclosure has been coordinated.

Public reports are acceptable for general hardening ideas, documentation improvements, non-sensitive build issues, or already-disclosed issues, but they must not include exploit details for an unresolved vulnerability.

A public pull request that fixes a security issue must not reveal enough information to exploit the issue before maintainers have reviewed the disclosure risk.

Security-sensitive pull requests may be delayed, rewritten, split, or moved to a private review process.

## 7. Security triage process

Security reports should be triaged through the following stages:

1. **Receipt** — acknowledge the report when received.
2. **Scope review** — identify affected surfaces, components, configurations, and claims.
3. **Reproduction** — attempt to reproduce the issue on the relevant source tree.
4. **Classification** — classify severity, exploitability, authority impact, boundary impact, evidence impact, and release impact.
5. **Containment** — identify immediate mitigations or claim-boundary corrections if needed.
6. **Repair** — implement a fix without weakening governance, tests, boundaries, or evidence.
7. **Validation** — run relevant tests, negative tests, CI gates, and sanitizer runtime where applicable.
8. **Evidence** — preserve logs, final flags, blockers, and review records.
9. **Disclosure** — coordinate public disclosure when appropriate.
10. **Regression protection** — add or update tests so the issue cannot silently return.

A security issue is not closed merely because a patch was written. It is closed only when the fix is validated within its claimed scope.

## 8. Severity classification

Severity should be assigned based on impact, exploitability, affected assets, and authority scope.

### Critical

A critical issue may allow compromise of secret material, private keys, authentication guarantees, cryptographic correctness, authority admission, release integrity, or production-facing security claims.

Examples may include:

- key or credential leakage;
- authentication bypass;
- exploitable memory corruption in a protected path;
- unauthorized execution authority;
- false production-ready or security-clean release state;
- cryptographic behavior that invalidates a claimed security property;
- release artifact substitution with plausible evidence.

### High

A high issue may compromise a protected subsystem, bypass a major boundary, corrupt security-sensitive state, or allow serious misuse under realistic conditions.

Examples may include:

- context or key-binding failures;
- unauthorized hardware dispatch in protected execution;
- bypass of `platform.api`;
- sanitizer-detectable memory safety issue in a protected domain;
- evidence manipulation that could lead to false acceptance;
- CI or build graph manipulation that hides required tests.

### Medium

A medium issue may weaken validation, create unsafe edge behavior, expose non-secret internal state, or allow a boundary violation with limited impact.

Examples may include:

- incomplete validation for uncommon parameter sets;
- misleading but non-release documentation claims;
- test-only helper leakage without production use;
- denial-of-service through malformed inputs in a non-critical path;
- incomplete error handling that still fails closed.

### Low

A low issue may include minor hardening gaps, incomplete diagnostics, non-sensitive documentation errors, or validation improvements with limited immediate risk.

Severity may be adjusted after reproduction and review.

## 9. Security-sensitive changes

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
- module boundaries;
- public API ownership;
- protected surfaces;
- tests that enforce security behavior;
- CI gates;
- sanitizer configuration or execution;
- evidence generation;
- final flags;
- release notes or security claims.

Security-sensitive changes require careful review, targeted tests, and evidence appropriate to their scope.

A security-sensitive change must not be rushed through as a convenience fix.

## 10. Repository-wide sensitive areas

The following areas are security-sensitive by default:

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
tests/
tools/
docs/
.github/
CMakeLists.txt
```

This list is not exhaustive. A change outside these paths may still be security-sensitive if it affects authority, validation, evidence, or claims.

## 11. Evidence required for security fixes

A security fix should include evidence appropriate to the issue.

Depending on scope, evidence may include:

- focused reproduction before the fix;
- focused passing test after the fix;
- negative tests for invalid or adversarial inputs;
- regression tests for the issue;
- CTest logs;
- sanitizer runtime logs;
- CI logs;
- final flags;
- source tree identifiers;
- patch or diff records;
- exact blocker reports if incomplete.

Evidence must match the claim.

A fix that only builds must not be described as fully validated.

A sanitizer configuration must not be described as sanitizer clean without runtime execution.

A target-only pass must not be described as full graph acceptance.

A subsystem-specific pass must not be described as repository-wide security acceptance.

## 12. Cryptographic security changes

Cryptographic changes require conservative review across all cryptographic domains.

This includes, where present:

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

A cryptographic change should explain:

- what security property is affected;
- what key, parameter, context, encoding, arithmetic, lifecycle, or verification behavior changed;
- what invalid behavior was rejected;
- what tests prove the corrected behavior;
- what negative tests were added;
- what claims remain out of scope.

Cryptographic fixes must not rely on metadata-only checks, test-only shortcuts, or references that do not measure the claimed production behavior.

Partial acceptance of one cryptographic subsystem must not be presented as global cryptographic acceptance.

## 13. FHE and advanced cryptographic subsystem changes

Some cryptographic subsystems, including FHE, may require additional claim discipline because their correctness depends on many interacting stages.

For FHE, sensitive areas may include:

- BFV behavior;
- CKKS behavior;
- bootstrap behavior;
- encoding and decoding;
- encryption and decryption;
- key generation;
- key switching;
- relinearization;
- Galois and rotation behavior;
- NTT and RNS arithmetic;
- scale, level, modulus, and error behavior;
- public API surface ownership;
- surface guards;
- test-only versus production behavior.

This section does not make the security policy FHE-specific. It records that FHE is one high-risk subsystem among several protected areas.

A BFV fix does not imply CKKS acceptance.

A CKKS fix does not imply bootstrap completion.

A bootstrap-stage fix does not imply FHE production readiness.

An FHE fix does not imply repository-wide production readiness.

Claims must remain scoped to the exact gate and evidence that established them.

## 14. Authority and boundary security changes

A change that affects authority or module boundaries must explain:

- what authority is required;
- where the authority comes from;
- what boundary is touched;
- what import or public surface changed;
- whether the change creates a new authority path;
- whether the change bypasses `platform.api`;
- whether the change allows domain-local probing;
- whether the change introduces an alias, shim, adapter, or compatibility path;
- how the boundary is tested.

Authority fixes must not be made by hiding the boundary violation.

A boundary violation should be repaired by restoring the proper boundary, creating a reviewed admitted surface, or failing closed with an exact blocker.

## 15. Memory safety and sanitizer policy

Memory safety issues should be treated seriously, especially in protected domains.

Relevant evidence may include:

- AddressSanitizer runtime logs;
- UndefinedBehaviorSanitizer runtime logs;
- ThreadSanitizer runtime logs;
- focused reproduction;
- regression tests;
- bounds tests;
- lifetime tests;
- concurrency tests.

Sanitizer truth requires runtime execution. A sanitizer build or sanitizer configuration is not sanitizer truth.

If sanitizer execution fails, times out, crashes, or cannot be completed, the result must identify the exact blocker and must not be described as clean.

## 16. Build, CI, and evidence security

Build, CI, and evidence behavior are security-sensitive.

A change may be security-relevant if it:

- changes CMake target materialization;
- changes CTest labels;
- disables or filters tests;
- changes sanitizer flags;
- changes generated evidence;
- changes final flags;
- changes release packaging;
- changes CI workflow conditions;
- changes artifact upload behavior;
- changes documentation generated from evidence.

Required posture:

- configure success must not be called build success;
- build success must not be called test success;
- sanitizer configuration must not be called sanitizer clean;
- target-only execution must not be called full graph truth;
- partial subsystem success must not be called global acceptance;
- missing evidence must remain visible.

## 17. Disclosure policy

XPScerpto follows coordinated disclosure.

Public disclosure should occur only after:

- maintainers have received the report;
- the issue has been triaged;
- the affected scope is understood;
- a fix or mitigation plan exists, where feasible;
- disclosure timing has been coordinated with the reporter when appropriate.

Do not publish exploit details while a vulnerability is unresolved unless there is a clear safety or public-interest reason and maintainers have been given a reasonable opportunity to respond.

The project may publish an advisory, release note, or security notice when disclosure is appropriate.

## 18. Embargo and handling expectations

During private handling:

- keep exploit details limited to people who need them;
- avoid unnecessary copying of secret material or proof-of-concept code;
- avoid public references that reveal the issue;
- preserve evidence securely;
- avoid changing the claim boundary without review;
- do not merge a misleading partial fix as full security closure.

An embargo is not a reason to hide blockers from maintainers or reviewers. It is a reason to control disclosure of exploit details.

## 19. Credit and attribution

XPScerpto may credit reporters who responsibly disclose vulnerabilities, subject to reporter preference and disclosure safety.

A reporter may request:

- public credit;
- anonymous credit;
- no credit.

Credit does not imply endorsement of the project, production readiness, or acceptance of unrelated claims.

## 20. Security advisories and release notes

Security advisories and release notes must be precise.

They should include:

- affected scope;
- impact summary;
- fixed versions or fixed commits, if applicable;
- mitigation guidance, if applicable;
- validation evidence summary;
- remaining limitations or blockers, if relevant;
- claim boundaries.

They must not overclaim.

A security advisory must not describe the entire project as secure or production-ready unless the relevant gates prove that broader claim.

## 21. Handling false positives

A report may be classified as not exploitable, not security-relevant, out of scope, duplicate, already fixed, or requiring more information.

A false positive classification should explain the reason without dismissing the reporter’s effort.

If a report reveals a documentation ambiguity, test gap, or hardening opportunity, it may still result in an improvement even if it is not a vulnerability.

## 22. Out-of-scope reports

The following reports may be out of scope unless they demonstrate a concrete security impact:

- generic scanner output without analysis;
- reports against unsupported or modified local forks;
- missing security headers on non-production demonstration pages;
- denial-of-service claims without realistic impact;
- social engineering scenarios outside the repository’s control;
- vulnerabilities in third-party systems not controlled by the project;
- reports that require a fully compromised maintainer account;
- issues based only on documentation goals rather than implemented behavior.

Out-of-scope does not mean ignored. It means the report may not be treated as a vulnerability without a clearer impact path.

## 23. No retaliation

Good-faith security research and responsible disclosure should be treated respectfully.

Reporters must not exploit, exfiltrate, destroy, or publicly disclose sensitive data. Reporters should avoid privacy violations, service disruption, and actions beyond what is necessary to demonstrate the issue.

Maintainers should respond professionally and evaluate reports based on evidence.

## 24. Security review checklist

A security-sensitive review should ask:

- What asset is affected?
- What trust boundary is crossed?
- What authority is required?
- Where does that authority come from?
- Does the change bypass `platform.api`?
- Does the change introduce domain-local probing?
- Does the change affect keys, seeds, randomness, credentials, parameters, or contexts?
- Does the change affect cryptographic semantics?
- Does the change affect memory safety?
- Does the change affect CI, sanitizer, evidence, release notes, or final flags?
- Were negative tests added where appropriate?
- Does the evidence prove the claim?
- Does the documentation claim more than the evidence proves?
- If incomplete, is the exact blocker preserved?

A change that cannot answer these questions is not ready for security acceptance.

## 25. Relationship to other documents

This document should be read after:

```text
START_HERE.md
GOVERNANCE.md
ENGINEERING_PRINCIPLES.md
AUTHORITY_MODEL.md
ARCHITECTURE.md
MODULE_BOUNDARIES.md
THREAT_MODEL.md
```

It should be read before:

```text
CI_VALIDATION_GATES.md
REVIEW_POLICY.md
CONTRIBUTING.md
CODEOWNERS
```

`THREAT_MODEL.md` defines the threats.

`SECURITY.md` defines how security reports and security-sensitive changes are handled.

`CI_VALIDATION_GATES.md` should define how validation gates prove or reject claims.

## 26. Current claim boundaries

This security policy does not elevate project status.

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

A security policy is not a security proof. It defines the process and discipline required to handle security correctly.

## 27. Final rule

The final security rule is:

```text
do not disclose exploit details publicly before triage
do not weaken authority to fix security
do not weaken tests to claim security
do not claim security without evidence
do not let one subsystem’s result speak for the whole repository
```

XPScerpto security work is accepted only when the fix, tests, evidence, documentation, and claim boundaries remain aligned.
