# ModuForge

**A secure, module-scoped multi-agent software development framework with human-gated cloud access.**

> ModuForge is an open architecture for running specialized AI development agents — each owning a specific software module — inside isolated Docker environments, with automated security gates, risk-scored human review, and just-in-time access to multi-cloud infrastructure.

---

## Why ModuForge?

Most multi-agent AI dev frameworks assign agents to **lifecycle phases** (design → code → test). ModuForge assigns agents to **software modules** (frontend, backend, data layer) — the way real engineering teams are actually structured.

| Existing frameworks | ModuForge |
|---|---|
| Phase-scoped agents (ChatDev, MetaGPT, AgentMesh) | **Module-scoped agents** — true parallel development |
| Single shared runtime or Docker Compose | **Per-agent isolated containers** — hardened sandboxes |
| No access control model | **Humans as the explicit access control plane** |
| Long-lived credentials for cloud access | **JIT tokens** — task-bound, auto-expiring |
| All PRs reviewed by humans | **Risk-scored triage** — humans only review what matters |
| No audit model | **Immutable audit log** — every action traceable |

---

## Architecture Overview

ModuForge is built on 8 defense-in-depth layers:

```
Layer 1  →  Human Governance          Policy, approvals, incident response
Layer 2  →  Orchestration             Signed task & artifact routing
Layer 3  →  Zero-Trust Gate           Identity, policy, scope — verified every call
Layer 4  →  Sandboxed Agent Docker    Non-root, read-only FS, no internet, signed images
Layer 5  →  Security Pipeline         SAST → Secret Detection → SCA → DAST
Layer 6  →  QA + Human Review Gate    Risk-scored — auto-merge / QA agent / human / dual-human
Layer 7  →  JIT Access + Audit Log    Short-lived tokens, WORM audit trail
Layer 8  →  Multi-Cloud Infrastructure Never accessed with direct agent credentials
```

See [`/docs/architecture.md`](/docs/architecture.md) for full Mermaid diagrams of all 8 layers, the security pipeline, JIT access flow, debugging flow, and human review decision tree.

---

## Key Concepts

### Module-Scoped Agents

Each agent owns one software module and runs in its own Docker container:

| Agent | Module | Responsibilities |
|---|---|---|
| `frontend-agent` | UI layer | Components, state management, API integration |
| `backend-agent` | Service layer | REST/GraphQL APIs, business logic, auth |
| `data-agent` | Data layer | Schema design, queries, migrations |
| `review-agent` | QA | Static analysis, security patterns, style |
| `test-agent` | QA | Unit tests, integration tests, coverage gating |

### Dual Orchestrators

Two independent orchestrators allow QA to scale without touching the dev pipeline:

- **Task Orchestrator** — routes work to dev agents via cryptographically signed task manifests
- **QA Orchestrator** — routes artifacts to QA agents via signed artifact routing

### Zero-Trust Gate

Every agent request is verified in real time. No standing trust. No shared service accounts.

- Identity and token verification on every call
- OPA (Open Policy Agent) policy evaluation
- Rate and scope enforcement — blast radius control

### Risk-Scored Human Review

AI generates code 4× faster than humans can review it. ModuForge solves this with automated risk scoring:

| Score | Route | Criteria |
|---|---|---|
| 0 – 30 LOW | Auto-merge | Clean scans, small diff, peripheral module |
| 31 – 70 MEDIUM | QA agent review | Minor findings, moderate diff size |
| 71 – 100 HIGH | Human required | Auth/DB/payment code, scan warnings, coverage drop |
| CRITICAL | Two humans + audit | Any security-flagged file in core modules |

### JIT Access Broker

Agents never hold long-lived cloud credentials. Every cloud access is:

1. Requested against a specific task manifest
2. Verified by the zero-trust gate
3. Issued as a short-lived scoped token (TTL: 15–60 min)
4. Auto-expired — no manual revocation needed
5. Fully logged to the immutable audit trail

### Immutable Audit Log

Every agent action, every token issued, every scan result, every human decision is written to a tamper-proof WORM log. When something goes wrong in production, you can reconstruct the exact sequence in seconds.

---

## Security Pipeline

All code artifacts pass through four automated gates before reaching QA or any human:

```
SAST (Semgrep / Snyk)       →  Static vulnerability scan, blocks on HIGH/CRITICAL
Secret Detection (Gitleaks)  →  Leaked credentials, zero-tolerance, pre-commit + CI
SCA (Snyk / OWASP)          →  Dependency CVEs and license violations
DAST (OWASP ZAP)            →  Runtime behavior, catches what SAST misses
```

Only artifacts that pass all four gates proceed to risk scoring.

---

## Docker Container Hardening

Every agent container — dev and QA — is hardened identically:

```yaml
# docker-compose.agent.yml (template)
services:
  agent:
    image: your-registry/agent-name:sha256-<digest>
    read_only: true
    tmpfs:
      - /tmp
    user: "1000:1000"
    cap_drop:
      - ALL
    security_opt:
      - no-new-privileges:true
    mem_limit: 512m
    cpus: "0.5"
    networks:
      - agent-internal    # no internet egress
```

