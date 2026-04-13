# selvo Security Scan Action

Scan your Linux infrastructure packages for CVEs, exploit maturity, CISA KEV status, and SLA breaches — directly in your CI pipeline.

Wraps the [`selvo`](https://github.com/Cope-Labs/selvo) CLI. **No account or API key required** — selvo is open-source and runs entirely inside the runner.

## Usage

```yaml
- uses: Cope-Labs/selvo-action@v1
  with:
    ecosystem: debian
    limit: 50
```

### Fail on critical findings

```yaml
- uses: Cope-Labs/selvo-action@v1
  with:
    ecosystem: ubuntu
    fail-on-kev: "true"
    fail-on-weaponized: "true"
    min-score: 60
```

### Scan an SBOM, scanner report, or container image

```yaml
- uses: Cope-Labs/selvo-action@v1
  with:
    sbom: ./sbom.cdx.json

- uses: Cope-Labs/selvo-action@v1
  with:
    grype-report: ./grype.json

- uses: Cope-Labs/selvo-action@v1
  with:
    image: ghcr.io/your-org/your-app:latest
```

### Use outputs in later steps

```yaml
- uses: Cope-Labs/selvo-action@v1
  id: selvo
  with:
    ecosystem: debian

- run: |
    echo "Found ${{ steps.selvo.outputs.kev-count }} KEV packages"
    echo "Max score: ${{ steps.selvo.outputs.max-score }}"
```

## Inputs

| Input | Default | Description |
|-------|---------|-------------|
| `ecosystem` | `all` | `debian`, `ubuntu`, `fedora`, `alpine`, `arch`, `all` |
| `limit` | `50` | Max packages to analyze |
| `context` | `auto` | `auto`, `local` (use runner's package manager), `reference` |
| `sbom` | — | Path to a CycloneDX/SPDX SBOM (skips discovery) |
| `grype-report` | — | Path to a Grype JSON report (skips discovery) |
| `trivy-report` | — | Path to a Trivy JSON report (skips discovery) |
| `image` | — | Container image to scan (e.g. `ubuntu:24.04`) |
| `fail-on-kev` | `false` | Fail if CISA KEV packages found |
| `fail-on-weaponized` | `false` | Fail if weaponized exploits found |
| `min-score` | `0` | Fail if any score exceeds this (0 = disabled) |
| `post-comment` | `true` | Post results as a PR comment on `pull_request` |
| `python-version` | `3.12` | Python used to install selvo |
| `selvo-version` | — | Pin a specific selvo release (default = latest) |

## Outputs

| Output | Description |
|--------|-------------|
| `total-packages` | Total packages analyzed |
| `packages-with-cves` | Packages with open CVEs |
| `kev-count` | CISA KEV packages |
| `weaponized-count` | Packages with weaponized exploits |
| `max-score` | Highest risk score |
| `passed` | `true` if all gates passed |
| `result-file` | Path to the full selvo JSON output |

## How it works

1. Installs the [`selvo`](https://pypi.org/project/selvo/) PyPI package
2. Runs `selvo analyze` (or `selvo scan` for SBOM/image inputs) with `-o json`
3. Parses results with `jq`
4. Writes a summary table to `$GITHUB_STEP_SUMMARY` and (optionally) a PR comment
5. Checks gates and exits non-zero if any fail

The full JSON result is left at `${{ steps.<id>.outputs.result-file }}` for downstream steps.

## Scaling beyond a single repo

Need fleet scanning, scheduled re-scans, SLA tracking, or Slack alerts? See [selvo.dev](https://selvo.dev).

## License

MIT
