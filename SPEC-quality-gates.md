# Spec: quality-gates

## Objective

Turn "the tests pass" into an enforced standard, so coverage cannot quietly
erode as the tools grow.

Depends on `ci-pipeline`. This module is last in the build order for a reason:
a coverage threshold can only be calibrated once real coverage numbers exist.
Picking a number before measuring is guessing.

## Tech Stack

`coverage.py`, installed **inside the CI workflow only**. It is never shipped
with a tool, so the runtime stays dependency-free and every README's
"no third-party dependencies" claim remains true.

## Commands

```
Measure:        python -m coverage run -m unittest discover -s tests
Report + gate:  python -m coverage report
Line detail:    python -m coverage report -m
HTML detail:    python -m coverage html
```

`fail_under` lives in `.coveragerc`, so `coverage report` is itself the gate —
it exits non-zero below the threshold with no extra flag at the call site.

## Project Structure

```
<repo>/.coveragerc
```

## Code Style

```ini
[report]
fail_under = 80
show_missing = True

exclude_lines =
    pragma: no cover
    if __name__ == .__main__.:
```

Exclusions are limited to lines that genuinely cannot be unit tested: the CLI
entry guard, and defensive branches that no test can provoke. Anything else
gets a test instead of an exclusion.

## Testing Strategy

**The threshold is 80%, chosen after measuring.** Actual coverage at the time
this was written:

| Tool | Coverage |
| --- | --- |
| recon-mapper | 98% |
| integrity-guard | 97% |
| cred-audit | 97% |
| apitest | 95% |
| sentinel-log-analyzer | 95% |
| vulnfeed | 95% |
| phishguard | 94% |
| securenotes | 86% |
| web-recon-scanner | 84% |
| netwatch | 81% |

Every repository clears 80% with headroom, so the gate protects against
regression without blocking honest work. The uncovered remainder is
concentrated in `main()` argument wiring and console formatting — the parts
where a test asserts on layout rather than behaviour, which is a test worth
skipping.

## Boundaries

**Always**
- Raise the threshold when coverage rises durably
- Add a test rather than an `exclude_lines` entry
- Keep `coverage` confined to the CI workflow

**Ask first**
- Adding a new exclusion pattern
- Adding a linter or type checker (each is a new dev dependency and a new
  class of build failure)

**Never**
- Lower `fail_under` to make a build pass
- Add `coverage` to a tool's runtime requirements
- Exclude a module from measurement to lift the number

## Success Criteria

1. `.coveragerc` exists in all ten repositories with `fail_under = 80`.
2. `python -m coverage report` exits non-zero when coverage drops below 80%.
3. All ten repositories pass the gate.
4. No tool gains a runtime dependency.

## Open Questions

**Status badges are deliberately absent.** A GitHub Actions badge only renders
for a public repository; on a private repo it shows as broken. If the
portfolio is ever made public, add to each README:

```markdown
![tests](https://github.com/TBPH-ABD/<repo>/actions/workflows/tests.yml/badge.svg)
```

**Linting and type checking were considered and deferred.** Both would add a
dev dependency and a new class of build failure for a codebase this size.
Revisit if the tools grow or gain contributors.
