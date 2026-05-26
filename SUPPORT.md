# XPScerpto Support

**Status:** support policy  
**Scope:** general support channels, non-security bug reports, build and installation help, documentation questions, feature requests, public-support boundaries, security-report separation, and support claim discipline

XPScerpto is a governed sovereign cryptographic infrastructure platform. This document explains how to request support, where to report non-security issues, what information to include, and when to use the private security process instead of public support channels.

This document is not a validation report, security proof, production-readiness claim, or acceptance artifact. It defines support expectations and safe communication boundaries.

## 1. Purpose

The purpose of this document is to help users, contributors, reviewers, and integrators ask for help in the right place with the right information.

Support requests should be clear, scoped, and safe to discuss publicly.

Security-sensitive reports must not be posted in public support channels.

## 2. Security reports

Do not report vulnerabilities through public support channels.

Use `SECURITY.md` for:

- vulnerabilities;
- exploit details;
- private keys or secret material exposure;
- credential, token, or seed leakage;
- cryptographic correctness failures with security impact;
- authority bypasses;
- protected-surface bypasses;
- unsafe fallback behavior;
- sanitizer findings in protected paths;
- CI, evidence, or release manipulation with security impact;
- any issue that could enable misuse if disclosed publicly.

If you are unsure whether an issue is security-sensitive, treat it as security-sensitive and follow `SECURITY.md`.

Do not include exploit details, secret material, private logs, credentials, tokens, or reproduction steps for an active vulnerability in a public issue, public pull request, public discussion, public website form, or social media post.

## 3. Official project information

Official project information may be available through:

```text
https://xpscerpto.eu
```

If the official website provides a contact option, it may be used for general project contact or to request a private security-reporting channel.

If the website contact mechanism is public or not clearly marked as private security reporting, do not include exploit details. Use it only to request a secure reporting channel.

## 4. General support channels

Use general support channels for non-security topics such as:

- build questions;
- installation questions;
- toolchain questions;
- documentation clarification;
- non-sensitive bug reports;
- feature requests;
- contribution workflow questions;
- CI usage questions;
- test execution questions;
- repository navigation questions.

Depending on what is enabled for the project, appropriate support channels may include:

- GitHub Issues for non-security bugs and documentation issues;
- GitHub Discussions for questions, ideas, and design discussion;
- the official website for project information or contact options;
- pull requests for proposed fixes with clear evidence.

Do not use public support channels for vulnerability disclosure.

## 5. Bug reports

A good non-security bug report should include:

- affected component or file path;
- source tree, branch, tag, commit, or package identifier;
- operating system and version;
- compiler and version;
- CMake version, if relevant;
- build type and important configuration flags;
- exact command executed;
- expected behavior;
- observed behavior;
- minimal reproduction steps;
- relevant logs;
- whether the issue is deterministic or intermittent;
- whether the issue affects a protected surface;
- whether any workaround is known.

If the bug may expose secrets, bypass authority, affect cryptography, or reveal exploit details, do not file it publicly. Use `SECURITY.md`.

## 6. Build and installation help

For build or installation help, include:

- operating system;
- shell or terminal environment;
- compiler and version;
- CMake version;
- generator used, if known;
- build command;
- full error message;
- first compiler, linker, configure, or CTest failure;
- whether the error happens in Release, Debug, sanitizer, or another configuration;
- whether this is a clean build directory.

Avoid screenshots when text logs are available. Text logs are easier to search, quote, and diagnose.

Do not paste extremely large logs directly into an issue. Attach a file or provide the smallest relevant excerpt, including the first error.

## 7. Test and CI help

For test or CI help, include:

- command executed;
- build directory;
- CTest label or target, if any;
- number of tests run;
- number of tests failed;
- first failing test;
- relevant test output;
- whether tests were skipped or disabled;
- whether the failure is local-only or also appears in CI.

Use precise language.

Do not say “all tests pass” unless the full claimed test scope actually ran.

Do not say “sanitizer clean” unless sanitizer runtime executed and the relevant logs support that claim.

## 8. Documentation questions

Documentation questions are welcome.

A documentation issue should explain:

- which document is unclear;
- which section is affected;
- what wording is confusing;
- what interpretation seems wrong;
- whether the issue affects claims, security posture, architecture, authority, boundaries, or validation.

Documentation that affects project claims may require stricter review.

