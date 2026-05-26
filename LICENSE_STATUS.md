# XPScerpto License Status

**Status:** no open-source license selected  
**Scope:** repository-wide licensing status, public visibility boundaries, permitted review use, prohibited reuse, future licensing, third-party contributions, and relationship to project governance

XPScerpto is currently published for review, evaluation, project visibility, and governance inspection only.

No open-source license has been selected for the repository at this time.

Unless and until a formal license is published by the project owner, no permission is granted to use, copy, modify, redistribute, sublicense, commercially deploy, host, package, fork for reuse, or create derivative works from the XPScerpto source code, documentation, tests, tools, evidence artifacts, or other repository materials.

All rights are reserved by the project owner unless a specific written permission or future license states otherwise.

This document is not a validation report, security proof, production-readiness claim, or acceptance artifact. It defines the current licensing status and permitted review boundaries.

## 1. Purpose

The purpose of this document is to make XPScerpto’s licensing status explicit.

A public repository without a selected open-source license should not be interpreted as free permission to reuse the work.

XPScerpto is visible for review and project evaluation, but visibility is not the same as licensing permission.

This file exists to prevent confusion about:

- whether XPScerpto is currently open-source licensed;
- whether third parties may copy or reuse the code;
- whether commercial use is allowed;
- whether redistribution is allowed;
- whether forks may be used as independent products;
- whether the project’s public visibility creates implied permission.

The current answer is conservative:

```text
No open-source license has been selected.
No reuse permission is granted by default.
All rights are reserved unless written permission or a later license says otherwise.
```

## 2. Current license status

Current status:

```text
LICENSE_SELECTED = NO
OPEN_SOURCE_LICENSED = NO
COMMERCIAL_USE_GRANTED = NO
REDISTRIBUTION_GRANTED = NO
DERIVATIVE_WORKS_GRANTED = NO
ALL_RIGHTS_RESERVED = YES
```

This status may change in the future if the project owner publishes a formal license.

Until then, XPScerpto must be treated as source-available for review only, not open-source licensed for reuse.

## 3. What public visibility allows

Public visibility may allow readers to:

- view the repository;
- review the architecture;
- inspect documentation;
- evaluate project direction;
- understand governance and validation discipline;
- discuss the project at a high level;
- report issues through the appropriate channels;
- propose contributions under the project’s contribution process.

Public visibility does not grant permission to reuse the work.

The ability to view a repository does not mean the ability to copy, redistribute, modify, sublicense, package, sell, or deploy it.

## 4. What is not permitted by default

Unless a formal license is later published or written permission is granted by the project owner, the following are not permitted:

- copying the code for reuse;
- modifying the code for independent use;
- redistributing the source code;
- redistributing modified versions;
- sublicensing the code;
- selling the code;
- using the code commercially;
- embedding the code into another product;
- hosting the code as a service;
- publishing derivative works;
- repackaging the project;
- using the project name or identity to imply endorsement;
- presenting a fork as an official XPScerpto release;
- removing attribution or ownership notices;
- using project evidence, reports, or documentation to claim independent validation.

This list is not exhaustive. If a use depends on permission, assume permission has not been granted unless it is explicitly provided.

## 5. Review-only use

The repository may be reviewed for understanding and evaluation.

Review-only use means reading or analyzing the repository without reusing the material as an independent product or derivative work.

Examples of review-only activities may include:

- reading documentation;
- reviewing architecture;
- evaluating governance;
- inspecting code structure;
- identifying possible issues;
- preparing responsible feedback;
- reporting bugs or vulnerabilities;
- discussing whether a future collaboration is appropriate.

Review-only use does not include copying the source code into another project, redistributing it, or using it to provide a product or service.

## 6. Contributions

Contributions may be accepted only under the project’s contribution process.

Before contributing, read:

```text
CONTRIBUTING.md
REVIEW_POLICY.md
GOVERNANCE.md
OWNERSHIP_MODEL.md
SECURITY.md
```

By submitting a contribution, a contributor should understand that:

- the project is not currently open-source licensed by default;
- contribution acceptance does not automatically create a public open-source license;
- the project may later adopt a formal license;
- contribution rights may need to be clarified before acceptance;
- security-sensitive contributions must follow `SECURITY.md`;
- contributions must not include third-party material that the contributor is not authorized to submit.

