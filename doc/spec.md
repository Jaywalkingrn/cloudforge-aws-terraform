# CloudForge — Initial Specification

**Status:** DRAFT — not approved for implementation
**Owner:** Jay (`jaywalkingrn`)
**Project type:** Personal Cloud / DevOps portfolio project
**Primary goals:** Infrastructure as Code, least-privilege cloud security, CI validation, reproducibility, safe operations, and demonstrable troubleshooting.

## 1. Problem statement
Show that a developer can define, validate, secure, and operate a small AWS service using repeatable infrastructure and deployment workflows—not just create an API endpoint manually in the cloud console.

## 2. Initial demonstrator
Provide a minimal HTTP API behind AWS API Gateway (HTTP API v2) backed by a Python Lambda function. The specific API behavior is deliberately simple:

| Request | Expected behavior |
| --- | --- |
| `GET /health` | HTTP 200 and JSON containing `status: "ok"` |
| Unknown route | A deliberate non-success response (404 where routing allows) |

The service must work in local/unit tests without deploying to AWS. The public endpoint is required only for the separately approved deployment verification milestone.

## 3. Functional requirements
- **CF-01:** Terraform defines all AWS resources used by the demonstrator; no manual console-only configuration is necessary after approved bootstrap steps.
- **CF-02:** Terraform configuration is parameterized for region, project naming, and environment while using restrained defaults.
- **CF-03:** API Gateway invokes Lambda through a documented HTTP API Lambda proxy integration (payload format 2.0).
- **CF-04:** The Python handler returns a documented JSON response and has unit tests covering health and unknown-route behavior.
- **CF-05:** Lambda uses a dedicated least-privilege execution role; the deployment identity is separate.
- **CF-06:** CloudWatch logging is configured with bounded retention and avoids secrets in application logs.
- **CF-07:** GitHub Actions verifies formatting, Terraform configuration, Python tests, and infrastructure security checks without creating AWS resources.
- **CF-08:** A separately approved manual deployment workflow may assume a scoped AWS IAM role using GitHub OIDC; no long-lived AWS credentials in repository secrets.
- **CF-09:** Document an owner-controlled deployment, smoke test, and teardown procedure; do not execute them without permission.

## 4. Non-functional requirements
- Reproducible setup on Windows with a documented shell environment; no undocumented local configuration.
- Repository secrets, Terraform state, `.terraform/` directories, credentials, and generated artifacts are excluded from Git as appropriate.
- Provider and dependency versions are pinned where practical.
- Simple structure suitable for a student-owned portfolio; avoid speculative abstractions, databases, networking stacks, or excessive services.
- All statements about tests, AWS operations, logs, or costs must be supported by actual evidence.

## 5. Testing and evidence
**Pre-deployment (approved for CI once implementation is approved):**
- `terraform fmt -check -recursive`
- `terraform init -backend=false` followed by `terraform validate` where applicable
- Python unit tests with recorded exit status and test counts
- Static IaC/security scanning, with results and justified exceptions documented
- Terraform tests, if added, must be demonstrably non-provisioning (mocked or plan-only); do not run tests that apply real cloud resources without approval

**Post-deployment (future, owner-approved):**
- Record the approved AWS account alias/region (not account secrets), infrastructure plan summary, and smoke-test response
- Verify a real API response and CloudWatch log event without exposing sensitive information
- Verify resource cleanup after an explicitly approved teardown and note any residual costs or state artifacts

Evidence belongs in `doc/testing.md` with commands, date, outcome, and sanitized excerpts or links to CI runs. Do not claim any item has passed before it actually has.

## 6. Milestones requiring approval
1. **M0 — Planning:** finalize architecture, this specification, and Codex command restrictions. Codex produces `doc/implementation-plan.md` in Plan Mode. **No coding.**
2. **M1 — Offline foundation:** repo layout, Python handler, unit tests, Terraform skeleton, static validation, CI checks. **No AWS credentials or deployment.**
3. **M2 — Infrastructure integration:** Terraform resource wiring and least-privilege IAM, plan/mock tests, security scanning, documentation. **No AWS deployment.**
4. **M3 — Controlled AWS verification:** owner-approved state strategy, scoped OIDC role, reviewed deployment plan, manual deployment and smoke tests, documented teardown. **Requires separate explicit approval and cost review.**
5. **M4 — Resume readiness:** refine README, architecture explanation, screenshots/CI evidence, runbook, limitations, and interview walkthrough. **Only claim verified capabilities.**

Approval of this specification is not approval to execute any milestone. Each implementation milestone must be marked approved in `doc/implementation-plan.md` before Codex begins writing implementation code.

## 7. Explicitly out of scope (initial version)
- Persistent databases, user authentication, multi-account infrastructure, large-scale traffic generation, high-availability claims, and production SLAs.
- Automatic deployments on every push, continuously running paid infrastructure, and autonomous billable resource provisioning.
- Artificial commits, manipulated commit timestamps, unsupported resume claims, or hiding required attribution.

## 8. Documentation deliverables
- Root: `README.md`, `AGENTS.md`, `ARCHITECTURE.md`
- Detail: `doc/spec.md`, `doc/implementation-plan.md`, `doc/testing.md`, `doc/decisions.md`, `doc/operations.md`
- Diagrams, reproducible commands, CI output, security rationale, and teardown guidance as the project matures.

## 9. Open decisions for owner review
- Confirm `ca-central-1` as default AWS region or pick another supported region.
- Choose the initial Terraform state/backend approach before any AWS deployment.
- Approve the precise CI security scanner, runtime version, and repository branch-protection rules in Codex Plan Mode.
- Decide whether AWS deployment is necessary to call the project resume-ready; until then label it a validated IaC prototype, not a deployed service.
