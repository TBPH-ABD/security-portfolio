# Capability Map: Portfolio Quality Hardening

Adding automated tests and CI to the ten portfolio tools.

| Module id | Responsibility | Depends on |
|---|---|---|
| `test-harness` | Shared testing conventions: directory layout, naming, and the local fake-server pattern that keeps tests off the real internet | — |
| `test-suites` | Unit tests for the logic across all ten tools | `test-harness` |
| `ci-pipeline` | GitHub Actions workflow per repository, Python version matrix | `test-suites` |
| `quality-gates` | Coverage measurement and threshold, static checks, README status badges | `ci-pipeline` |

**Build order:** `test-harness` → `test-suites` → `ci-pipeline` → `quality-gates`

## Why these boundaries

**`test-harness` is separated because six of the ten tools perform network
I/O.** Writing test suites before a shared isolation pattern exists would
produce six different ad-hoc approaches that then have to be rewritten. The
pattern is cheap to agree on now and expensive to retrofit later.

**`ci-pipeline` depends on `test-suites`** because a workflow with nothing to
run is not shippable. Tests, however, are useful on their own — they can be run
locally the day they are written.

**`quality-gates` is last** because a coverage threshold can only be calibrated
once real coverage numbers exist. Choosing a number before measuring is guessing.

## Fixed decisions

| Decision | Choice | Reason |
|---|---|---|
| Test framework | `unittest` (stdlib) | Every tool's README promises "no third-party dependencies". Using pytest would break that promise. |
| Coverage tool | `coverage.py`, **CI only** | Not available in the stdlib. Installed inside the workflow, never shipped with a tool, so runtime stays zero-dependency. |
| Network in tests | Forbidden | Tests bind local fakes on ephemeral ports. No test may contact a real external service. |
| Test location | `tests/` inside each repository | Each repo stays standalone and independently clonable. |
| Python matrix | 3.10, 3.11, 3.12, 3.13 | The code uses `X | None` annotations, which require 3.10+. |
| Repo visibility | Private (unchanged) | Actions works on private repos. Note: README status badges only render publicly. |

## Scope

Ten repositories: `recon-mapper`, `web-recon-scanner`, `sentinel-log-analyzer`,
`integrity-guard`, `cred-audit`, `netwatch`, `vulnfeed`, `securenotes`,
`apitest`, `phishguard`.

Measured baseline: 2,683 code lines, 115 functions. Four tools
(`sentinel-log-analyzer`, `integrity-guard`, `securenotes`, `phishguard`)
make no network calls and are testable with no fakes at all.
