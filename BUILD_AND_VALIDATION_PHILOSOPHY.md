# XPScerpto Build and Validation Philosophy

**Status:** normative validation philosophy  
**Scope:** build evidence, test evidence, sanitizer evidence, graph validation, runtime truth

XPScerpto validation is evidence-driven. A build is not a security proof. A local test is not full graph truth. A sanitizer configuration is not sanitizer runtime truth. A benchmark is not authority.

## 1. Validation hierarchy

The validation hierarchy is:

```text
source inventory
  ↓
configure evidence
  ↓
build materialization
  ↓
module graph validation
  ↓
policy and import boundary scans
  ↓
target-level tests
  ↓
full labelled test graph
  ↓
sanitizer runtime execution
  ↓
runtime attestation and audit evidence
  ↓
claim review
```

A claim may only reference the highest level actually executed.

## 2. Configure is not build

A successful configure proves that the build system generated a plan. It does not prove that targets compile, link, or execute.

Rejected claim:

```text
Configured with ASAN, therefore ASAN clean.
```

Acceptable claim:

```text
ASAN build completed, ASAN-labelled tests executed, no sanitizer findings observed in the attached runtime log.
```

## 3. Build is not runtime truth

A successful build proves materialization. It does not prove that cryptographic semantics, fail-closed behavior, routing admission, or sanitizer runtime behavior are correct.

## 4. Target pass is not full graph pass

A single target pass may be useful for a local repair. It must not be used to claim:

- full FHE graph pass;
- full repository regression pass;
- sanitizer clean for the repository;
- production readiness;
- coverage closure.

## 5. Sanitizer truth

Sanitizer truth requires executed runtime evidence. For each sanitizer claim, record:

- sanitizer type;
- compiler and version;
- build configuration;
- target/test scope;
- command executed;
- output log;
- failures, timeouts, unsupported cases, or compiler issues.

If sanitizer execution is incomplete, say incomplete. Incomplete evidence is not failure by itself, but it is not clean.

## 6. Runtime attestation truth

Runtime routing or provider changes require evidence that the correct authority path was used. Evidence may include:

- authority capsule or routing record;
- HW evidence snapshot;
- audit ledger output;
- provider admission result;
- negative test proving bypass rejection;
- no local probe scan result.

## 7. Policy scan truth

Policy scans must be scoped and repeatable. A scan should state:

- what it scanned;
- what patterns it rejects;
- what paths are intentionally excluded and why;
- whether the result is advisory or gate-enforced.

A policy scan that relies on a whitelist must justify the whitelist. The preferred model is no exceptions unless governance approves them.

## 8. Benchmark truth

Performance claims must include:

- command and environment;
- input sizes and distributions;
- p50/p95/p99 or equivalent latency data when relevant;
- competitor and baseline definitions;
- authority state, including whether optimized routes were authorized;
- statement that performance did not bypass security constraints.

A benchmark improvement does not justify authority bypass.

## 9. Evidence retention

Sensitive PRs should retain evidence as CI artifacts or repository evidence files when appropriate. Evidence should be named by scope, not by vague success terms.

Good:

```text
ctest-full-label-fhe-2026-05-24.txt
asan-runtime-tests-platform-hw-simd.txt
module-graph-scan-after-simd-route-change.txt
```

Bad:

```text
success.txt
final.txt
clean.txt
```

## 10. Claim discipline

Claims must be exact:

| Evidence | Allowed claim | Disallowed claim |
|---|---|---|
| one target passed | target passed | full graph passed |
| configure succeeded | configure succeeded | build clean |
| sanitizer build succeeded | sanitizer build succeeded | sanitizer runtime clean |
| sanitizer tests executed cleanly | sanitizer clean for executed scope | sanitizer clean for all scopes |
| documentation added | governance documented | production-ready |

## 11. Failure evidence

A failure can be useful when it is classified correctly. Report:

- failed command;
- first failed target or test;
- failure class;
- exact blocker;
- whether production code was affected;
- next required repair.

Do not hide failure behind broad success language.
