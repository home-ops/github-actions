# github-actions

Reusable GitHub Actions workflows for home-ops repositories.

## DHI CVE delta

`.github/workflows/dhi-cve-delta.yml` is a reusable workflow that comments the
Trivy CVE delta for `dhi.io` image digest updates in Renovate pull requests.

### Caller contract

- Secrets: `DHI_USER` and `DHI_TOKEN` (dhi.io registry credentials). Both are required.
- Permissions: `contents: read` and `pull-requests: write`; nothing else.
- Trigger: `pull_request_target` (`opened`, `reopened`, `synchronize`).
- Scope: only same-repository Renovate branches starting with `renovate/dhi.io-`.
  Fork PRs and other branches are skipped.
- Pin the workflow to a full commit SHA, never a branch or tag.

```yaml
name: DHI CVE Delta

on:
  pull_request_target:
    types: [opened, reopened, synchronize]

permissions:
  contents: read
  pull-requests: write

jobs:
  dhi-cve-delta:
    uses: <owner>/github-actions/.github/workflows/dhi-cve-delta.yml@<full-40-char-commit-sha>
    secrets:
      DHI_USER: ${{ secrets.DHI_USER }}
      DHI_TOKEN: ${{ secrets.DHI_TOKEN }}
```

### Operation

- The PR diff is fetched through the GitHub API before registry login. The PR head is never checked out and no caller code runs.
- The workflow checks out only its own source at `job.workflow_repository` / `job.workflow_sha`.
- Registry credentials live in a temporary `DOCKER_CONFIG` (mode 700), mounted read-only into Trivy, and removed in an always-run step.
- One comment, identified by the `<!-- dhi-cve-delta -->` marker, is created or replaced on each run, including after a scan failure (it then reports that no comparison is available).
- A failed scan still fails the job; the failure comment does not turn it green.
- Severity output is informational only. It never fails the check.

### Tests

```sh
python3 -m unittest discover -s actions/dhi-cve-delta -p 'test_*.py' -v
```
