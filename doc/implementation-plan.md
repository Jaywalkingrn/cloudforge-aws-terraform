# CloudForge — Implementation Plan

**Status:** Roadmap accepted in principle; implementation unapproved.  
**Owner:** Jay (`jaywalkingrn`)  
**Planning record updated:** November 25, 2026

### Approval and operating boundaries

| Milestone | Status |
|---|---|
| M0 — Planning and toolchain prerequisites | Owner installation/integrity verification reported complete; runtime evidence recorded; remaining security review items documented |
| M1 — Offline foundation | Unapproved |
| M2 — Infrastructure integration | Unapproved |
| M3 — Controlled AWS verification | Unapproved; separate cost and operational approvals required |
| M4 — Portfolio readiness | Unapproved; verified M3 required for completion |

Architecture or roadmap acceptance does not authorize implementation. Each milestone requires explicit owner approval recorded here before work begins. Approval of one milestone does not authorize another.

Preserve all sandbox, filesystem, command, Git publishing, credential, and AWS restrictions. No `rm`, destructive Git history operations, force pushes, unapproved deletion, installations, or permission changes. Pushes and merges require owner approval. Cloud provisioning, deployment, teardown, paid resources, and credential changes require explicit approval.

Keep `AGENTS.md` owner-maintained. Keep `ARCHITECTURE.md` concise and stable; change it only for significant architectural changes. Detailed requirements, decisions, progress, runbooks, and verified evidence belong under `doc/`.

### Technology baseline

| Component | Selected version/disposition |
|---|---|
| Git | 2.56.1.windows.1 |
| Owner PowerShell | 7.7.0; preferred patched execution environment |
| Codex PowerShell | Bundled 7.6.9; compatibility target with documented security exposure |
| Python | 3.14.10, standard GIL build |
| Preserved Python | 3.11.9; not the CloudForge runtime |
| Lambda | `python3.14`; managed patch level controlled by AWS |
| Terraform | 1.17.2 |
| AWS provider | 6.74.0 |
| Archive provider | 2.8.1 |
| Trivy | 0.76.2 |
| Cosign | 3.1.5 |
| Application/testing libraries | Python standard library |
| CI runners | `ubuntu-24.04`, `windows-2025` |
| Proposed actions | checkout 7.0.1, setup-python 7.1.0, setup-terraform 4.0.1; verify full release SHAs before use |

Retain these selections subject to availability, documented advisory/dependency review, and integrity verification. Propose substitutions for review rather than silently upgrading or downgrading.

### M0 — Planning and Windows prerequisites

**Objective:** establish a documented, usable toolchain without implementing the project.

**Files:** `doc/implementation-plan.md`, `doc/testing.md`, and `doc/decisions.md`. Recording this roadmap is documentation-only and does not authorize implementation.

**Completed owner work:** November 25 installation and integrity checks, including the Git upgrade. See `testing.md` for the distinction between owner-provided and independently observed evidence.

**Execution strategy:**

- Use verified absolute executable paths through local invocation overrides.
- Support PowerShell 7.6.9 and 7.7.0 syntactically; prefer patched 7.7.0 where accessible.
- Do not modify bundled Codex executables, global PATH/configuration, profiles, execution policy, or sandbox permissions.
- Temporary PATH changes must affect only the process and be restored in `finally`.
- Keep personal executable paths out of tracked script defaults.

**Owner setup/recovery checklist:**

1. Discover executables before concluding a tool is absent:
   ```powershell
   Get-Command git,pwsh,python,terraform,trivy,cosign-windows-amd64 -All
   py --list
   ```
