# OSV org scan

Google [OSV-Scanner](https://google.github.io/osv-scanner/) is the org-wide Snyk replacement for **lockfile and manifest CVEs**.

Trivy stays on images that actually ship (TaskFlow already scans Docker that way). Running Trivy across every tutorial repo would recreate Snyk’s OS-package backlog.

## What this repo does

Every Monday (and on `workflow_dispatch`) GitHub Actions:

1. Lists public, non-fork, non-archived `raimonvibe` repos
2. Shallow-clones each one
3. Runs `osv-scanner` recursively
4. Uploads the full `osv-org-report.md` as an artifact and opens or updates the **OSV org-wide report** issue with a compact summary (GitHub issue bodies cap at 65,536 characters)

Private repos are skipped unless you add a `OSV_SCAN_TOKEN` secret with `repo` scope and extend the workflow.

## Per-repo PR gating

Copy `templates/osv-scanner.yml` into a project as `.github/workflows/osv-scanner.yml`.

- **Pull requests:** only **new** vulnerabilities fail the check
- **Push to default branch / Monday schedule:** full scan, `fail-on-vuln: false` so an old backlog does not keep CI red
- **SARIF:** `upload-sarif: false` on purpose. GitHub Code Scanning is not enabled on most of these repos (private GitHub Free cannot upload SARIF without Advanced Security). The scan still writes the **OSV Scanner SARIF file** Actions artifact. Public repos that later enable Code Scanning can flip that input to `true`.

Pin is `google/osv-scanner-action` **v2.6.0**.

## Why not Trivy org-wide

Trivy also flags container OS packages, IaC, and secrets. That is useful on TaskFlow images. Org-wide it is the same noise Snyk produced on learning repos. Use OSV here; keep Trivy where you build images.
