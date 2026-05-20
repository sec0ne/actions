# Sec1 GitHub Actions Integration

## Overview

This GitHub Action integrates [Sec1](https://sec1.io/) to run **SCA (FOSS)** and **SAST** security scans on your GitHub projects. Both scans run by default — use the inputs below to customize behavior.

## Quick start

```yaml
name: Sec1 Security
on: push
jobs:
  security:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Run Sec1 Scan
        uses: sec0ne/actions/security@main
        with:
          apikey: ${{ secrets.SEC1_API_KEY }}
```

By default this runs **both SCA and SAST** scans.

## Inputs

| Input | Description | Default |
|---|---|---|
| `apikey` | **Required.** Your Sec1 API Key. | – |
| `runSca` | Run SCA (FOSS) scan. | `true` |
| `runSast` | Run SAST scan. | `true` |
| `scanTag` | Tag to identify this scan. | branch name |
| `scmUrl` | Override the SCM URL (auto-detected by default). | – |
| `sastIncrementalScan` | Run SAST in incremental mode (requires a baseline full scan). | `false` |
| `asyncScan` | Fire-and-forget mode. Submit scan and exit without waiting. Ignored if a threshold is configured. | `false` |
| `scanThreshold` | Per-severity thresholds (e.g. `critical=1 high=5 medium=10 low=50`). | – |
| `actionOnThresholdBreached` | What to do if a threshold is breached: `fail` \| `unstable` \| `continue`. `unstable` emits a warning annotation but does not fail the build. | `fail` |
| `instanceUrl` | Custom Sec1 API URL. | `https://api.sec1.io` |
| `dashboardUrl` | Custom Sec1 Dashboard URL (used for report links). | `https://unified.sec1.io` |
| `scanType` | **Deprecated.** Use `runSca`/`runSast`. Accepted values: `foss`, `sast`, `both`. | – |

## Outputs

| Output | Description |
|---|---|
| `reportUrl` | URL of the Sec1 scan report. |
| `critical` | Number of critical vulnerabilities. |
| `high` | Number of high vulnerabilities. |
| `medium` | Number of medium vulnerabilities. |
| `low` | Number of low vulnerabilities. |
| `totalCve` | Total number of vulnerabilities. |
| `scanStatus` | Final scan status: `COMPLETED`, `SUBMITTED` (async), or `FAILED`. |

## Examples

### Run only SCA

```yaml
- uses: sec0ne/actions/security@main
  with:
    apikey: ${{ secrets.SEC1_API_KEY }}
    runSast: "false"
```

### Run only SAST

```yaml
- uses: sec0ne/actions/security@main
  with:
    apikey: ${{ secrets.SEC1_API_KEY }}
    runSca: "false"
```

### Run on pull requests

```yaml
name: Sec1 PR check
on:
  pull_request:
    branches: [main, master]
jobs:
  security:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: sec0ne/actions/security@main
        with:
          apikey: ${{ secrets.SEC1_API_KEY }}
```

### Fail the build on threshold breach

```yaml
- uses: sec0ne/actions/security@main
  with:
    apikey: ${{ secrets.SEC1_API_KEY }}
    scanThreshold: "critical=1 high=5"
    actionOnThresholdBreached: fail
```

### Mark the build unstable instead of failing

```yaml
- uses: sec0ne/actions/security@main
  with:
    apikey: ${{ secrets.SEC1_API_KEY }}
    scanThreshold: "critical=1 high=5"
    actionOnThresholdBreached: unstable
```

`unstable` emits a `::warning::` annotation that shows up on the workflow run summary but does not fail the step.

### SAST in incremental mode

Incremental scans only analyze changed code. They require a baseline (full) scan to exist on the Sec1 server.

```yaml
- uses: sec0ne/actions/security@main
  with:
    apikey: ${{ secrets.SEC1_API_KEY }}
    sastIncrementalScan: "true"
```

### Fire-and-forget (async) mode

Submit the scan and return immediately. Useful for long-running scans on large repos.

```yaml
- uses: sec0ne/actions/security@main
  with:
    apikey: ${{ secrets.SEC1_API_KEY }}
    asyncScan: "true"
```

Note: `asyncScan` is ignored when `scanThreshold` is set, because threshold evaluation needs the final scan counts.

### Custom Sec1 endpoints (self-hosted / on-prem)

```yaml
- uses: sec0ne/actions/security@main
  with:
    apikey: ${{ secrets.SEC1_API_KEY }}
    instanceUrl: https://api.sec1.example.com
    dashboardUrl: https://dashboard.sec1.example.com
```

### Tag the scan explicitly

```yaml
- uses: sec0ne/actions/security@main
  with:
    apikey: ${{ secrets.SEC1_API_KEY }}
    scanTag: release-2026-q2
```

### Use scan outputs in a later step

```yaml
- name: Run Sec1 Scan
  id: sec1
  uses: sec0ne/actions/security@main
  with:
    apikey: ${{ secrets.SEC1_API_KEY }}

- name: Post Sec1 results to Slack
  if: steps.sec1.outputs.totalCve != '0'
  run: |
    echo "Sec1 found ${{ steps.sec1.outputs.totalCve }} vulnerabilities."
    echo "Critical: ${{ steps.sec1.outputs.critical }}"
    echo "Report: ${{ steps.sec1.outputs.reportUrl }}"
```

### Override the SCM URL

By default the action uses `$GITHUB_SERVER_URL/$GITHUB_REPOSITORY`. Override when auto-detection isn't what you want:

```yaml
- uses: sec0ne/actions/security@main
  with:
    apikey: ${{ secrets.SEC1_API_KEY }}
    scmUrl: https://github.com/your-org/your-repo
```

## Step summary

When the scan completes, a markdown summary is added to the workflow run's summary page with vulnerability counts and the report link — no need to dig through logs.

## Getting your Sec1 API Key

1. Visit [Sec1 portal](https://scopy.sec1.io/) and log in.
2. Go to **Settings** → **API key** → **Generate API key**.
3. Copy the key and add it to your repo or org [GitHub Actions secrets](https://docs.github.com/en/actions/security-guides/using-secrets-in-github-actions).

## Migration from older versions

If you were using:

```yaml
with:
  apikey: ...
  scanType: sast
```

Switch to:

```yaml
with:
  apikey: ...
  runSca: "false"
  runSast: "true"
```

The `scanType` input still works for back-compat but is deprecated.

## Troubleshooting

- **`401 Unauthorized`** — check that `apikey` is set and valid.
- **`No SCAN_DIRECTORY and no SCM URL`** — ensure the action runs in a checked-out repo (`actions/checkout` before this step) or set `scmUrl` explicitly.
- **Scan times out** — scans poll for 30 minutes max. For large repos use `asyncScan: "true"` (without `scanThreshold`) to fire-and-forget.

---

— Sec1 team
