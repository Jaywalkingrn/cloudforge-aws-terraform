# CloudForge — Verification Evidence

**Status:** M0 evidence only. No implementation or deployment results exist.  
**Record date:** November 25, 2026, America/Toronto.

### Evidence attribution

Owner statements are labeled owner-provided. Agent observations identify commands and actual results. Version checks establish executable availability, not artifact integrity or comprehensive security suitability.

### M0 — Owner-provided evidence

The owner reported installation and integrity verification completed on **November 25, 2026**:

| Component | Owner-reported result |
|---|---|
| Python 3.14.10 | Matching SHA-256; valid Python Software Foundation Authenticode signature |
| Terraform 1.17.2 | Matching SHA-256; valid HashiCorp Authenticode signature |
| PowerShell 7.7.0 | WinGet installer checksum verification |
| Cosign 3.1.5 | WinGet installer checksum verification |
| Trivy 0.76.2 | Matching SHA-256; successful Cosign verification using official release workflow identity |
| Git | Upgraded to 2.56.1.windows.1; existing author identity preserved |
| Python 3.11.9 | Preserved |

The agent did not repeat these integrity checks or independently verify author-identity preservation. No Git-upgrade integrity method beyond the owner’s supplied statement is asserted.

### M0 — Independently observed runtime results

Working directory:

```text
%USERPROFILE%\Projects\cloudforge-aws-terraform
```

#### Current Git audit

Commands:

```powershell
Get-Command git,pwsh -All
git --version
pwsh --version
git status --short
```

| Check | Result | Native exit |
|---|---|---|
| Git location | `C:\Program Files\Git\cmd\git.exe` | Not applicable |
| Git version | `2.56.1.windows.1` | 0 |
| PowerShell version | Bundled `7.6.9` | 0 |
| Working tree | `git status --short`: no output | 0 |

Git 2.45.1.windows.1 was observed in an earlier audit. It is historical evidence, superseded by the current runtime result.

A standard `C:\Program Files\PowerShell\7\pwsh.exe` executable was not found. Earlier invocation of the user’s WindowsApps PowerShell alias failed. The owner’s 7.7.0 installation remains owner-verified; it has not been independently executed from the agent.

#### Verified tool selection and process-local PATH test

| Tool | Independently verified executable |
|---|---|
| Python | `%USERPROFILE%\AppData\Local\Programs\Python\Python314\python.exe` |
| Terraform | `%USERPROFILE%\Tools\Terraform\1.17.2\terraform.exe` |
| Trivy | `%USERPROFILE%\Tools\Trivy\0.76.2\trivy.exe` |
| Cosign | `%USERPROFILE%\AppData\Local\Microsoft\WinGet\Packages\Sigstore.Cosign_Microsoft.Winget.Source_8wekyb3d8bbwe\cosign-windows-amd64.exe` |

A prior audit on November 25 temporarily set process PATH to only these verified tool directories, resolved each application, compared its absolute location, and ran:

| Command | Observed output | Native exit |
|---|---|---|
| `python --version` | Python 3.14.10 | 0 |
| `terraform version` | Terraform 1.17.2, windows_amd64 | 0 |
| `trivy --version` | Version 0.76.2 | 0 |
| `cosign-windows-amd64 version` | v3.1.5, windows/amd64 | 0 |

All paths matched. Original process PATH restoration was confirmed.

The initial harness had a PowerShell parameter error; it was corrected before the complete successful rerun.

Default process PATH differs from stored user PATH. Absolute-path selection is the accepted strategy; no global PATH repair is required.

User-specific path prefixes have been redacted to %USERPROFILE%. These paths are audit evidence, not tracked verification-script defaults.

### Security review status

See `decisions.md`:

- Git November 10 advisories: disposition recorded against observed 2.56.1.
- PowerShell CVE-2026-122985: bundled 7.6.9 remains affected; prefer owner 7.7.0.
- Python, Terraform, Trivy, Cosign, provider, and CI dependency reviews remain pending where no documented review exists.
- Provider checksum and action-SHA verification must precede dependency execution during M1.

No blanket “security review passed” claim is made.

### Results not yet available

No application tests, Terraform validation/mock tests, package reproducibility checks, infrastructure scans, CI runs, AWS calls, deployment, or cleanup verification have occurred.

Future entries must record actual date, environment, tool versions, command, native exit, test count where applicable, outcome, sanitized evidence, and limitations. Failed checks remain recorded with their resolution.

M1–M4 remain unapproved. This record contains M0 evidence only.