2. Check known installation paths and invoke discovered executables directly. Distinguish app aliases, PATH failures, execution failures, and missing installations.
3. Obtain artifacts only from official release sources:
   - [Python 3.14.10](https://www.python.org/downloads/release/python-31410/)
   - [Terraform 1.17.2](https://releases.hashicorp.com/terraform/1.17.2/)
   - [Trivy 0.76.2](https://github.com/aquasecurity/trivy/releases/tag/v0.76.2)
   - [Cosign 3.1.5](https://github.com/sigstore/cosign/releases/tag/v3.1.5)
4. Compare SHA-256 using `Get-FileHash -Algorithm SHA256`. Verify Python/Terraform publisher signatures with `Get-AuthenticodeSignature`; verify Trivy’s Sigstore bundle using its exact official workflow identity.
5. Record source, version, verification method, date, and outcome. Do not reinstall already working tools merely because bare command discovery fails.

**Verification:** run version commands and capture native exit codes immediately. Confirm resolved executable locations. Record results in `testing.md`.

**Acceptance:** installed versions are usable through the selected paths; integrity evidence is attributed correctly; known security dispositions and remaining reviews are explicit.

**Remaining work:** explicit M1 approval and effective write-permission review; dependency/advisory checks remain gates before dependency use. Default PATH alignment is not a prerequisite because absolute paths work.

### M1 — Offline foundation

**Objective and scope:** create a minimal tested handler, Terraform skeleton, portable verification, and credential-free CI. Define no AWS resources.

**Create:**

```text
app/handler.py
tests/test_handler.py
tests/fixtures/insecure-s3/main.tf
infra/{versions,providers,variables,main,outputs}.tf
infra/.terraform.lock.hcl
scripts/{verify-toolchain,verify,verify-offline}.ps1
.github/workflows/ci.yml
.python-version
.terraform-version
.gitattributes
doc/operations.md
```

Modify README, `.gitignore`, and detailed documentation, including approved specification clarifications.

**Steps and rationale:**

1. Review retained tool/provider releases and advisories. Verify provider origins, signed checksums, and lock entries before loading providers. Verify each action’s full upstream release SHA and security/runner compatibility before executing it.
2. Implement `handler(event, context)` for HTTP API payload 2.0:
   - `GET /health`: 200, JSON content type, `{"status":"ok"}`.
   - Other method/path combinations: deliberate 404.
3. Test response contracts, unknown routes, wrong methods, incomplete event fields, query-string independence, structured logs, and sensitive-input exclusion.
4. Log only request ID, normalized outcome, and status; exclude bodies, headers, queries, and arbitrary paths.
5. Define exact Terraform/provider requirements and validated project/environment/region inputs. Defaults: `cloudforge`, `dev`, `ca-central-1`.
6. Generate locks for Windows/Linux amd64. Use LF source files and portable relative paths.
7. Implement executable overrides:
   `-PythonExecutable`, `-TerraformExecutable`, `-TrivyExecutable`, `-CosignExecutable`, and `-GitExecutable`.
   Resolve once, invoke absolute paths, reject invalid explicit overrides, and preserve native exit codes.
8. Use no framework, WSL, Docker, Make, or additional application dependencies.

**CI and security:**

- Python/Terraform checks on both runners; Linux security job.
- PR, `main` push, and manual triggers.
- `contents: read`; no AWS credentials, repository secrets, or OIDC permission.
- Pin verified action SHAs; disable persisted checkout credentials and Terraform’s wrapper.
- Report all Trivy severities; block unresolved HIGH/CRITICAL findings. Document resource-specific exceptions.
- Require detection of the deliberately insecure fixture.
- Ignore state/backups, `.terraform/`, plans and plan JSON, credentials, generated packages, raw logs, local backend configuration, and `.work/`. Track dependency locks.
- Publish only reviewed summaries or sanitized evidence; do not upload raw Terraform working directories or deployment artifacts.

**Exact verification commands after approved implementation:**

```powershell
terraform -chdir=infra providers lock -platform=windows_amd64 -platform=linux_amd64
python -m unittest discover -s tests -p "test_*.py" -v
terraform fmt -check -recursive
pwsh -NoProfile -File scripts/verify-offline.ps1
trivy config --skip-check-update --exit-code 1 --severity HIGH,CRITICAL infra
trivy config --skip-check-update --exit-code 1 --severity HIGH,CRITICAL tests/fixtures/insecure-s3
git diff --check
```

The lock command is initial dependency setup, not a routine lock update. Use the reviewed executable overrides when bare commands are unsuitable.

**Acceptance:** real test counts recorded; normal checks exit 0; the insecure fixture exits 1 with the expected finding. Both CI platforms pass after an authorized push. An empty infrastructure scan is identified as such.

**Dependencies/blockers:** M0 baseline; independent M1 approval; approved write capability. Dependency review, checksum verification, and action-SHA verification are initial M1 gates before dependency use.

### M2 — Infrastructure integration without AWS access

**Objective:** define and test the service graph without credentials, cloud calls, or provisioning.

**Files:** modify `infra/*.tf`, verification/CI, and detailed specifications. Add `infra/tests/service.tftest.hcl` and `scripts/verify-package.ps1`.

**Implementation and security:**

- Define Lambda, its dedicated role and scoped inline logging policy, explicit seven-day log group, HTTP API, payload 2.0 integration, `GET /health`, `$default` stage, and route-scoped invocation permission.
- Use Python 3.14, `handler.handler`, x86_64, 128 MB, three-second timeout, and low API throttling.
- Allow log-stream creation and event writes only within the pre-created group.
- Package only `handler.py`, fix archive mode to `0644`, and wire the archive hash to Lambda.
- Export API URL, function name, and log-group name.
- Add no function URL, wildcard route, database, VPC, CORS, layers, or broad managed IAM policy.
- Mock all AWS provider instances. Every Terraform test run explicitly uses `command = plan`; Archive performs only local packaging.
- Assert configuration wiring, IAM, retention, runtime, permissions, hashes, and input validation.

**Offline isolation, retained after M3:**

- Stage allowlisted source into fresh ignored `.work/offline/<run-id>/` directories. Never copy live state, metadata, credentials, packages, plans, or local backend files.
- Validate the complete configuration, including future `backend.tf`, using fresh metadata and `init -backend=false`.
- Run mocked tests in another source-identical copy excluding only dedicated `backend.tf`, which must contain backend configuration only.
- Compare source manifests to prevent accidental omission of unrelated configuration.
- Clear credential/profile overrides only in child processes; use empty AWS config/credential files and disable metadata lookup.
- Use a verified local provider mirror with no direct-download fallback for offline proof.

**Commands:**

```powershell
pwsh -NoProfile -File scripts/verify-offline.ps1 -PrepareOnly -ProviderMirror .work/provider-mirror
pwsh -NoProfile -File scripts/verify-offline.ps1 -ProviderMirror .work/provider-mirror
pwsh -NoProfile -File scripts/verify-package.ps1
pwsh -NoProfile -File scripts/verify.ps1
```

Within staged directories, the helper runs:

```text
terraform init -backend=false -input=false -lockfile=readonly
terraform validate
terraform init -input=false -lockfile=readonly
terraform test -no-color
```

The first pair targets complete configuration; the second targets the backend-free test copy.

Where supported without permission escalation, prove Linux network independence after dependency preparation:

```bash
unshare --user --map-root-user --net pwsh -NoProfile -File scripts/verify-offline.ps1 -ProviderMirror .work/provider-mirror
```

If unavailable, record the network-isolation proof as pending; credential-free success alone is not proof of blocked networking.

**Packaging acceptance:**

- Compare two builds on the same platform for repeat-build reproducibility.
- Separately compare Windows/Linux hashes for identical commits and tools.
- Transfer only hash summaries between jobs.
- Report exactly what each test establishes; functional correctness does not imply cross-platform byte identity.

**Acceptance:** actual Python/Terraform counts, package contract checks, safe scanner dispositions, and credential-free success on both platforms.

**Dependencies/blockers:** accepted M1, independent M2 approval; any actual mock, provider, packaging, or isolation failures discovered during implementation.

### M3 — Controlled AWS verification

**Objective:** verify real deployment, observability, OIDC planning, convergence, and cleanup.

**Files:** add `bootstrap/` and its lock, `infra/backend.tf`, backend example, `.github/workflows/aws-plan.yml`, smoke/cleanup scripts, and `doc/cost-review.md`. Update offline tests and operations/evidence.

**Dependencies and approval gates:**

- Accepted M2 and independent M3 approval.
- Dated `ca-central-1` price/availability/quota review and owner budget.
- Separate approvals for bootstrap, apply, and teardown.
- AWS CLI v2 patch/provenance selected and recorded at M3.

**Resource ownership and backend:**

- Inspect existing account-level resources before writes.
- Reference an existing GitHub OIDC provider by approved ARN; do not import, modify, or destroy it.
- Stop on bucket/role naming collisions. Resource adoption or shared-resource changes need explicit review.
- Create a dedicated private, versioned, SSE-S3, TLS-only state bucket with public-access blocking.
- Use `cloudforge/dev/terraform.tfstate`, native S3 locking, no workspaces, and no DynamoDB table.
- Keep bootstrap state protected outside Git; retain bootstrap/state storage by default.
- Re-run the isolated offline suite after backend introduction.

**OIDC planning role:**

| Capability | Scope |
|---|---|
| `s3:ListBucket` | Dedicated bucket; restricted state/lock prefixes |
| `s3:GetObject` | Exact application state object |
| `s3:GetObject`, `s3:PutObject`, `s3:DeleteObject` | Exact corresponding `.tflock` object |
| AWS read APIs | Explicit plan-time metadata reads; resource scope wherever supported |
| State modification | Explicitly deny state-object writes/deletes |
| Infrastructure modification | No create/update/delete, `iam:PassRole`, role chaining, or Lambda invocation |

State reads expose state contents; lock writes/deletes can disrupt concurrent operations. Serialize operations, retain locking, and document unavoidable read APIs requiring broader resource scope. Do not broaden permissions or disable locks to conceal failures.

**Workflow:** manual dispatch on protected `main`, environment `cloudforge-plan`, Jay as sole required reviewer, self-review prevention disabled. Use `contents: read` and job-scoped `id-token: write`; trust exact repository/environment subject and STS audience. No apply/destroy job. Environment approval authorizes that planning run only.

If required reviewers are unavailable for the repository plan/visibility, resolve that feature limitation before enabling the workflow.

**Owner-run commands, only after relevant approvals:**

```powershell
aws sts get-caller-identity --profile $OwnerProfile
aws iam list-open-id-connect-providers --profile $OwnerProfile
terraform -chdir=bootstrap init -input=false
terraform -chdir=bootstrap plan -out=bootstrap.tfplan
terraform -chdir=bootstrap apply bootstrap.tfplan
terraform -chdir=infra init -reconfigure -backend-config=backend.local.hcl
terraform -chdir=infra plan -out=deploy.tfplan -detailed-exitcode
terraform -chdir=infra apply deploy.tfplan
$ApiUrl = terraform -chdir=infra output -raw api_url
pwsh -NoProfile -File scripts/smoke-test.ps1 -BaseUrl $ApiUrl
terraform -chdir=infra plan -detailed-exitcode
```

Use the approved temporary owner identity consistently. Review saved plans privately before applying; retain the original ZIP. Capture correlated logs privately and publish sanitized evidence only.

After separate teardown approval:

```powershell
terraform -chdir=infra plan -destroy -out=destroy.tfplan
terraform -chdir=infra apply destroy.tfplan
terraform -chdir=infra state list
pwsh -NoProfile -File scripts/verify-cleanup.ps1 -InventoryPath .work/deployed-resources.json -Profile $OwnerProfile -Region ca-central-1
```

**Acceptance:** initial reviewed plan exit 2; successful apply; health 200; unmatched route 404; correlated structured log; convergence exit 0; empty application state and service-read confirmation of cleanup. Verify pre-existing resources unchanged and disclose retained costs.

**Potential blockers:** actual account, cost, quota, IAM, environment-feature, state, or cleanup issues discovered during M3.

### M4 — Portfolio readiness

**Objective:** present verified engineering capability after M3.

**Files:** refine README and detailed docs; add `doc/walkthrough.md` and sanitized `doc/assets/`.

**Steps:** document architecture/trust boundaries, offline tests, state/locking, OIDC, troubleshooting, recovery, costs, and cleanup. Link real CI evidence and trace resume claims to verified results. Distinguish historical deployment from a currently running service.

**Verification:**

```powershell
pwsh -NoProfile -File scripts/verify.ps1
git diff --check
```

Manually verify links, sanitization, and claim-to-evidence mapping.

**Acceptance:** checks pass, M3 evidence exists, claims are accurate, and sensitive artifacts remain excluded.

**Dependencies/blockers:** independent M4 approval and accepted M3 evidence; actual evidence gaps block completion.

### Development sessions

Recurring development days are Saturday and Sunday. Sessions are meaningful work packages, not mandatory weeks:

1. M0 planning, setup evidence, and dispositions.
2. M1 handler/tests and repository hygiene.
3. M1 Terraform skeleton, isolation, and CI.
4. M2 wiring/IAM.
5. M2 mocks, packaging, and security review.
6. M3 cost/access review and approved bootstrap.
7. M3 approved deployment, verification, and cleanup.
8. M4 documentation and portfolio evidence.

Combine or split sessions according to actual progress. Do not extend infrastructure lifetime to fit a weekend schedule. Manufacture no commits, timestamps, activity, or test evidence.
