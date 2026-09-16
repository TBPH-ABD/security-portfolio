# Spec: ci-pipeline

## Objective

Every push and pull request runs the full test suite automatically, on every
supported Python version, so a regression is caught by the repository rather
than by whoever clones it next.

Depends on `test-suites` — a workflow with nothing to run is not shippable.

## Tech Stack

GitHub Actions. `actions/checkout@v4`, `actions/setup-python@v5`, both pinned
to a major version.

## Commands

The workflow runs exactly what a developer runs locally:

```
python -m unittest discover -s tests -v
python -W error::ResourceWarning -m unittest discover -s tests
python -m compileall -q .
```

## Project Structure

```
<repo>/.github/workflows/tests.yml
```

One workflow per repository. The repos are independent, so a shared reusable
workflow would introduce a cross-repo dependency for no benefit at this size.

## Code Style

Three jobs, each answering one question:

| Job | Question it answers |
| --- | --- |
| `test` | Does the suite pass on every supported Python? |
| `coverage` | Is enough of the code actually exercised? |
| `compile` | Does the code parse on the oldest supported Python? |

The matrix runs **3.10, 3.11, 3.12, 3.13** with `fail-fast: false`, so one
version failing still reports the others. 3.10 is the floor because the code
uses `X | None` annotations.

`permissions: contents: read` is declared explicitly — the workflow only reads
the repository, and the default token grant is wider than it needs.

## Testing Strategy

The `test` job runs the suite twice: once verbosely for a readable log, and
once with `-W error::ResourceWarning` so a leaked socket, file, or database
connection fails the build. That second run is what caught four real leaks
during `test-suites`.

There is **no install step** in the `test` job. The tools and their tests are
standard library only, so there is nothing to install — which also means no
dependency resolution to break the build.

## Boundaries

**Always**
- Pin action versions
- Declare `permissions` explicitly
- Keep the local command and the CI command identical

**Ask first**
- Adding a job that needs write permissions or secrets
- Adding a dependency to the `test` job
- Dropping a Python version from the matrix

**Never**
- Put a credential in a workflow file
- Let CI reach an external service that the tests are not allowed to reach
- Mark a job `continue-on-error` to get a green tick

## Success Criteria

1. `tests.yml` exists in all ten repositories.
2. The suite passes on 3.10 through 3.13.
3. A resource leak fails the build.
4. A syntax error under 3.10 fails the build.
5. No job requires an installed dependency except `coverage` in the coverage job.

## Open Questions

**Status badges.** A README badge only renders for a public repository. The
repos are private by decision, so badges are deliberately not added — see
[SPEC-quality-gates.md](SPEC-quality-gates.md).
