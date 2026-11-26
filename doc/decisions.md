# CloudForge — Decisions and Security Dispositions

**Status:** Owner-selected planning decisions; no implementation authorization.  
**Recorded:** November 25, 2026.

### D01 — Minimal architecture

Use one small Terraform service root, API Gateway HTTP API, Python Lambda, dedicated IAM execution role, and bounded CloudWatch logs. Add modules only when actual reuse warrants them. Exclude application frameworks, databases, VPCs, and speculative services.

### D02 — Governance and documentation

Milestone approvals are independent. Keep `AGENTS.md` owner-maintained and the architecture overview stable. Detailed requirements, decisions, implementation progress, operations, and verifiable evidence belong under `doc/`.

All sandbox, publishing, credential, deletion, installation, and AWS approval restrictions remain in force.

### D03 — Git security disposition

Select Git **2.56.1.windows.1**. The active agent runtime independently reports that version from `C:\Program Files\Git\cmd\git.exe`.

Git 2.56.1 includes fixes for the November 10, 2026 advisories. This disposition is supported by the [Git security announcement](https://github.blog/open-source/git/git-security-vulnerabilities-announced-7/), updated November 17.

This closes the identified version gap for those advisories; it does not certify all bundled Git copies or future advisories. Do not replace bundled executables.

Owner-reported upgrade date: November 25, 2026. Preservation of author identity is owner-provided evidence.

### D04 — PowerShell compatibility and security

Maintain script compatibility with Codex PowerShell 7.6.9 and owner PowerShell 7.7.0. Prefer patched 7.7.0 where accessible.

PowerShell 7.6.9 is affected by CVE-2026-122985; Microsoft lists patched releases 7.6.10 and 7.7.0 in its [official advisory](https://github.com/PowerShell/Announcements/blob/main/pwsh/2026/11/CVE-2026-122985.md).

Compatibility acceptance does not mean 7.6.9 is patched. Record which shell actually executes checks. Leave bundled Codex executables unchanged; do not alter permissions, profiles, or global configuration.

### D05 — Tool selection and portability

Retain Python 3.14.10, Terraform 1.17.2, Trivy 0.76.2, and Cosign 3.1.5, subject to documented dependency/advisory review.

Use verified absolute paths supplied through script parameters. Do not hardcode personal paths into implementation defaults. Temporary process environment changes must be restored.

Owner-reported integrity checks and independently observed version checks are separate evidence classes.

### D06 — Dependency verification gates

During M1, before dependency execution:

- Review official release notes/security advisories for retained tools and providers.
- Verify provider signed checksums and commit Windows/Linux lock entries.
- Verify every action’s exact upstream release SHA, advisory status, and runner compatibility.
- Record evidence and disposition; stop for review if a selected dependency is unsuitable.

Proposed providers: AWS 6.74.0 and Archive 2.8.1.

Proposed action releases: checkout 7.0.1, setup-python 7.1.0, setup-terraform 4.0.1. Their release SHAs have not yet been verified for implementation.

### D07 — Offline testing and state separation

M1/M2 require no AWS credentials, API access, or infrastructure creation. Initial dependency downloads are distinct from offline verification.

Use disposable staged source, fresh metadata, isolated child environments, mocked AWS providers, and a verified provider mirror. Validate full configuration separately from backend-free mocked execution. Repeat after S3 introduction.

Do not edit production backend configuration to run tests.

### D08 — ZIP claims

Local repeat-build reproducibility and cross-platform byte identity are separate claims. Test and report them separately. Do not infer either from functional tests alone.

### D09 — Region and M3 state

Select `ca-central-1`, subject to M3 availability/cost review. Use private encrypted versioned S3 state with native locking and one application state key.

Bootstrap state stays protected outside Git. Existing shared resources remain externally owned unless adoption is separately approved.

### D10 — OIDC and existing resource protection

Use a dedicated planning role with explicit cloud reads, exact state-read access, and exact lock-object operations. Deny state writes; grant no infrastructure mutation capability.

Reference an existing GitHub OIDC provider without taking ownership or altering it. Stop on resource collisions.

Use owner-run apply/teardown. GitHub environment `cloudforge-plan` has Jay as sole required reviewer, with self-review prevention disabled. Manual workflow approval does not authorize deployment.

### D11 — Artifact and evidence handling

Keep state, plans, credentials, generated deployment packages, and unsanitized logs out of Git and public CI artifacts. Commit dependency locks and reviewed sanitized evidence only.

A green CI run is not deployment evidence.

### D12 — Development cadence and portfolio gate

Develop primarily Saturday and Sunday, with sessions sized to actual work. Do not manufacture commits, activity, elapsed duration, or results. Do not leave resources running solely to fit the schedule.

M4 requires real M3 deployment and cleanup evidence. Claims must distinguish a historical verified deployment from a currently running service.
