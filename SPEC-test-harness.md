# Spec: test-harness

## Objective

Establish one testing pattern that every tool in the portfolio follows, so the
ten test suites read as one system rather than ten personal styles.

The hard problem it solves: **six of the ten tools perform network I/O**
(`recon-mapper`, `web-recon-scanner`, `cred-audit`, `netwatch`, `vulnfeed`,
`apitest`). Their tests must exercise that code without ever contacting a real
external service. A test that depends on `example.com` or the NVD API is slow,
flaky, and fails whenever a third party has an outage.

Success: a developer can open any of the ten repos, run one command, and get a
deterministic pass in under ten seconds with no network and nothing installed.

## Tech Stack

- Python 3.10+ (`X | None` annotations require it)
- `unittest` — stdlib test framework
- `unittest.mock` — stdlib, for patching module-level URL constants
- `http.server`, `socket`, `threading` — stdlib, for the local fakes

No third-party packages at any point.

## Commands

```
Run one repo's tests:   python3 -m unittest discover -s tests -v
Run a single module:    python3 -m unittest tests.test_phishguard -v
Run a single test:      python3 -m unittest tests.test_phishguard.TestScoring.test_typosquat -v
Compile check:          python3 -m compileall -q .
```

## Project Structure

Identical in all ten repositories:

```
<repo>/
  <tool>.py              → the tool (unchanged)
  tests/
    __init__.py          → empty; makes tests a package so discovery works
    support/
      __init__.py
      fakes.py           → the shared harness (copied verbatim into each repo)
    test_<tool>.py       → the tool's test suite
```

`support/fakes.py` is **copied**, not shared through a package. Each repo must
stay standalone and clonable on its own, and a shared package would be exactly
the third-party dependency the portfolio promises not to have. The duplication
is ~90 lines and is accepted deliberately.

## Code Style

Two fakes, both context managers yielding a real address to connect to.

```python
# A real listening TCP socket on an ephemeral port.
with tcp_listener(banner=b"SSH-2.0-OpenSSH_9.6") as (host, port):
    is_open, banner = recon_mapper.is_port_open(host, port, timeout=2.0)
    self.assertTrue(is_open)
    self.assertIn("OpenSSH", banner)

# A real HTTP server serving programmed routes.
routes = {"/range/5BAA6": (200, {}, "1E4C9B93F3F0682250B6CF8331B7EE68FD8:99\r\n")}
with http_fake(routes) as base_url:
    with mock.patch.object(cred_audit, "HIBP_RANGE_URL", base_url + "/range/"):
        self.assertEqual(cred_audit.check_breach_corpus("password"), 99)
```

Conventions:

- One `TestCase` class per behaviour area, named `Test<Area>`.
- Test methods named `test_<condition>_<expected outcome>`.
- Assert on **behaviour**, never on log text or formatting.
- Every test is independent; no shared mutable state, no ordering dependency.
- Temporary files go in `tempfile.TemporaryDirectory()`, cleaned up in `tearDown`.

## Testing Strategy

| Concern | Approach |
| --- | --- |
| Pure logic (scoring, parsing, diffing, entropy) | Direct calls, table-driven assertions |
| Outbound HTTP | `http_fake` + `mock.patch.object` on the module's URL constant |
| Outbound TCP | `tcp_listener` on an ephemeral port |
| Databases (`netwatch`, `securenotes`) | Real SQLite in a `TemporaryDirectory` — not mocked |
| Filesystem (`integrity-guard`) | Real files in a `TemporaryDirectory` |
| CLI `main()` and argparse | Called with an explicit `argv` list, asserting the exit code |

Ephemeral ports (`bind(("127.0.0.1", 0))`) are mandatory — fixed port numbers
make tests fail when run in parallel or when a developer has something running.

## Boundaries

**Always**
- Bind fakes to `127.0.0.1` on an ephemeral port
- Clean up servers and temp directories in `tearDown` or a context manager
- Keep each test deterministic and independent

**Ask first**
- Adding any third-party package
- Changing a tool's production behaviour to make it easier to test
- Adding a test that takes longer than two seconds

**Never**
- Contact a real external service (`example.com`, NVD, HIBP, any live host)
- Scan or connect to any host the test does not itself start
- Hard-code a port number
- Weaken an assertion to make a failing test pass

## Success Criteria

1. `tests/support/fakes.py` exists identically in all ten repositories.
2. `tcp_listener` yields a connectable address and can serve a banner.
3. `http_fake` serves programmed status codes, headers, and bodies, and records
   the requests it received so tests can assert on them.
4. Both fakes shut down cleanly with no leaked threads or sockets.
5. A self-test suite for the harness itself passes.
6. Running the full suite with the network disabled still passes.

## Open Questions

None. Framework (`unittest`) and coverage approach (`coverage.py`, CI only)
were decided before this spec was written; both are recorded in
[CAPABILITY-MAP.md](CAPABILITY-MAP.md).