Do not request documentation that claims production readiness, security hardening, sanitizer cleanliness, full graph success, or subsystem acceptance unless the relevant evidence exists.

## 9. Feature requests

Feature requests should include:

- the problem being solved;
- the proposed behavior;
- expected users or use cases;
- affected subsystem, if known;
- security or authority impact, if any;
- testing expectations;
- compatibility concerns;
- whether the feature changes public API behavior.

Feature requests that affect cryptography, authority, platform behavior, hardware routing, SIMD dispatch, CI gates, public surfaces, or release claims may require deeper discussion before implementation.

A feature request is not an acceptance claim.

## 10. Contribution support

For help preparing a contribution, read:

```text
CONTRIBUTING.md
REVIEW_POLICY.md
CI_VALIDATION_GATES.md
MODULE_BOUNDARIES.md
```

A contribution support request should explain:

- what you are trying to change;
- what files are affected;
- what tests you ran;
- what evidence you have;
- what is blocked;
- what claim you are not making.

If the contribution is security-sensitive, follow `SECURITY.md`.

## 11. What not to include publicly

Do not include the following in public support channels:

- private keys;
- secret keys;
- seed material;
- credentials;
- tokens;
- passwords;
- private user data;
- confidential logs;
- exploit details;
- active vulnerability reproduction steps;
- undisclosed security findings;
- sensitive infrastructure details;
- real production secrets;
- private customer or third-party data.

Use synthetic examples whenever possible.

If sensitive information was posted publicly by mistake, remove it if possible and notify maintainers through the security process.

## 12. Unsupported requests

The project may decline or redirect requests that are outside the support scope.

Examples include:

- requests for production-readiness guarantees without validation evidence;
- requests to bypass governance;
- requests to weaken tests;
- requests to remove security checks for convenience;
- requests to ignore module boundaries;
- requests to provide exploit instructions;
- requests to debug private infrastructure without enough information;
- requests based on unsupported forks or heavily modified trees;
- requests that require access to secrets or private data;
- requests that ask maintainers to certify security without an applicable gate.

Declining a request does not imply hostility. It preserves project integrity and safe support boundaries.

## 13. Response expectations

XPScerpto may be maintained by a small team. Response time may vary.

Support is best-effort unless a separate written support agreement exists.

Maintainers may ask for:

- exact commands;
- full error output;
- a smaller reproduction;
- affected commit or source tree;
- logs;
- environment details;
- clarification of whether the issue is security-sensitive.

A request with clear evidence is easier to handle than a broad or vague report.

## 14. Claim boundaries in support discussions

Support discussions must not elevate project status.

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

A support answer may help diagnose or repair an issue. It is not a production-readiness claim, security proof, or acceptance artifact unless the relevant gate and evidence explicitly establish that claim.

## 15. Good support request template

Use this structure when opening a non-security support issue:

```markdown
## Summary

Describe the issue or question briefly.

## Scope

Which component, file, command, or document is affected?

## Environment

- OS:
- Compiler:
- CMake:
- Build type:
- Branch / commit / package:

## Command

```text
paste the exact command here
```

## Expected behavior

What did you expect to happen?

## Observed behavior

What happened instead?

## Logs

Paste the smallest useful log excerpt, including the first error.

## Security sensitivity

Is this security-sensitive?

- [ ] No, this is safe for public discussion.
- [ ] Unsure. I will follow SECURITY.md instead of posting details publicly.

## Additional context

Add any relevant information.
```

Do not use this template for vulnerabilities. Use `SECURITY.md`.

## 16. Relationship to other documents

This document should be read with:

```text
START_HERE.md
CONTRIBUTING.md
SECURITY.md
CI_VALIDATION_GATES.md
REVIEW_POLICY.md
```

`SUPPORT.md` explains how to ask for help.

`SECURITY.md` explains how to report vulnerabilities.

`CONTRIBUTING.md` explains how to prepare changes.

`CI_VALIDATION_GATES.md` explains how validation claims are proven.

`REVIEW_POLICY.md` explains how changes are reviewed.

## 17. Final support rule

The final support rule is:

```text
ask publicly for general help
report privately for security issues
include evidence for technical claims
do not post secrets
do not overclaim project status
```

XPScerpto support is most effective when requests are precise, safe to discuss, and aligned with the project’s evidence and claim discipline.