If a contributor cannot grant the project the rights needed to review, modify, and potentially include the contribution, the contribution should not be submitted.

## 7. Third-party material

Do not add third-party code, text, test vectors, generated artifacts, images, documentation, or other materials unless their license and origin are clearly compatible with the project’s current licensing position and future licensing plans.

A contribution that includes third-party material should identify:

- the source of the material;
- the applicable license;
- whether modification is allowed;
- whether redistribution is allowed;
- whether commercial use is allowed;
- whether attribution is required;
- whether patent or trademark restrictions apply;
- why the material is appropriate for inclusion.

Unclear third-party licensing is a blocker.

## 8. Future licensing

XPScerpto may adopt a formal license in the future.

Possible future licensing models may include, but are not limited to:

- a permissive open-source license;
- a copyleft open-source license;
- a dual-license model;
- a commercial license;
- a source-available license;
- a separate license for documentation and code;
- a separate license for examples, tools, or non-core assets.

No future license is implied by this document.

A future license must be published explicitly by the project owner.

## 9. Documentation and non-code materials

Unless a separate license is published, documentation, diagrams, reports, governance files, validation files, test descriptions, and other non-code materials are also all rights reserved.

This includes, but is not limited to:

```text
README.md
START_HERE.md
GOVERNANCE.md
ENGINEERING_PRINCIPLES.md
AUTHORITY_MODEL.md
ARCHITECTURE.md
MODULE_BOUNDARIES.md
THREAT_MODEL.md
SECURITY.md
CI_VALIDATION_GATES.md
REVIEW_POLICY.md
CONTRIBUTING.md
OWNERSHIP_MODEL.md
SUPPORT.md
docs/
reports/
evidence/
```

Public readability does not grant permission to copy, republish, translate, redistribute, or commercially reuse documentation.

## 10. Project name and identity

The XPScerpto name, identity, branding, domain names, repository identity, visual identity, documentation style, governance language, and project presentation must not be used to imply endorsement, partnership, certification, or official status without written permission.

Forks, mirrors, references, reviews, and commentary must not present themselves as official XPScerpto releases unless explicitly authorized.

The project’s public identity is separate from source-code licensing.

## 11. No production-readiness claim

This licensing status does not elevate project status.

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

A licensing status file is not a production-readiness claim, security proof, acceptance artifact, or validation report.

## 12. No warranty

XPScerpto is provided for review and evaluation only in its current licensing state.

No warranty is provided.

No representation is made that the project is fit for production use, commercial deployment, regulatory use, security-critical use, or any other specific purpose.

Any evaluation is at the evaluator’s own risk.

## 13. Requesting permission

To request permission for use beyond review-only evaluation, contact the project owner or use the official project contact route if available.

Official project information may be available at:

```text
https://xpscerpto.eu
```

A permission request should include:

- who is requesting permission;
- the intended use;
- whether the use is commercial;
- whether redistribution is intended;
- whether modification is intended;
- whether the project name will be referenced;
- whether the use involves security-sensitive or cryptographic deployment;
- whether a written agreement is required.

Permission is not granted unless the project owner provides it explicitly in writing or publishes a formal license that covers the requested use.

## 14. Relationship to other documents

This document should be read with:

```text
README.md
START_HERE.md
GOVERNANCE.md
CONTRIBUTING.md
SECURITY.md
OWNERSHIP_MODEL.md
SUPPORT.md
```

`README.md` introduces the project.

`LICENSE_STATUS.md` defines the current licensing status.

`CONTRIBUTING.md` defines how contributions should be prepared.

`SECURITY.md` defines how vulnerabilities should be reported.

`OWNERSHIP_MODEL.md` defines repository ownership policy.

`SUPPORT.md` defines general support channels.

## 15. README 

XPScerpto is currently published for review, evaluation, and project visibility only. No permission is granted to use, copy, modify, redistribute, sublicense, or commercially deploy the code unless a formal license is later published or written permission is granted by the project owner.
```

## 16. Final licensing rule

The final licensing rule is:

```text
public visibility is not reuse permission
review access is not redistribution permission
no license means no open-source grant
written permission or a published license is required for use beyond review
```

XPScerpto may adopt a formal license later. Until then, all rights are reserved unless explicitly stated otherwise.
