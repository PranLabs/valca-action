# Valca Security Scan — GitHub Action

Runs [Valca](https://github.com/PranLabs/valca) in CI and uploads the results to
GitHub code scanning.

Valca catches insecure code, infrastructure-as-code and AI-agent configuration —
including the docker-compose port binding that exposes a database to the internet,
prompt injection reachable from untrusted input, and unsafe Model Context Protocol
server configuration.

## Usage

```yaml
- uses: PranLabs/valca-action@v1
  with:
    path: .
    severity: HIGH
```

Uploading to code scanning needs `security-events: write`:

```yaml
permissions:
  contents: read
  security-events: write

steps:
  - uses: actions/checkout@v7

  - uses: PranLabs/valca-action@v1
    id: valca
    with:
      severity: HIGH

  - uses: github/codeql-action/upload-sarif@v4
    if: always()
    with:
      sarif_file: valca-results.sarif
      category: valca
```

A complete workflow is in [`workflow-template.yml`](workflow-template.yml).

## Inputs

| Input | Default | Description |
|---|---|---|
| `path` | `.` | File or directory to scan |
| `severity` | `HIGH` | Minimum severity reported: `CRITICAL`, `HIGH`, `MEDIUM`, `LOW`, `INFO` |
| `sarif-output` | `valca-results.sarif` | Where to write the SARIF file |
| `fail-on-findings` | `true` | Exit non-zero when CRITICAL or HIGH findings are present |
| `version` | *(latest)* | Valca version to install. **Pin this for reproducible CI** |

## Outputs

| Output | Description |
|---|---|
| `findings-count` | Findings at or above the severity threshold |
| `sarif-file` | Path to the generated SARIF file |

## Pinning

`version` is empty by default, so CI installs the newest Valca. That means a new
release can change your build's behaviour without you changing anything. For
reproducible pipelines, pin it:

```yaml
- uses: PranLabs/valca-action@v1
  with:
    version: "0.6.0"
```

## Exit codes

| Code | Meaning |
|---|---|
| `0` | No findings at or above the threshold |
| `2` | CRITICAL or HIGH findings present — the step fails unless `fail-on-findings: false` |

## Note for anyone who copied an earlier template

Versions of this template published before 2026-09-27 ran `pip install vigilsec`.
That is the pre-rename package, last released at 0.2.1 in July 2026, and it has
100 rules rather than 116 — missing the whole AI-agent set. If your workflow still
says `pip install vigilsec`, change it to `pip install valca` or switch to this
action. A release cannot fix a workflow file inside your repository.

## Licence

[BUSL-1.1](LICENSE). Production use is permitted, including commercial use, except
for offering Valca to third parties on a hosted or embedded basis in competition
with its paid offerings. Each version converts to MIT four years after release.
