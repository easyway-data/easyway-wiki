# Product Map

Everything that can be extracted from EasyWay and HALE-BOPP as standalone, reusable products.
Each product has a maturity assessment based on the [MVP Maturity Checklist](standards/mvp-maturity-checklist.md).

## Why So Many Repositories?

We are entering the age of **multi-agent collaboration**. Soon, multiple AIs will work on the same codebase simultaneously — one handling schema governance, another managing PRs, another scanning for secrets, another writing tests. Each needs a clear boundary, a clear interface, a clear contract.

That's why we split into many repositories. Not because we like complexity, but because:

1. **Each repo = one responsibility.** An AI agent working on Iron Dome cannot accidentally break the ETL engine.
2. **Each repo = one release cycle.** hale-bopp-db can reach v1.0 while easyway-ado is still at v0.1.
3. **Each repo = one maturity level.** Some are ready for the world, others are still private. The Three Circles model works per-repo, not per-org.
4. **Each repo = one potential product.** What started as internal tooling becomes a standalone framework that anyone can adopt.

The polyrepo architecture is not a cost — it's the factory floor where products are born.

---

## The Full Picture

```
                    ┌─────────────────────────────────────────────┐
                    │             OPEN SOURCE (Circle 1)           │
                    │                                              │
                    │  HALE-BOPP           Extracted Frameworks    │
                    │  ┌──────────┐       ┌─────────────────────┐ │
                    │  │ hale-db  │       │ GEDI (AI ethics)    │ │
                    │  │ hale-etl │       │ Iron Dome (secrets) │ │
                    │  │ hale-arg │       │ ewctl (governance)  │ │
                    │  └──────────┘       │ Maturity Checklist  │ │
                    │                     │ Agentic Playbook    │ │
                    │                     └─────────────────────┘ │
                    ├─────────────────────────────────────────────┤
                    │          SOURCE-AVAILABLE (Circle 2)         │
                    │                                              │
                    │  ┌─────────────────────────────────────┐    │
                    │  │ easyway-ado (ADO SDK + MCP Server)  │    │
                    │  │ easyway-wiki (knowledge base)       │    │
                    │  │ easyway-agents (agent platform)     │    │
                    │  └─────────────────────────────────────┘    │
                    ├─────────────────────────────────────────────┤
                    │              PRIVATE (Circle 3)              │
                    │                                              │
                    │  ┌─────────────────────────────────────┐    │
                    │  │ easyway-portal (the data portal)    │    │
                    │  │ easyway-infra (docker, deploy)      │    │
                    │  └─────────────────────────────────────┘    │
                    └─────────────────────────────────────────────┘
```

---

## Circle 1 — Open Source Products

### 1. HALE-BOPP DB — Schema Governance Engine

