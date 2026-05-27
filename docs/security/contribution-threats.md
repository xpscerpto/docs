# Contribution Threats

**Status:** security reference  
**Scope:** threat patterns introduced through contribution, review, CI, and documentation

XPScerpto treats contribution as an attack surface. This file gives concrete examples reviewers should look for.

## 1. Authority bypass through runtime code

### Pattern

A patch adds direct hardware probing to a domain module.

Examples:

- CPUID inside SIMD;
- `__builtin_cpu_supports` inside domain code;
- reading `/proc/cpuinfo` to select execution;
- compiler ISA macros used as runtime authority.

### Risk

The domain creates authority outside the platform/HW chain.

### Required response

Reject or require redesign through approved HW/platform routing.

## 2. Dependency insertion

### Pattern

A patch adds a third-party dependency to solve parsing, crypto, threading, CPU detection, logging, or benchmarking.

### Risk

Supply-chain, licensing, reproducibility, audit, timing, and authority risks.

### Required response

Require governance approval and dependency threat analysis. Default is rejection.

## 3. Ownership bypass

### Pattern

A patch adds an alias, shim, adapter, forwarding file, or compatibility surface to make old and new owners appear valid.

### Risk

Canonical ownership becomes ambiguous. Public API consumers may bind to the wrong authority.

### Required response

Reject for protected surfaces unless an RFC approves a bounded compatibility plan.

## 4. Test masking

### Pattern

A patch deletes, disables, renames, excludes, or weakens a failing test.

### Risk

The validation system is made less truthful.

### Required response

Require root-cause repair or coverage transfer proof.

## 5. Sanitizer overclaim

### Pattern

A patch says “sanitizer clean” after configure or build only.

### Risk

Memory, UB, or race issues may remain unexecuted.

### Required response

Require runtime sanitizer logs or change the claim to the exact evidence scope.

## 6. Target-only overclaim

### Pattern

A single test target passes and the PR claims full graph closure.

### Risk

Unbuilt or unexecuted graph paths may still fail.

### Required response

Require full labelled graph evidence or limit the claim.

## 7. Documentation overclaim

### Pattern

A document states that a system is production-ready, fully secure, or fully closed without matching evidence.

### Risk

Users, auditors, and funders may rely on unsupported claims.

### Required response

Rewrite to distinguish intent, implementation, validation, and readiness.

## 8. Fail-open repair

### Pattern

A patch converts a rejecting guard into a permissive fallback to make tests pass.

### Risk

The system becomes unsafe in unknown states.

### Required response

Reject unless an RFC and security review approve the changed failure semantics.

## 9. Graph pollution

### Pattern

A patch adds broad imports or shared headers to repair compile failures quickly.

### Risk

Module ownership and dependency direction become unclear.

### Required response

Require minimal imports, ownership rationale, and graph scan evidence.

## 10. Maintainer checklist

For sensitive PRs, ask:

- What authority did this change consume or produce?
- Did the change add a new path around platform/HW admission?
- Did the change add a dependency?
- Did the change weaken a guard, test, or assertion?
- Does the evidence match the claim?
- Is an RFC or decision record required?
