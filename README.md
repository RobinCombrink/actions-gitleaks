# actions-gitleaks

Reusable GitHub Actions workflow that scans a repository's **full commit history** for
secrets using the MIT-licensed [gitleaks](https://github.com/gitleaks/gitleaks) CLI
(checksum-verified pinned binary — not the vendor-EULA'd gitleaks-action). Findings are
reported as SARIF to the repo's **Security → Code scanning** tab.

Part of a fleet of independently versioned reusable workflows (`actions-*` repos on this
account), released by
[actions-release](https://github.com/RobinCombrink/actions-release).

## Usage

`.github/workflows/gitleaks.yml` in the calling repo:

```yaml
name: gitleaks

on:
  pull_request:
  push:
    branches: [main]

permissions:
  contents: read
  security-events: write

concurrency:
  group: "${{ github.workflow }}-${{ github.ref }}"
  cancel-in-progress: true

jobs:
  scan:
    uses: RobinCombrink/actions-gitleaks/.github/workflows/gitleaks.yml@<commit-sha> # vX.Y.Z
```

Pin the exact commit SHA of the release you want (the `# vX.Y.Z` comment keeps it readable);
Dependabot bumps SHA and comment together.

### Private repos

Code-scanning upload requires GitHub Advanced Security on private repos. Disable the upload
and rely on the job log + failure status instead:

```yaml
    with:
      sarif-upload: false
```

## Behavior

- Scans every commit ever pushed, not just the diff — a secret committed and later deleted
  still fails the scan.
- Secrets are redacted in logs (`--redact`).
- The job fails when leaks are found; the SARIF upload still runs so findings land in code
  scanning either way.
