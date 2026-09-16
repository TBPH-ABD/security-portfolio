# Security & Full-Stack Portfolio — Salah Abdullah

Ten original security and full-stack tools, written in **Python 3 with no
third-party dependencies**. Each one solves a real problem in an assessment,
a monitoring setup, or a secure build, and each is documented with usage,
example output, and the reasoning behind its design.

**Penetration Tester | IT & Full-Stack Developer**
BSc Information Technology, University of Science and Technology (honors)

---

## Offensive security — enumeration and testing

| Project | What it does |
| --- | --- |
| **[recon-mapper](https://github.com/TBPH-ABD/recon-mapper)** | Network asset and service discovery across CIDR ranges — concurrent TCP scanning, service identification, banner grabbing, JSON reporting |
| **[web-recon-scanner](https://github.com/TBPH-ABD/web-recon-scanner)** | Passive web application audit — security headers, TLS posture, cookie attributes, form input surface, information disclosure |
| **[apitest](https://github.com/TBPH-ABD/apitest)** | REST API security testing — broken authentication, CORS misconfiguration, verbose errors, rate limiting, sensitive data exposure |

## Defensive security — detection and monitoring

| Project | What it does |
| --- | --- |
| **[sentinel-log-analyzer](https://github.com/TBPH-ABD/sentinel-log-analyzer)** | Detects SSH brute force, post-brute-force compromise, directory enumeration, and injection attempts by correlating server logs per source IP |
| **[integrity-guard](https://github.com/TBPH-ABD/integrity-guard)** | File integrity monitoring — cryptographic baselining that catches dropped web shells, tampered binaries, and unauthorised config edits |
| **[phishguard](https://github.com/TBPH-ABD/phishguard)** | Phishing URL analysis — typosquatting, homoglyph and punycode deception, credential lures, redirect-chain tracing |
| **[cred-audit](https://github.com/TBPH-ABD/cred-audit)** | Password auditing scored on real offline-cracking resistance, with privacy-preserving breach corpus checks (k-anonymity) |

## Vulnerability management

| Project | What it does |
| --- | --- |
| **[vulnfeed](https://github.com/TBPH-ABD/vulnfeed)** | CVE lookup and triage against the NIST NVD — severity filtering, network-exploitability flags, local caching |

## Full-stack

| Project | What it does |
| --- | --- |
| **[securenotes](https://github.com/TBPH-ABD/securenotes)** | A web application built to demonstrate secure coding — PBKDF2 hashing, hashed session tokens, CSRF protection, IDOR-safe queries, strict CSP |
| **[netwatch](https://github.com/TBPH-ABD/netwatch)** | Network service monitoring with a live dashboard, SQLite history, and a JSON API — no framework, no build step |

---

## Principles behind every project

**Zero dependencies.** Every tool runs on a clean Python 3.10+ install. Nothing
to `pip install`, nothing to audit, nothing to break.

**Findings explain themselves.** Each one states not just *what* was found but
*why it matters*, so output goes straight into an assessment report.

**Built for automation.** Tools exit non-zero on significant findings, so they
drop into CI pipelines and cron jobs without a wrapper script.

**Authorization is explicit.** Every tool that touches a remote target requires
an `--authorized` flag. Testing systems without permission is illegal, and the
flag makes that decision deliberate rather than accidental.

**Honest about limits.** Where a problem is better solved outside the
application — encryption at rest, TLS termination, reputation feeds — the
README says so rather than shipping a weak imitation.

## License

All projects are MIT licensed.
