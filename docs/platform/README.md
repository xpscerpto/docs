# xps.platform documentation

Operating-system facts are easy to misuse when every module can read them directly. CPU leaves, runtime topology, filesystem state, clocks, entropy sources, telemetry, and provider identity all look like simple inputs, but they become security-sensitive once they influence cryptographic or hardware decisions. `xps.platform` exists to keep that authority in one place.

The short version is this: `xps.platform` owns host observation. It turns selected raw observations into typed, attributed, sealed platform facts. Other modules consume those facts through narrow contracts; they do not build their own private view of the machine.

This directory is the publication entry point for that boundary. The canonical material is now organized around a smaller set of reader-oriented documents that separate first questions from evidence and maintenance rules. The former numbered paths are retained only as compatibility pages because existing reports and generated graphs refer to them; they point readers to the canonical sections rather than carrying a second copy of the architecture.

## What to read

| Reader need | Start here | Then read |
| --- | --- | --- |
| You are new to XPScerpto and need the shape of the module. | [Platform authority boundary guide](platform_authority_boundary_guide.md) | [Maintainer guide](platform_maintainer_guide.md#how-to-work-safely-in-xpsplatform) |
| You are reviewing security boundaries. | [Authority boundary](platform_authority_boundary_guide.md#the-authority-boundary) | [Evidence and closure guide](platform_evidence_and_closure_guide.md#what-counts-as-platform-evidence) |
| You are checking closure or admission readiness. | [Evidence and closure guide](platform_evidence_and_closure_guide.md) | [Reader simulation and audit](platform_reader_simulation_and_audit.md) |
| You are adding a provider. | [Provider model](platform_authority_boundary_guide.md#provider-model-live-test-replay-and-invalid-sources) | [Adding a provider safely](platform_maintainer_guide.md#adding-a-provider-safely) |
| You are adding a new platform fact. | [How a platform fact is made](platform_authority_boundary_guide.md#how-a-platform-fact-is-made) | [Adding a platform fact safely](platform_maintainer_guide.md#adding-a-platform-fact-safely) |
| You are debugging a failure. | [Failure model](platform_authority_boundary_guide.md#failure-model) | [Evidence effects of failure](platform_evidence_and_closure_guide.md#failure-and-evidence) |
| You are reviewing public API stability. | [Public API and private implementation](platform_authority_boundary_guide.md#public-api-and-private-implementation) | [Changing public surfaces](platform_maintainer_guide.md#changing-public-surfaces) |
| You are checking whether a claim may be made. | [Allowed and forbidden claims](platform_evidence_and_closure_guide.md#allowed-and-forbidden-claims) | [Audit verdict](platform_reader_simulation_and_audit.md#final-editorial-verdict) |

## The module in one page

`xps.platform` is the OS-facing authority boundary for XPScerpto. It is responsible for provider selection, runtime/backend identity, platform fact construction, platform-to-hardware evidence construction, path and report custody checks in the platform policy tooling, and evidence rules that say which observations may support closure.

The public surface is intentionally small. Consumers are expected to use `xps.platform.api`, `xps.platform.abi`, and the value contracts in `xps.platform.facts`. Hardware-facing consumers use `xps.platform.facts.value` only through the allowed hardware boundary. The private surface is where raw authority lives: provider leaves, runtime-control mutation, telemetry custody, raw OS wrappers, and fact-sealing constructors.

`xps.platform` is not a general utility layer. It does not own cryptographic primitives, FHE algorithms, SIMD optimization policy, or hardware policy decisions. It may observe and seal hardware-related evidence; `xps.hw` interprets that evidence. Crypto, FHE, SIMD, and higher modules must not probe the operating system or host runtime directly to create their own platform truth.

## Claim language used in these documents

The documents use four claim levels.

| Claim level | Meaning |
| --- | --- |
| Implemented | The behavior is present in source, build rules, tests, policy scans, or registered reports reviewed during this round. |
| Partially implemented | The design exists, but enforcement is narrower than the wording a reader might expect. The text says what is enforced and what is not. |
| Gap | The repository does not currently prove the stronger claim. The documentation may recommend the behavior, but it does not treat it as completed. |
| Out of scope | The behavior belongs to another module or to a future admission process, not to `xps.platform`. |

This wording matters. A document is not evidence of runtime behavior, a green-looking report is not automatically closure evidence, and a test provider result is not a live-machine result.

## Source areas behind this documentation

The documentation was written from the repository, not from prior chat context. The main source areas reviewed were:

- `include/xps/crypto/platform/` for exported platform APIs, value contracts, providers, runtime/control seams, telemetry custody, and private authority wrappers.
- `include/xps/crypto/hw/` for the hardware ingress that consumes sealed platform evidence.
- `include/xps/crypto/FHE/`, `include/xps/crypto/simd/`, and selected crypto surfaces for downstream use of hardware authority rather than direct platform probing.
- `tests/platform/`, `tests/platform_hw/`, and the platform negative-compile harness for boundary behavior.
- `CMakeLists.txt`, `tests/CMakeLists.txt`, `cmake/CheckPlatformImportPolicy.cmake`, `cmake/CheckPlatformStdlibAuthorityPolicy.cmake`, `cmake/platform_authority_registry.cmake`, and `cmake/platform_source_identity_manifest.cmake` for build, import, source-identity, and authority registration rules.
- `reports/` and `evidence/platform/` for existing registered platform evidence and non-claim boundaries.

## Non-claims

These documents do not claim full-repository closure, production readiness, complete security assurance, or external audit completion. Existing platform reports in the repository explicitly deny those broader claims. The documentation preserves that distinction.

