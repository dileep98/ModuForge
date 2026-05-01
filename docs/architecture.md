# Revised Multi-Agent Software Development Architecture
### With Security, Debugging, Human Access Control & Cloud Integration

> Paste any code block below into [https://mermaid.live](https://mermaid.live) to render it.

---

## Diagram 1 — Full System Overview

```mermaid
flowchart TB

  %% ─────────────────────────────────────────
  %% LAYER 1: HUMAN GOVERNANCE
  %% ─────────────────────────────────────────
  subgraph HUMAN["👤 Layer 1 — Human Governance"]
    direction LR
    H1["Access Manager\n─────────────\nDefines policies\nApproves JIT tokens\nRotates secrets"]
    H2["Code Reviewer\n─────────────\nHigh-risk PRs only\nRisk score ≥ 71"]
    H3["Incident Responder\n─────────────\nAudit log alerts\nBreach response"]
  end

  %% ─────────────────────────────────────────
  %% LAYER 2: ORCHESTRATION
  %% ─────────────────────────────────────────
  subgraph ORCH["🧠 Layer 2 — Orchestration (Signed Manifests)"]
    direction LR
    TO["Task Orchestrator\n─────────────\nSigned task manifests\nRoutes to dev agents\nTracks task state"]
    QO["QA Orchestrator\n─────────────\nSigned artifact routing\nRoutes to QA agents\nEnforces gate results"]
  end

  HUMAN -->|"policy + approvals"| ORCH

  %% ─────────────────────────────────────────
  %% LAYER 3: ZERO-TRUST GATE
  %% ─────────────────────────────────────────
  subgraph ZTG["🔐 Layer 3 — Zero-Trust Gate"]
    direction LR
    ZT1["Identity & Auth\n─────────────\nShort-lived tokens\nNo shared accounts\nRe-verify every call"]
    ZT2["Policy Engine\n─────────────\nOPA rule evaluation\nScope enforcement\nAction allow/deny"]
    ZT3["Rate & Scope Limiter\n─────────────\nCPU / memory caps\nRequest rate limits\nBlast radius control"]
  end

  ORCH -->|"verified requests only"| ZTG

  %% ─────────────────────────────────────────
  %% LAYER 4: SANDBOXED DEV AGENTS
  %% ─────────────────────────────────────────
  subgraph DOCKER_DEV["🐳 Layer 4 — Dev Agent Containers (Sandboxed)"]
    direction LR

    subgraph DA1["frontend-agent"]
      FA["Frontend Agent\n─────────────\nUI / state mgmt\nRead-only FS\nNon-root user\nNo internet\nSigned image"]
    end

    subgraph DA2["backend-agent"]
      BA["Backend Agent\n─────────────\nAPIs / services\nRead-only FS\nNon-root user\nNo internet\nSigned image"]
    end

    subgraph DA3["data-agent"]
      DA["Data Agent\n─────────────\nSchema / queries\nRead-only FS\nNon-root user\nNo internet\nSigned image"]
    end
  end

  ZTG -->|"scoped task dispatch"| DOCKER_DEV

  %% ─────────────────────────────────────────
  %% CODE OUTPUT BUS
  %% ─────────────────────────────────────────
  CB[["📦 Code Output Bus\n(message queue · signed artifacts · append-only)"]]

  FA -->|"push signed artifact"| CB
  BA -->|"push signed artifact"| CB
  DA -->|"push signed artifact"| CB

  %% ─────────────────────────────────────────
  %% LAYER 5: AUTOMATED SECURITY PIPELINE
  %% ─────────────────────────────────────────
  subgraph SEC["🛡️ Layer 5 — Automated Security Pipeline"]
    direction LR
    S1["SAST\n─────────────\nSemgrep / Snyk\nCWE pattern scan\nBlocks on HIGH+"]
    S2["Secret Detection\n─────────────\nGitleaks\nTruffleHog\nZero tolerance"]
    S3["SCA\n─────────────\nDependency CVEs\nLicense check\nBlocks on HIGH+"]
    S4["DAST\n─────────────\nRuntime testing\nOWASP ZAP\nBlocks on CRITICAL"]
  end

  CB -->|"all artifacts"| SEC

  %% ─────────────────────────────────────────
  %% RISK SCORING ENGINE
  %% ─────────────────────────────────────────
  RS{{"⚖️ Risk Scoring Engine\nfile path + change size\n+ scan findings\n+ coverage delta"}}

  SEC -->|"scan results"| RS

  RS -->|"score 0–30 LOW\nauto-merge"| AUTOMERGE(["✅ Auto Merge"])
  RS -->|"score 31–70 MEDIUM\nQA agent review"| QA_LAYER
  RS -->|"score 71–100 HIGH\nhuman required"| H2

  %% ─────────────────────────────────────────
  %% LAYER 6: QA AGENTS
  %% ─────────────────────────────────────────
  subgraph QA_LAYER["🔬 Layer 6 — QA Agent Containers (Sandboxed)"]
    direction LR

    subgraph QA1["review-agent"]
      RA["Code Review Agent\n─────────────\nStatic analysis\nSecurity patterns\nStyle enforcement\nRisk summary"]
    end

    subgraph QA2["test-agent"]
      TA["Test Runner Agent\n─────────────\nUnit + integration\nCoverage gating\nRegression checks\nTest report"]
    end
  end

  QO -->|"route to QA"| QA_LAYER
  RA -->|"review feedback"| TO
  TA -->|"test results"| TO

  %% ─────────────────────────────────────────
  %% LAYER 7: JIT ACCESS + AUDIT LOG
  %% ─────────────────────────────────────────
  subgraph TRUST["🔑 Layer 7 — JIT Access & Audit"]
    direction LR
    JIT["JIT Access Broker\n─────────────\nShort-lived tokens\nTTL: 15–60 min\nScope: task only\nAuto-expire"]
    AUDIT[("Immutable Audit Log\n─────────────\nEvery agent action\nEvery token issued\nEvery scan result\nWORM storage")]
  end

  DOCKER_DEV -->|"request scoped token"| JIT
  QA_LAYER   -->|"request scoped token"| JIT
  JIT        -->|"all events"| AUDIT
  ZTG        -->|"all decisions"| AUDIT
  SEC        -->|"all scan results"| AUDIT
  H3         -->|"reviews"| AUDIT

  %% ─────────────────────────────────────────
  %% LAYER 8: CLOUD INFRASTRUCTURE
  %% ─────────────────────────────────────────
  subgraph CLOUD["☁️ Layer 8 — Multi-Cloud Infrastructure"]
    direction LR

    subgraph POD1["Cloud A"]
      DB[("Primary Database\nPostgres / Cloud SQL\nRow-level security\nEncrypted at rest")]
    end

    subgraph POD2["Cloud B"]
      REPO[("Code & Artifacts\nGit repo · CI/CD\nObject store\nSigned commits")]
    end

    subgraph POD3["Cloud C"]
      SEC2[("Secrets & Config\nHashiCorp Vault\nEnv policies\nAccess ACLs")]
    end
  end

  JIT -->|"JIT token only\nnever direct creds"| DB
  JIT -->|"JIT token only\nnever direct creds"| REPO
  JIT -->|"JIT token only\nnever direct creds"| SEC2
  HUMAN -->|"manages policies\n& access ACLs"| CLOUD

  %% ─────────────────────────────────────────
  %% STYLES
  %% ─────────────────────────────────────────
  classDef humanNode   fill:#d1fae5,stroke:#059669,color:#065f46,font-weight:bold
  classDef orchNode    fill:#ede9fe,stroke:#7c3aed,color:#4c1d95,font-weight:bold
  classDef ztNode      fill:#fce7f3,stroke:#db2777,color:#831843
  classDef devAgent    fill:#dbeafe,stroke:#2563eb,color:#1e3a8a
  classDef qaAgent     fill:#fef3c7,stroke:#d97706,color:#78350f
  classDef secNode     fill:#fee2e2,stroke:#dc2626,color:#7f1d1d
  classDef bus         fill:#fef9c3,stroke:#ca8a04,color:#78350f,font-weight:bold
  classDef riskNode    fill:#f3f4f6,stroke:#6b7280,color:#111827,font-weight:bold
  classDef trustNode   fill:#ecfdf5,stroke:#10b981,color:#064e3b
  classDef cloudNode   fill:#f1f5f9,stroke:#64748b,color:#1e293b
  classDef autoNode    fill:#d1fae5,stroke:#059669,color:#065f46

  class H1,H2,H3 humanNode
  class TO,QO orchNode
  class ZT1,ZT2,ZT3 ztNode
  class FA,BA,DA devAgent
  class RA,TA qaAgent
  class S1,S2,S3,S4 secNode
  class CB bus
  class RS riskNode
  class JIT,AUDIT trustNode
  class DB,REPO,SEC2 cloudNode
  class AUTOMERGE autoNode
```

---

## Diagram 2 — Security Pipeline Detail (Layer 5)

```mermaid
flowchart LR

  ART(["Artifact from\nCode Bus"])

  ART --> SAST

  subgraph SEC_PIPE["Automated Security Pipeline"]
    direction TB
    SAST["SAST Scan\nSemgrep / Snyk\nCodeQL"]
    SECRET["Secret Detection\nGitleaks\nTruffleHog"]
    SCA["SCA Scan\nDep CVEs\nLicense check"]
    DAST["DAST Scan\nOWASP ZAP\nRuntime tests"]

    SAST --> SECRET --> SCA --> DAST
  end

  DAST --> GATE{All scans\npassed?}

  GATE -->|"YES"| RISK["Risk\nScoring\nEngine"]
  GATE -->|"NO — HIGH/CRITICAL"| BLOCK["🚫 Block\nNotify orchestrator\nLog incident\nAlert humans"]
  GATE -->|"SECRET found"| CRIT["🔴 Critical Block\nAlert human operators\nRotate credentials\nFull incident log"]

  RISK --> R1["Score 0–30\nLOW"]
  RISK --> R2["Score 31–70\nMEDIUM"]
  RISK --> R3["Score 71–100\nHIGH"]
  RISK --> R4["CRITICAL\noverride"]

  R1 -->|"auto-merge"| MERGE(["✅ Merge"])
  R2 -->|"QA agents"| QA(["QA Review"])
  R3 -->|"human review"| HR(["👤 Human"])
  R4 -->|"dual human review"| DHR(["👤👤 Two\nHumans"])

  classDef blockNode fill:#fee2e2,stroke:#dc2626,color:#7f1d1d,font-weight:bold
  classDef passNode  fill:#d1fae5,stroke:#059669,color:#065f46,font-weight:bold
  classDef warnNode  fill:#fef3c7,stroke:#d97706,color:#78350f
  classDef scanNode  fill:#dbeafe,stroke:#2563eb,color:#1e3a8a

  class BLOCK,CRIT blockNode
  class MERGE passNode
  class QA,HR,DHR warnNode
  class SAST,SECRET,SCA,DAST scanNode
```

---

## Diagram 3 — JIT Access Flow (Layer 7)

```mermaid
sequenceDiagram
  participant Agent
  participant ZTGate as Zero-Trust Gate
  participant JIT as JIT Access Broker
  participant Vault as Secrets Vault
  participant Cloud as Cloud Resource
  participant Audit as Immutable Audit Log

  Agent->>ZTGate: Request resource access (task manifest + identity)
  ZTGate->>ZTGate: Verify token, evaluate OPA policy
  ZTGate-->>Audit: Log: access request + policy decision

  alt Policy DENIED
    ZTGate-->>Agent: 403 Denied
    ZTGate-->>Audit: Log: denial reason + agent ID
  else Policy APPROVED
    ZTGate->>JIT: Issue scoped token request
    JIT->>Vault: Fetch narrow credential (TTL 15–60 min)
    Vault-->>JIT: Short-lived scoped credential
    JIT-->>Agent: Scoped token (task-bound, auto-expires)
    JIT-->>Audit: Log: token issued, scope, TTL, task ID

    Agent->>Cloud: Access resource (with scoped token)
    Cloud-->>Agent: Resource response
    Cloud-->>Audit: Log: resource access + data touched

    Note over Agent,Cloud: Token auto-expires after TTL
    Note over Audit: Every step is tamper-proof WORM log
  end
```

---

## Diagram 4 — Human Review Decision Flow (Layer 6)

```mermaid
flowchart TD

  PR(["PR submitted\nby agent"])

  PR --> AUTO["Automated scans\nSAST + Secret + SCA + DAST"]
  AUTO --> SCORE["Risk score\ncalculated"]

  SCORE --> L["LOW\n0–30"]
  SCORE --> M["MEDIUM\n31–70"]
  SCORE --> H["HIGH\n71–100"]
  SCORE --> C["CRITICAL"]

  L --> AM(["✅ Auto-merge\nNo human needed"])

  M --> QAR["QA Agent reviews\nTests + static analysis\nRisk summary generated"]
  QAR --> QP{QA\npassed?}
  QP -->|YES| AM
  QP -->|NO| HF["Human notified\nof specific issue"]
  HF --> FIX["Agent receives\nfeedback + retries"]

  H --> HR["👤 Human reviewer\nassigned"]
  HR --> HRD{Human\ndecision}
  HRD -->|Approve| MERGE(["✅ Merge"])
  HRD -->|Request changes| FEED["Feedback →\nOrchestrator →\nAgent retries"]
  HRD -->|Reject| REJ(["❌ Rejected\nTask re-scoped"])

  C --> TH["👤👤 Two humans\nmust approve"]
  TH --> SA["Full security\naudit triggered"]
  SA --> THP{Both\napprove?}
  THP -->|YES| MERGE
  THP -->|NO| BLOCK2(["🚫 Blocked\nIncident opened"])

  classDef autoNode fill:#d1fae5,stroke:#059669,color:#065f46,font-weight:bold
  classDef warnNode fill:#fef3c7,stroke:#d97706,color:#78350f
  classDef dangerNode fill:#fee2e2,stroke:#dc2626,color:#7f1d1d,font-weight:bold
  classDef humanNode fill:#ede9fe,stroke:#7c3aed,color:#4c1d95

  class AM,MERGE autoNode
  class QAR,HF,FIX warnNode
  class REJ,BLOCK2 dangerNode
  class HR,TH,SA humanNode
```

---

## Diagram 5 — Agent Sandbox Hardening (Layer 4)

```mermaid
flowchart LR

  subgraph HOST["Host Machine"]
    subgraph CONTAINER["Docker Container — each agent"]
      direction TB
      APP["Agent process\n(non-root uid:1000)"]
      RFS["Read-only filesystem\n/ is immutable"]
      TMP["/tmp only writable\n(tmpfs, no persist)"]
      NET["Network: internal only\nNo outbound internet\nNo cross-agent direct calls"]
      RES["Resource caps\nCPU: 0.5 cores\nMemory: 512 MB\nPID limit: 100"]
      CAP["Capabilities dropped\n--cap-drop=ALL\nno-new-privileges"]
      IMG["Signed image only\nCosign verified\nSHA256 pinned"]
    end

    CBUS["Code Output Bus\n(only egress allowed)"]
    ZTGATE["Zero-Trust Gate\n(only ingress allowed)"]
  end

  ZTGATE -->|"signed task manifest"| APP
  APP -->|"signed artifact"| CBUS
  APP -.->|"blocked"| INTERNET(["🚫 Internet"])
  APP -.->|"blocked"| OTHERAGENT(["🚫 Other agents\n(direct)"])

  classDef blockNode fill:#fee2e2,stroke:#dc2626,color:#7f1d1d
  classDef safeNode  fill:#d1fae5,stroke:#059669,color:#065f46
  classDef infoNode  fill:#dbeafe,stroke:#2563eb,color:#1e3a8a

  class INTERNET,OTHERAGENT blockNode
  class CBUS,ZTGATE safeNode
  class APP,RFS,TMP,NET,RES,CAP,IMG infoNode
```

---

## Diagram 6 — Debugging Flow (Audit Trail)

```mermaid
flowchart TD

  INC(["🚨 Incident detected\nin production"])

  INC --> AL["Query immutable\naudit log"]

  AL --> Q1["Which agent\ntriggered the action?"]
  AL --> Q2["Which task manifest\nauthorized it?"]
  AL --> Q3["Which JIT token\nwas used?"]
  AL --> Q4["Which scan results\ndid it pass?"]
  AL --> Q5["Which human\napproved it?"]

  Q1 --> TL["Full timeline\nreconstructed"]
  Q2 --> TL
  Q3 --> TL
  Q4 --> TL
  Q5 --> TL

  TL --> ROOT["Root cause\nidentified"]

  ROOT --> FIX1["Patch: agent\nprompt / config"]
  ROOT --> FIX2["Patch: scan\nrule gap"]
  ROOT --> FIX3["Patch: risk score\nthreshold"]
  ROOT --> FIX4["Revoke: token\n+ rotate creds"]

  FIX1 --> POST["Post-incident\nreview + policy update"]
  FIX2 --> POST
  FIX3 --> POST
  FIX4 --> POST

  classDef incNode  fill:#fee2e2,stroke:#dc2626,color:#7f1d1d,font-weight:bold
  classDef fixNode  fill:#d1fae5,stroke:#059669,color:#065f46
  classDef midNode  fill:#fef3c7,stroke:#d97706,color:#78350f

  class INC incNode
  class FIX1,FIX2,FIX3,FIX4,POST fixNode
  class Q1,Q2,Q3,Q4,Q5,TL,ROOT,AL midNode
```

---

## Architecture Component Reference

| Component | Layer | Purpose | Key Security Property |
|---|---|---|---|
| Human operators | 1 | Policy, approvals, incidents | Only humans can override gates |
| Task orchestrator | 2 | Routes work via signed manifests | Agents only act on verified instructions |
| QA orchestrator | 2 | Routes artifacts to QA | Artifact integrity enforced |
| Zero-trust gate | 3 | Verify every request | No standing trust, re-verify always |
| Dev agent containers | 4 | Module-scoped code generation | Sandboxed, non-root, no internet |
| QA agent containers | 4 | Testing + review | Same sandbox hardening as dev agents |
| Code output bus | — | Signed artifact transport | Append-only, tamper-evident |
| SAST | 5 | Static vulnerability scan | Blocks HIGH/CRITICAL findings |
| Secret detection | 5 | Finds leaked credentials | Zero-tolerance, pre-commit + CI |
| SCA | 5 | Dependency CVE scan | Blocks vulnerable dependencies |
| DAST | 5 | Runtime behavior test | Catches what SAST misses |
| Risk scoring engine | 5→6 | Prioritizes human attention | Humans review only what matters |
| JIT access broker | 7 | Issues short-lived tokens | TTL 15–60 min, task-scoped only |
| Immutable audit log | 7 | Records every action | WORM storage, debugging superpower |
| Cloud infrastructure | 8 | Persistent data + artifacts | Never accessed with direct agent creds |

---

## Your Unique Innovations (vs. Prior Art)

| What exists | What you add |
|---|---|
| ChatDev / MetaGPT: phase-scoped agents (design → code → test) | **Module-scoped agents** (frontend / backend / data) — maps to real team structure, enables true parallelism |
| Single orchestrator in most frameworks | **Dual orchestrators** (task + QA) — independent scaling of QA without touching dev pipeline |
| Shared Docker Compose runtime | **Per-agent isolated containers** with full sandbox hardening per agent |
| No access control model in academic frameworks | **Humans as the explicit access control plane** between ephemeral containers and persistent cloud |
| HULA (Atlassian): human-in-loop in JIRA | **Risk-scored triage** — humans only review HIGH-risk PRs; everything else flows automatically |
| No JIT model in any published multi-agent dev framework | **JIT access broker** with task-bound, auto-expiring tokens per agent action |

---

*Render each diagram at [https://mermaid.live](https://mermaid.live) — paste the code block contents.*
*Architecture version: 2.0 — May 2026*