| | |
|---|---|
| **What** | CLI + library for PostgreSQL schema governance: diff, deploy, drift detection |
| **Language** | Python |
| **Repo** | [hale-bopp-data/hale-bopp-db](https://github.com/hale-bopp-data/hale-bopp-db) |
| **License** | Apache 2.0 |
| **Status** | Running on server, 17 tests, systemd service |
| **Maturity** | L1: 3/4 — L2: 3/4 — L3: 2/3 — L4: 1/4 |

**To reach full maturity:**
- [ ] Test on a PostgreSQL instance that isn't ours (L1.2)
- [ ] Handle bad credentials gracefully (L2.2)
- [ ] End-to-end demo: schema diff + deploy + verify (L3.3)
- [ ] CHANGELOG with semver (L4.1)
- [ ] `pip install hale-bopp-db` (L4.3)
- [ ] 3 documented use cases with evidence (L4.4)

---

### 2. HALE-BOPP ETL — Lightweight ETL Runner

| | |
|---|---|
| **What** | Event-driven ETL with file watcher + webhook trigger, YAML pipeline definitions |
| **Language** | Python |
| **Repo** | [hale-bopp-data/hale-bopp-etl](https://github.com/hale-bopp-data/hale-bopp-etl) |
| **License** | Apache 2.0 |
| **Status** | Running on server, 30 tests, webhook + watcher services |
| **Maturity** | L1: 3/4 — L2: 3/4 — L3: 2/3 — L4: 1/4 |

**To reach full maturity:**
- [ ] Test on a different server (L1.2)
- [ ] Graceful error on malformed YAML pipeline (L2.2)
- [ ] Demo: file drop → transform → load → verify (L3.3)
- [ ] CHANGELOG (L4.1)
- [ ] `pip install hale-bopp-etl` (L4.3)

---

### 3. HALE-BOPP Argos — Policy Gating Engine

| | |
|---|---|
| **What** | Rule-based policy evaluation for data quality, schema compliance, and deployment gates |
| **Language** | Python |
| **Repo** | [hale-bopp-data/hale-bopp-argos](https://github.com/hale-bopp-data/hale-bopp-argos) |
| **License** | Apache 2.0 |
| **Status** | Running on server, 14 tests, systemd service |
| **Maturity** | L1: 3/4 — L2: 3/4 — L3: 1/3 — L4: 1/4 |

**To reach full maturity:**
- [ ] Test with custom policy rules from another project (L1.2)
- [ ] Clear error on invalid policy definition (L2.2)
- [ ] Demo: define policy → evaluate → gate pass/fail (L3.3)
- [ ] CHANGELOG (L4.1)
- [ ] `pip install hale-bopp-argos` (L4.3)

---

### 4. GEDI — Ethical Decision Framework for AI Agents

| | |
|---|---|
| **What** | A lightweight, embeddable decision framework that helps AI agents make principled choices. 16 principles, OODA loop, never blocks — only illuminates. The wise grandfather, the cricket on your shoulder. |
| **Language** | Language-agnostic (JSON manifest + methodology) |
| **Current location** | `easyway-agents/agents/agent_gedi/manifest.json` |
| **Target repo** | `easyway-data/gedi` (new, GitHub) |
| **License** | Apache 2.0 |
| **Status** | 31 documented cases in GEDI Casebook, used in every session |
| **Maturity** | L1: 2/4 — L2: 1/4 — L3: 3/3 — L4: 2/4 |

**What to extract:**
- `manifest.json` — the 16 principles
- `GEDI_CASEBOOK.md` — 31 real decision cases
- Integration guide: how to embed GEDI in any AI agent workflow
- Prompt templates for OODA loop consultation

**To reach full maturity:**
- [ ] Test in a non-EasyWay project (L1.2)
- [ ] No EasyWay-specific references in core (L1.3)
- [ ] Automated validation of principle application (L2.1)
- [ ] CHANGELOG (L4.1)
- [ ] npm/pip package or standalone repo (L4.3)

---

### 5. Iron Dome — Pre-Commit Secrets Scanner

| | |
|---|---|
| **What** | Rule-based secrets scanner that runs as a pre-commit gate. Catches PATs, API keys, connection strings before they reach the repo. |
| **Language** | PowerShell |
| **Current location** | `easyway-agents/scripts/pwsh/modules/ewctl/ewctl.secrets-scan.psm1` |
| **Target repo** | `easyway-data/iron-dome` (new, GitHub) |
| **License** | Apache 2.0 |
| **Status** | Production — runs on every `ewctl commit` |
| **Maturity** | L1: 2/4 — L2: 2/4 — L3: 2/3 — L4: 0/4 |

**What to extract:**
- `ewctl.secrets-scan.psm1` — core scanning module
- Pattern library (PATs, API keys, connection strings, private keys)
- Git hook integration script
- Configuration: custom patterns, exclude paths

**To reach full maturity:**
- [ ] Works outside ewctl (standalone mode) (L1.1)
- [ ] Test on a non-EasyWay repo (L1.2)
- [ ] No EasyWay patterns hardcoded (L1.3)
- [ ] Tests with known-bad fixtures (L2.1)
- [ ] Cross-platform (PowerShell Core on Linux) (L2.4)
- [ ] CHANGELOG + semver (L4.1)
- [ ] Install as git hook with one command (L4.3)

---

### 6. MVP Maturity Checklist — Universal Readiness Framework

| | |
|---|---|
| **What** | A 4-level, 15-question framework for evaluating whether any MVP is ready for progressively wider audiences. Born from real failures. |
| **Language** | Methodology (Markdown) |
| **Current location** | `easyway-wiki/standards/mvp-maturity-checklist.md` |
| **Target** | Include in GEDI repo or standalone guide |
| **License** | CC-BY-4.0 |
| **Status** | Written Session 75, applied to 4 projects |
| **Maturity** | L1: 4/4 — L2: N/A — L3: 3/3 — L4: 2/4 |

**Already mature** — it's a document, not code. Needs a home (GEDI repo or standalone).

---

### 7. The Agentic Playbook — How to Build Software with AI Agents

| | |
|---|---|
| **What** | A practical guide extracted from 75+ sessions: how to set up agentic SDLC, manage context across sessions, handle PR governance, maintain knowledge lifecycle. Not theory — a playbook with evidence. |
| **Language** | Methodology (Markdown + examples) |
| **Current location** | Scattered across wiki (guides/, standards/, chronicles/) |
| **Target repo** | `easyway-data/agentic-playbook` (new, GitHub) |
| **License** | CC-BY-4.0 |
| **Status** | Content exists, needs extraction and curation |
| **Maturity** | L1: 1/4 — L2: N/A — L3: 3/3 — L4: 1/4 |

**What to extract:**
- Session continuity pattern (MEMORY.md + wiki + handoff)
- Agentic SDLC flow (PBI → branch → PR → review → merge)
- PAT identity governance (service accounts, least privilege)
- Connection Registry pattern (Electrical Socket)
- Brain vs Muscles architecture (AI + deterministic separation)
- Chronicles approach (narrative documentation)
- GEDI integration (ethical decision framework)
- Lessons learned (curated from 75+ sessions)

**To reach full maturity:**
- [ ] Rewrite without EasyWay-specific references (L1.3)
- [ ] Test: someone follows the playbook on a greenfield project (L1.1)
- [ ] 3 complete walkthroughs with evidence (L4.4)

---

### 8. ewctl — The Governance CLI ("The Sacred Interface")

| | |
|---|---|
| **What** | A polyglot governance CLI with a universal interface: `check`, `plan`, `fix`. Pluggable modules for secrets scanning, KB consistency, DB drift, VCS operations. The "traffic cop" that enforces rules across repos, teams, and languages. |
| **Language** | PowerShell (designed for Python/Rust ports) |
| **Current location** | `easyway-agents/scripts/pwsh/ewctl.ps1` + `modules/ewctl/` |
| **Target repo** | `easyway-data/ewctl` (new, GitHub) |
| **License** | Apache 2.0 |
| **Status** | Production — used in every commit, every session |
| **Maturity** | L1: 2/4 — L2: 2/4 — L3: 3/3 — L4: 0/4 |
| **Pitch** | "Stop writing messy bash scripts for CI/CD. Use ewctl and pre-built modules to validate Wiki, DevOps, and Database with one cross-platform command." |

**What to extract:**
- `ewctl.ps1` — Sacred Kernel (check, plan, fix, commit, describe)
- Module system (ewctl.*.psm1 auto-discovery)
- `ewctl.secrets-scan.psm1` → becomes Iron Dome standalone
- `ewctl.governance.psm1` → KB consistency checker
- `ewctl.vcs.psm1` → VCS abstraction (Electrical Socket Pattern)

**To reach full maturity:**
- [ ] Works without EasyWay-specific modules (L1.3)
- [ ] Module template for users to create their own (L1.1)
- [ ] Tests for module discovery + isolation (L2.1)
- [ ] Cross-platform CI (L2.4)
- [ ] CHANGELOG + semver (L4.1)

---

## Circle 2 — Source-Available Products

### 8. easyway-ado — ADO SDK + MCP Server

| | |
|---|---|
| **What** | TypeScript SDK, CLI, and MCP Server for Azure DevOps. PAT routing, work items, PRs, briefing, governance — all consumable by AI agents via MCP. |
| **Language** | TypeScript + bash + PowerShell |
| **Repo** | ADO: `easyway-ado` → GitHub: `easyway-data/easyway-ado` (when mature) |
| **License** | Source-available (TBD: Apache 2.0 when Circle 1) |
| **Status** | Phase 0 complete (bootstrap), Phase 1-3 pending |
| **Maturity** | L1: 1/4 — L2: 0/4 — L3: 0/3 — L4: 0/4 |

**Roadmap to Circle 1:**
- Phase 1: Copy scripts (no breaking changes)
- Phase 2: TypeScript core (pat-router, ado-client, modules)
- Phase 3: MCP server (20 tools, 3 resources)
- Phase 4: Backward compatibility bridge
- Phase 5: Cleanup + GitHub mirror
- When L1-L3 all pass → promote to Circle 1

---

### 9. easyway-agents — Agent Platform

| | |
|---|---|
| **What** | The agent runtime: L2/L3 agents, skills registry, ewctl CLI, knowledge API, parallel execution, GEDI integration. |
| **Language** | PowerShell + Python + bash |
| **Repo** | ADO + GitHub: `easyway-data/easyway-agents` |
| **License** | Source-available |
| **Status** | Production — 35 skills, 10+ agents, Iron Dome |
| **Maturity** | L1: 2/4 — L2: 2/4 — L3: 3/3 — L4: 1/4 |

**Path forward:** As individual components are extracted (GEDI, Iron Dome, easyway-ado), this repo becomes leaner. The remaining agent platform could become Circle 1 when generic enough.

---

## Circle 3 — Private (Not for Extraction)

### easyway-portal

The data portal. Business-specific, customer-facing. Stays private.

### easyway-infra

Docker Compose, deploy scripts, server config. Org-specific. Stays private.

---

## Summary: 10 Products from 1 Project

| # | Product | Type | Circle Target | Current Maturity |
|---|---------|------|:------------:|:----------------:|
| 1 | **hale-bopp-db** | Schema governance | 1 | L2 partial |
| 2 | **hale-bopp-etl** | ETL runner | 1 | L2 partial |
| 3 | **hale-bopp-argos** | Policy gating | 1 | L2 partial |
| 4 | **GEDI** | AI ethics framework | 1 | L3 (needs extraction) |
| 5 | **Iron Dome** | Secrets scanner | 1 | L2 (needs extraction) |
| 6 | **ewctl** | Governance CLI | 1 | L2 (needs extraction) |
| 7 | **Maturity Checklist** | Methodology | 1 | L4 ready |
| 8 | **Agentic Playbook** | Methodology guide | 1 | L1 (needs curation) |
| 9 | **easyway-ado** | ADO SDK + MCP | 2 → 1 | L1 partial |
| 10 | **easyway-agents** | Agent platform | 2 | L2 partial |

> We started writing a data portal. We ended up with ten products,
> a methodology, and a story worth telling.

---

## What's Next

**Short term (Sprint 1-2):**
- Complete easyway-ado Phase 1-3 (MCP server working)
- HALE-BOPP: pip packaging + CHANGELOG for all 3 engines
- Extract GEDI into standalone repo

**Medium term (Sprint 3-6):**
- Extract Iron Dome as standalone tool
- Write Agentic Playbook v1.0
- Promote easyway-ado to Circle 1
- First public article: "How we built 9 products while building 1"

**Long term:**
- npm publish easyway-ado
- pip publish hale-bopp-*
- GEDI adopted by other agentic projects
- Conference talk: the EasyWay story