No agent container has outbound internet access. All communication flows through the orchestration layer.

---

## Repository Structure

```
modulforge/
├── README.md
├── docs/
│   ├── architecture.md          # All 6 Mermaid architecture diagrams
```

---

## Threat Model

ModuForge is designed to contain damage when (not if) an agent is compromised.

| Threat | Mitigation |
|---|---|
| Compromised agent exfiltrates data | No internet egress, JIT tokens scope-limited to task |
| Agent prompt injection via malicious code | Signed task manifests, zero-trust gate rejects unsigned instructions |
| AI generates vulnerable code | SAST/DAST blocks before any human or QA sees it |
| Leaked credentials in generated code | Gitleaks pre-commit hook + CI gate, zero-tolerance policy |
| Supply chain attack via AI-pulled dependency | SCA blocks CVE-flagged packages on every dependency change |
| Agent escalates privileges inside container | `cap_drop=ALL`, `no-new-privileges`, non-root UID |
| Incident with no trace | Immutable WORM audit log captures every action end-to-end |
| Human review bottleneck | Risk scoring routes ~80% of PRs away from humans automatically |

---

## Toolchain

| Component | Open Source Option | Commercial Option |
|---|---|---|
| Zero-trust / identity | Keycloak | BeyondTrust |
| Container hardening | gVisor, Falco | Aqua Security, Sysdig |
| SAST | Semgrep, CodeQL | Snyk Code |
| DAST | OWASP ZAP | Burp Suite Enterprise |
| Secret detection | Gitleaks, TruffleHog | GitGuardian |
| SCA | OWASP Dependency-Check | Snyk Open Source |
| Policy engine | OPA (Open Policy Agent) | HashiCorp Sentinel |
| JIT access | HashiCorp Vault | CyberArk |
| Audit log | OpenSearch + WORM | Splunk, Datadog |

---

## Prior Art & Innovation

ModuForge builds on and extends existing work:

- **ChatDev** (2023) — phase-scoped agents in a virtual software company
- **MetaGPT** (2023) — structured outputs between role-based agents
- **AgentMesh** (2025) — Planner / Coder / Debugger / Reviewer pipeline
- **HULA** (Atlassian, 2024) — human-in-the-loop agents inside JIRA

**What ModuForge adds that none of these have:**

1. Module-scoped agent ownership (not phase-scoped)
2. Per-agent hardened Docker sandboxes with no internet egress
3. Dual independent orchestrators (task + QA)
4. Humans as the explicit access control plane to cloud infrastructure
5. Five-stage automated security pipeline before any human sees code
6. Risk-scored PR triage — humans review HIGH-risk changes only
7. JIT access broker with task-bound, auto-expiring tokens
8. Immutable WORM audit log for every agent action

---

## Consequences of Skipping Security Layers

| Skipped layer | Consequence |
|---|---|
| Zero-trust gate | One compromised agent acts with full system privileges |
| Container hardening | Agent can exfiltrate data or call external endpoints |
| SAST / DAST | 45% of AI-generated code ships with unreviewed vulnerabilities |
| Secret detection | Credentials embedded in code — one leaked key = breach |
| JIT access | Long-lived credentials are high-value targets; breach = unlimited access |
| Audit log | Incidents are undiagnosable; impossible to prove what happened |
| Risk scoring | Human reviewers drown in low-risk PRs and miss the critical ones |

---

## Quick-Start Checklist

- [ ] All agent containers run as non-root with read-only filesystems
- [ ] No agent container has outbound internet access
- [ ] Every agent request passes identity verification (no shared service accounts)
- [ ] SAST runs on every commit; pipeline blocks on HIGH/CRITICAL
- [ ] Secret detection is a pre-commit hook (catch before it's ever committed)
- [ ] SCA runs on every dependency change
- [ ] Agents use JIT tokens (TTL ≤ 60 min), never long-lived credentials
- [ ] Every agent action is written to an immutable audit log
- [ ] PRs are risk-scored; only HIGH-risk PRs require human review
- [ ] Humans own policy definition and can override any agent at any time
- [ ] Incident response runbook exists for: compromised agent, leaked credential, scan spike

---

## Viewing the Architecture Diagrams

The full architecture is in [`/docs/architecture.md`](/docs/architecture.md) as Mermaid diagrams. They render automatically on GitHub. To view locally:

- **VS Code** — install `Markdown Preview Mermaid Support`, then `Cmd+Shift+V`
- **Obsidian** — drop the file into any vault, Mermaid renders natively
- **Mermaid Live** — paste individual diagram code blocks at [mermaid.live](https://mermaid.live)

---

## License

MIT — see [`LICENSE`](/LICENSE)

---

*ModuForge — Module-scoped. Sandboxed. Human-governed.*
*Architecture version 2.0 — May 2026*
