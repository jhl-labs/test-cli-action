# test-cli-action

GitHub Action for running [`test-cli`](https://github.com/jhl-labs/test-cli) in
CI: it installs the binary, runs standardized tests with coverage across
**python, typescript, go, rust, c#, and java**, publishes failing tests as
workflow annotations, writes a Markdown job summary, and exposes
machine-readable outputs for AI agents and downstream gates.

```yaml
- name: Run standardized tests & coverage
  id: tests
  uses: jhl-labs/test-cli-action@main
  with:
    profile: ci
    target: .
    fail-under: "80"
```

The action installs the latest public `test-cli` binary through the
Pages-hosted installer (with the raw script as a fallback):

```bash
curl -fsSL https://jhl-labs.github.io/test-cli/install.sh | bash
```

It then runs `test-cli run`, reads the normalized `report.json`, emits
`::error` annotations for each failing test, and appends `report.md` to the
GitHub Actions job summary.

> Set up the language toolchain of the project under test **before** this action
> (e.g. `actions/setup-go`, `actions/setup-node`, `actions/setup-python`), so
> `test-cli` can invoke the native test runner.

## Inputs

| Input | Default | Description |
|---|---|---|
| `token` | `${{ github.token }}` | Token for checkout and install. |
| `checkout` | `true` | Checkout the repo before running. Set `false` if you already checked out. |
| `repository` / `ref` / `fetch-depth` | _(context)_ | Checkout parameters. |
| `version` | `latest` | test-cli version to install (e.g. `v0.1.0`). |
| `command` | `run` | test-cli command (`run`, `ingest`, `report`). |
| `target` | `.` | Path/target passed to test-cli. |
| `profile` | `ci` | `default` · `ci` · `release`. |
| `languages` | _(auto)_ | Comma-separated language restriction. |
| `output-dir` | `reports/test` | Where reports are written. |
| `formats` | `stdout,json,junit,cobertura,markdown,html` | Output formats. |
| `fail-under` | `0` | Fail if line coverage `< PCT`. |
| `annotation-limit` | `100` | Max failure annotations. |
| `job-summary` | `true` | Append `report.md` to the job summary. |
| `fail-on-test-failure` | `true` | Fail the job on test failure / coverage gate. |
| `extra-args` | `""` | Extra args appended to the command. |

## Outputs

| Output | Description |
|---|---|
| `passed` | `true` when green and the coverage gate is satisfied. |
| `tests-total` | Total tests executed. |
| `tests-failed` | Failing tests (failures + errors). |
| `coverage-pct` | Total line coverage percentage. |
| `exit-code` | Raw test-cli exit code. |
| `report-dir` | Directory containing the reports. |

## Example

```yaml
name: Tests
on: [push, pull_request]
permissions:
  contents: read
  checks: write
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-go@v5
        with: { go-version-file: go.mod }
      - id: tests
        uses: jhl-labs/test-cli-action@main
        with:
          checkout: "false"
          profile: ci
          fail-under: "80"
      - if: always()
        run: echo "coverage=${{ steps.tests.outputs.coverage-pct }}% passed=${{ steps.tests.outputs.passed }}"
      - if: always()
        uses: actions/upload-artifact@v4
        with:
          name: test-report
          path: reports/test
```

## Relationship to test-cli

This repository is the thin Action wrapper. The CLI, its report schema, and the
HTML visualizations live in [jhl-labs/test-cli](https://github.com/jhl-labs/test-cli)
— mirroring the `security-cli` / `security-cli-action` split.

## License

MIT
