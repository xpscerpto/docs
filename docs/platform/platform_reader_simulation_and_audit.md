# xps.platform reader simulation and documentation audit

This document records the final editorial check for the platform documentation round. It is not a substitute for tests or admission evidence. Its purpose is to show whether the documentation set can stand on its own for the readers it was written for.

## Documentation structure chosen

The platform documentation now uses four canonical documents plus this audit record:

- `README.md` is the entry point and reader map.
- `platform_authority_boundary_guide.md` explains the module, boundary, public/private surfaces, provider model, fact flow, relationships, failures, examples, and source alignment.
- `platform_evidence_and_closure_guide.md` explains evidence, closure gates, provider-mode claims, admission blockers, and allowed/non-allowed claims.
- `platform_maintainer_guide.md` explains how to add providers, facts, evidence, tests, and public APIs safely.
- `platform_reader_simulation_and_audit.md` records the reader simulation, consistency audit, and editorial verdict.

The former numbered paths remain as short compatibility pages because existing repository reports and generated graphs point to those filenames. They are not the canonical documentation structure. They prevent historical links from going dead while directing readers to the new sections.

This structure was chosen because the old numbered notes made the reader hop across many small files before understanding the central problem. The new structure starts with the reader’s confusion, explains the authority model in one narrative, and separates operational evidence from maintenance advice. That is a better fit for `xps.platform`, whose main difficulty is not API volume but authority clarity.

## Source areas reviewed

The review covered:

- Public platform modules: `platform.api.ixx`, `platform.abi.ixx`, `facts.ixx`, and `facts.value.ixx`.
- Private platform implementation: provider registry, runtime, runtime bridge, control, internal helpers, telemetry custody, telemetry hooks, OS/provider foreign files, and raw authority modules.
- Provider implementations and bootstrap wrappers for Linux, Windows, macOS, Android, and POSIX paths.
- Hardware ingress through `xps.hw.api` and its handling of `PlatformFacts` plus `PlatformHardwareEvidence`.
- Downstream FHE, SIMD, and selected crypto use of hardware authority rather than private platform providers.
- Platform tests, provider tests, platform/HW boundary tests, and negative compile harnesses.
- Platform-related CMake rules, source identity manifest, authority registry, import-policy scan, and stdlib/authority policy scan.
- Existing platform reports and evidence files under `reports/` and `evidence/platform/`.
- Searches for connected governance, court, FCC, replay, and receipt-style paths. Court and FCC-related policy/report paths were found; replay appears as an evidence-source label and blocker, while no separate receipt subsystem was found as a platform consumer during this review.
- Existing platform documentation, whose numbered content was retired and replaced by compatibility pages because the old structure duplicated material and made publication quality harder to verify.

## Reader simulation

### New engineer

Clear after reading: the engineer can identify `xps.platform` as the OS-facing authority boundary, find the public API surfaces, understand why private providers and runtime-control seams are not consumer APIs, and follow the raw-observation-to-sealed-fact path.

Unclear on first pass: the difference between “platform observes hardware-related evidence” and “hardware owns hardware policy” needed a concrete example. The final guide now includes the `/proc/cpuinfo` example and the platform-to-HW diagram.

Improved: the reader map points directly to the authority guide and maintainer guide instead of asking the reader to infer the right order.

Still source-dependent: exact provider behavior remains OS-specific and must be checked in the provider leaf being modified.

### Security reviewer

Clear after reading: the trust boundary, forbidden access paths, private implementation surfaces, provider-mode separation, telemetry limits, and fail-closed behavior are stated with source anchors.

Unclear on first pass: “mock,” “test,” and “replay” could have sounded like equivalent provider modes. The evidence guide now says precisely what the source implements: fake/test providers support tests; replay and test-harness labels are recognized and rejected as operational evidence; no general production replay provider leaf was found in this review.

Improved: boundary diagrams now distinguish consumer access to public APIs from private OS authority.

