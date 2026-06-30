#platform maintainer guide

The safest way to change `xps.platform` is to ask one question before touching code: does this change create, move, or interpret platform truth? If it creates or observes truth, it belongs below the platform authority boundary. If it interprets sealed hardware evidence, it likely belongs in `xps.hw`. If it is an algorithm choice in crypto, FHE, or SIMD, it should consume the appropriate authority object instead of probing the host.

This guide is for future maintainers who need to add providers, facts, evidence, or tests without weakening the boundary.

## How to work safely in xps.platform

A safe platform change keeps three properties intact.

First, raw host access stays inside provider leaves or private authority modules. Do not move OS calls into consumer code just because it is convenient.

Second, public API returns typed values, statuses, reports, or sealed facts. It does not expose raw backend storage, provider internals, private bridge writers, private constructors, or OS handles.

Third, evidence remains labeled and registered. A test fixture, replay fixture, telemetry snapshot, or stdout log may help development, but it cannot be promoted into live platform evidence.

## Adding a provider safely

A provider is not just a set of function pointers. It is an authority path that says where platform observations came from.

When adding a provider:

1. Keep OS-specific calls in a provider leaf or private authority wrapper. The existing pattern uses exported provider modules and foreign implementation files for hosted OS paths.
2. Give the provider a stable nonzero provider ID and nonempty provider name. Registration rejects zero IDs and malformed providers.
3. Fill the ABI table and capability mask honestly. `backend_abi_compatible` and `backend_caps_consistent` reject incompatible tables and missing hooks.
4. Register through `xps.platform.provider`; do not install raw backend pointers through the runtime bridge.
5. If the provider adds extension records, keep extension payloads valid, non-stack-owned, and compatible with the registry checks.
6. Add provider-control and provider-matrix tests for the normal path, malformed inputs, duplicate semantics, stub separation, and unsupported capabilities.
7. Add negative compile coverage if the new provider introduces a tempting private import or raw authority shortcut.
8. Update source identity and authority registry entries when new files or authority classes are introduced.

A provider is not safe merely because it compiles. It is safe only when registration, activation, fact sealing, evidence labeling, and boundary scans all agree about its role.

## Adding a platform fact safely

A platform fact should be added only when a consumer needs platform truth that cannot be expressed by the existing contract. Before adding a field, decide who observes it, who interprets it, how failure behaves, and whether it is evidence or advisory report data.

A safe fact change normally requires:

- A typed field in the platform fact or hardware-evidence contract.
- A schema or minor-version update when the contract changes in a way consumers must notice.
- A missing and rejected representation. Do not make absence look like authorization.
- Provider-side validation and normalization. Clamp or sanitize values before sealing when the domain has invalid or ambiguous states.
- Source attribution. A fact must say whether it came from an operational provider, bootstrap/control/runtime bridge, test harness, replay fixture, telemetry, or another source class.
- Proof/seal computation updates so stale or mismatched values are rejected.
- Hardware adapter updates if the fact crosses into `xps.hw`.
- Consumer tests, negative compile tests, and evidence updates.

Do not add a convenience constructor that lets a caller manufacture an authorized fact. The existing `PlatformFacts` and `PlatformHardwareEvidence` types deliberately keep their raw constructors private and route authorized construction through the fact authority.

## Updating evidence emission

Evidence changes are risky because they can accidentally make a weak signal look authoritative. Keep these rules intact:

- Provider mode must be visible where provider identity matters.
- Telemetry, test-harness, replay fixture, stdout/stderr, command output, FCC output, model output, and UI status are not operational platform evidence.
- Mock or fake provider evidence may support deterministic tests, not live provider closure.
- Replay evidence may support replay behavior, not live runtime closure.
- Missing evidence is a blocker, not a pass.
- Generated evidence must be registered, scoped to the right root, and included in the admission chain when it is used for a claim.

When adding an evidence file, update the source identity or evidence manifest required by the platform policy. When changing an existing evidence file, do not copy a prior report’s verdict unless the corresponding build/test/admission chain was rerun or the report explicitly remains valid for the changed source.

## Changing public surfaces

Before exporting a new symbol from `xps.platform.api`, `xps.platform.abi`, `xps.platform.facts`, or `xps.platform.facts.value`, check whether the symbol leaks authority.

A public symbol is appropriate when it exposes a value, status, report, sealed fact, validated evidence object, or capability query. A public symbol is not appropriate when it exposes provider storage, raw hook mutation, runtime bridge writers, private fact constructors, private seal internals, OS handles, or raw host structures.

If a new public API is needed:

1. Define the failure result first.
2. Decide whether the API is operational authority, advisory reporting, or a consumer convenience wrapper.
3. Keep provider and runtime mutation below the control seam.
4. Add tests for unsupported, stub, malformed, and provider-mismatch cases.
5. Add negative compile coverage for the private bypass that the new API might tempt a caller to use.
6. Update documentation only after the source and tests define the actual behavior.

## Changing private implementation

Private implementation may change, but it must remain private. The easiest way to break the boundary is to move a useful helper into a header or module that consumers can import.

Do not publish:

- `BackendVTable` mutation paths or active runtime storage.
- Runtime bridge writer functions.
- Provider foreign implementation helpers.
- Private authority wrappers.
- Private fact-seal constructors.
- Telemetry custody hooks as operational evidence.

The retired private authority headers are intentionally hard tombstones. Re-enabling them as normal headers would weaken the boundary even if tests still pass locally.

## Updating tests without weakening them

Tests should become stricter when the API changes. They should not be rewritten to accept weaker behavior.

For positive tests, cover the supported public path and the explicit failure path. For provider tests, include valid providers, malformed providers, conflicting duplicates, stub behavior, and unsupported capabilities. For fact tests, include missing, rejected, unauthorized, forbidden source, mismatched proof, and successful authorized cases.

For negative compile tests, keep the failure reason meaningful. The existing harness distinguishes authority rejection from accidental module-resolution failure; preserve that property. A negative test that “passes” because the compiler could not find any module does not prove the boundary.

## Updating source identity and manifests

Platform documentation and source files are part of source identity. If you add, remove, or modify a platform file that the policy treats as reviewed source, update `cmake/platform_source_identity_manifest.cmake` with the correct SHA-256, owner, kind, and class. If you introduce a new raw authority path, update `cmake/platform_authority_registry.cmake` and the corresponding policy tests.

Do not weaken the manifest to make a change easier. Source identity is the mechanism that lets an admission reviewer know what source was actually reviewed.

## Practical review questions

Use these questions before approving a change:

| Question | Safe answer |
| --- | --- |
| Does a non-platform module read the OS, CPU, filesystem, registry, compiler ISA macros, or runtime host state directly? | No. It consumes sealed platform facts or hardware interpretation. |
| Does a public API expose raw provider internals? | No. It returns values, statuses, reports, or sealed facts. |
| Can a caller manufacture an authorized fact? | No. Authorized construction remains behind the fact authority. |
| Can fake, test, replay, or telemetry evidence satisfy live closure? | No. The source and reports keep those modes separate. |
| Is missing evidence treated as success? | No. It is a blocker. |
| Did source identity and authority registry entries change with the files? | Yes, when applicable. |
| Did the change require new negative compile coverage? | Yes, when it opens a tempting bypass path. |
| Did the documentation claim more than tests and evidence prove? | No. Gaps remain labeled as gaps. |

