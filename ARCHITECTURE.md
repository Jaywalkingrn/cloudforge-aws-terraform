# CloudForge — Architecture

**Status:** Draft for owner review. No implementation or cloud deployment is authorized.

## Purpose
CloudForge demonstrates secure, reproducible AWS infrastructure through Terraform, minimal application code, automated validation, and documented operational controls. Its engineering focus is **infrastructure and DevOps**, not an elaborate web application.

## Proposed system

```text
Developer / GitHub pull request
            |
            v
GitHub Actions (no cloud credentials)
  - Terraform format + validate
  - Python unit tests
  - IaC/security checks
            |
            v
Approved release workflow (future milestone; manual approval)
            |
            v
GitHub OIDC -> scoped AWS IAM deployment role
            |
            v
Terraform-managed AWS resources
  API Gateway HTTP API (v2, Lambda proxy)
             |
             v
       AWS Lambda (Python)
             |
             v
       CloudWatch Logs
```

## Components and decisions
- **Terraform:** source of truth for AWS resource configuration. Begin with a small root module and split into reusable modules only when the design justifies it. Pin provider versions and commit dependency lock files.
- **AWS API Gateway HTTP API:** public HTTPS endpoint routed to Lambda using payload format 2.0. A tiny demonstration API keeps the project's focus on infrastructure.
- **AWS Lambda:** supported, pinned Python runtime; no database, VPC, or continuously running compute in the initial scope.
- **IAM:** Lambda runtime and CI deployment roles have distinct minimum permissions. Future GitHub-to-AWS deployments use OIDC with a repository- and environment-scoped trust policy, not stored AWS access keys.
- **Observability:** structured application logging to CloudWatch; bounded log retention. Alarms and additional monitoring are considered only after costs are reviewed.
- **Terraform state:** never committed. The initial backend and collaborative state strategy require separate approval before any AWS deployment.

## Trust and cost boundaries
- Pull-request checks should run without AWS credentials or infrastructure creation.
- No `terraform apply`, `terraform destroy`, or billable cloud provisioning without the owner's explicit approval. A green CI run is **not** proof of successful AWS deployment.
- The AWS region is configurable; the proposed initial target is `ca-central-1`, subject to resource and pricing verification.
- Protect logs from secrets and personally identifying data. Keep all deployment roles and workflow permissions narrow.

## Documentation and change control
`doc/spec.md` defines acceptance criteria. Codex proposes an implementation plan in Plan Mode; the owner approves it before coding. Detailed decisions, verification evidence, and operational steps belong in `doc/`, and `ARCHITECTURE.md` changes only for material system design changes.