Still source-dependent: future modules outside the reviewed graph require the same policy-scan coverage before the boundary claim can be extended to them.

### External technical reader

Clear after reading: the module exists because host facts are dangerous when every subsystem can gather them independently. The reader can see why platform is separate from hardware, crypto, FHE, and SIMD, and why advisory telemetry is not proof.

Unclear on first pass: whether existing reports imply production readiness. The evidence guide now states that existing platform reports deny production readiness, full-repository closure, security completion, and external audit completion.

Improved: the documentation avoids inflated claims and treats the repository’s reports as bounded evidence rather than marketing material.

Still source-dependent: the external reader would need the repository and registered evidence artifacts to independently verify the closure reports.

### Closure or admission reviewer

Clear after reading: closure is presented as a gate chain, not as a single green status. The reviewer can identify configure, build, test, negative compile, provider separation, source identity, consumer boundary, fail-closed, and admission gates.

Unclear on first pass: whether this documentation round reran sanitizer and release evidence. The final evidence guide now says it reviewed existing reports but did not rerun full release, ASAN/UBSAN, or TSAN builds.

Improved: the allowed/forbidden claims table separates module closure claims from production/security/external-audit claims.

Still source-dependent: a fresh admission decision must use current registered evidence for the exact source revision under review.

### Future maintainer

Clear after reading: the maintainer can see where to add a provider, how to add a fact, why manifests must change, and what not to expose publicly.

Unclear on first pass: whether a documentation-only change needed manifest work. The maintainer guide now states that platform documentation is source-identity tracked and that manifest hashes must be updated when platform docs change.

Improved: practical review questions now catch the most likely boundary mistakes.

Still source-dependent: exact target names and CI invocations may vary by build environment; the guide gives the platform-unit workflow and points to CMake options rather than pretending one command covers every admission mode.

## Consistency audit

| Audit item | Result |
| --- | --- |
| Documentation entry point exists. | `docs/platform/README.md` is the entry point and reader map. |
| Diagrams match source-reviewed relationships. | Diagrams show OS/provider/platform/facts/HW/consumer/evidence flow, provider-mode separation, fact sealing, and closure gates. No diagram claims raw consumer access is allowed. |
| Public API descriptions match exports. | Public descriptions align with `xps.platform.api`, `xps.platform.abi`, `xps.platform.facts`, and `xps.platform.facts.value`. |
| Private implementation is separated. | Runtime bridge, control, provider internals, telemetry custody, raw authority modules, private seal internals, and provider foreign files are documented as private. |
| Provider model avoids overclaiming. | Hosted providers, fake/test providers, replay/test labels, telemetry, and invalid provider behavior are distinguished. Missing production replay-provider leaf is marked as a source limitation. |
| Fact flow is source-aligned. | The guide covers validation, normalization, source attribution, policy checks, sealing/proof, rejected/missing values, and consumer use. |
| Boundary rules match enforcement. | Import policy, tombstoned headers, runtime bridge guard, private constructors, negative compile harness, and court blockers are cited as enforcement mechanisms. |
| Evidence and closure rules are not overstated. | Existing reports are summarized as existing artifacts; this documentation round does not claim to have rerun full release or sanitizer evidence. |
| Failure behavior is not overstated. | Time and path behavior are described narrowly: path/report custody is enforced by policy tooling, while unavailable time must be treated as non-authoritative unless covered by evidence. |
| Terminology is consistent. | The docs use `xps.platform`, platform authority boundary, sealed platform fact, platform provider, provider mode, evidence receipt, fail-closed, consumer module, admission gate, public API, private implementation, source identity, manifest registration, and negative compile test consistently. |
| Internal links are valid. | The documentation set links only to files created in this directory. Source file paths are written as code references rather than fragile links. |
| Stale documentation corrected. | The old numbered platform content was retired. Compatibility pages remain only to keep historical report links resolvable and to send readers to the canonical sections. |
| Closure and production claims are bounded. | The docs reject production readiness, full-repository closure, complete security, and external audit claims unless separate registered evidence supports them. |

