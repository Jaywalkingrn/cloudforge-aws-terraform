# AGENTS.md — CloudForge

This file is intentionally short and maintained by the project owner. Read `ARCHITECTURE.md` and the relevant `doc/` specifications before acting.

## Workflow
- Use Plan Mode first. Do not implement until the owner approves a specific milestone in `doc/implementation-plan.md`.
- Work only within the approved scope; propose material design or scope changes for review.
- Make small, reviewable changes on a feature branch. Do not merge without owner approval.
- Run relevant tests, inspect their actual output, fix failures, and record evidence before claiming success.
- Update detailed specifications in `doc/` after significant changes; keep the root architecture overview stable.

## Boundaries
- No AWS provisioning, deployment, teardown, paid resource creation, or cloud credential changes without explicit approval.
- Never expose secrets or commit Terraform state, credentials, or generated secret files.
- Never execute `rm`, destructive Git history operations, force-push, or unapproved deletion. Git pushes require owner approval.
- Follow Codex sandbox and command rules; these written instructions are **not** a substitute for enforced permissions.
- Do not fabricate tests, timestamps, authorship, commits, or activity.

## Read next
- `ARCHITECTURE.md` — stable design and system boundaries
- `doc/spec.md` — requirements and acceptance criteria
- `doc/implementation-plan.md` — owner-approved execution scope (not yet created)
- `doc/testing.md` — verifiable test evidence (not yet created)
- `doc/operations.md` — deployment and recovery procedures (not yet created)