## Claims verified, partial, and gap-aware

### Verified by reviewed source or registered reports

- `xps.platform` owns the OS-facing authority boundary for platform facts.
- Public API surfaces expose values, reports, statuses, sealed facts, and hardware evidence rather than raw OS handles.
- Fact authorization requires schema, source, identity, consistency, and proof checks.
- Telemetry, test-harness, and replay fixture sources are rejected as operational evidence.
- Provider registration rejects malformed provider identity, ABI, capability, and extension-table cases.
- Runtime backend mutation is controlled by platform runtime/control seams and guard/freeze state.
- `xps.hw` consumes sealed platform facts and hardware evidence through its authorized input path.
- Known FHE/SIMD paths consume hardware authority rather than private platform providers.
- Negative compile tests cover private constructor, private header, raw authority, provider/control/bootstrap, consumer bypass, and FCC-style evidence shortcuts.
- Existing platform reports deny full-repository closure, production readiness, security completion, and external audit completion.

### Partially implemented or narrowly implemented

- Path policy is enforced strongly for report/source custody in CMake policy scans. The documentation does not claim a general runtime filesystem sandbox for every provider operation.
- Time authority exists as a private platform authority wrapper, but this review did not find a broad public sealed-time fact contract. The documentation treats unavailable time as non-authoritative rather than claiming a completed time-fact subsystem.
- Mock/provider separation is implemented through fake/test provider paths and provider-matrix tests. A general production “mock provider mode” is not documented as implemented.
- Replay is represented as a source/evidence label and is rejected for operational evidence. A standalone production replay provider leaf was not found.
- Existing reports state sanitizer and closure results, but this documentation round did not rerun those build modes.

### Gaps that remain source gaps, not documentation gaps

- A stronger claim that all future modules avoid OS probing requires continued import-policy and negative-test coverage as new modules appear.
- A stronger live-provider closure claim for a specific host environment requires current registered live-provider evidence for that environment.
- A stronger runtime path/time policy claim would require source and tests showing a public sealed path/time fact model or equivalent runtime enforcement.
- A production readiness claim requires evidence beyond the platform module closure reports reviewed here.
- External audit completion requires external audit evidence; it cannot be inferred from repository-internal reports.

## Human-readability pass

The final pass checked whether the documents read like an engineer explaining a real boundary rather than a generated checklist.

| Check | Result |
| --- | --- |
| Introduction starts with the reader’s confusion. | The entry point and authority guide start with why direct host facts are dangerous. |
| Each major section answers a real question. | Sections are framed around what the module is, what it is not, who may access the OS, how facts are made, how evidence works, and how to modify safely. |
| Marketing language avoided. | The docs do not describe the module as “robust,” “comprehensive,” or production-ready. |
| Repetition reduced. | Repeated boundary slogans were consolidated into diagrams, examples, and evidence rules. |
| Diagrams are explained in normal language. | Each diagram has a preceding explanation and a concrete rule after it. |
| Examples are practical. | Examples cover `/proc/cpuinfo`, fake provider closure, missing facts, FHE autotuning, and private provider implementation leakage. |
| Reader is guided instead of overwhelmed. | The README provides a reader map; detail is split into architecture, evidence, and maintenance documents. |
| External trust posture is credible. | Gaps and non-claims are explicit, and existing reports are not inflated into production or audit claims. |

## Final editorial verdict

`DOCUMENTATION_COMPLETE_WITH_SOURCE_GAPS`

The documentation set is coherent, diagram-supported, source-aligned, and suitable for publication as the platform boundary manual. The verdict is gap-aware rather than absolute because this round did not rerun the full build/sanitizer/admission chain and because some concepts requested by policy language, such as production replay provider mode and broad runtime path/time policy, are either represented only as rejection vocabulary or enforced more narrowly than a reader might assume.

