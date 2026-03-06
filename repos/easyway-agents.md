---
id: repo-easyway-agents
title: 'Repo: easyway-agents'
summary: Scheda operativa del repository easyway-agents — piattaforma agenti L2/L3, skills, ewctl, GEDI, Iron Dome.
status: active
owner: team-platform
created: '2026-03-05'
updated: '2026-03-06'
tags: [easyway-agents, process/repos, circle-2, domain/agents, layer/platform, audience/dev, privacy/internal, language/it]
entities: []
llm:
  include: true
  pii: none
  chunk_hint: 300
type: reference
---

# easyway-agents

La piattaforma agenti: runner L2/L3, skills registry, ewctl, GEDI, Iron Dome, Connection Registry.

## Anagrafica

| Campo | Valore |
|-------|--------|
| **ADO repo** | `easyway-agents` (GUID: `fa068c67-b5ee-4b41-aab3-38b0166ffb27`) |
| **GitHub** | [easyway-data/easyway-agents](https://github.com/easyway-data/easyway-agents) (source-available) |
| **Cerchio** | 2 (Source-available) |
| **Linguaggio** | PowerShell / Python / bash |
| **CI/CD** | ADO Pipeline #317 — manifest validate + shellcheck + GitHub mirror |
| **Branch protetti** | `main` — merge no-ff, min 1 reviewer |

## Struttura locale

```
C:\old\easyway\agents\
```

## Componenti chiave

- **Agenti L3**: agent_review, agent_security, agent_infra, agent_levi, agent_scrummaster, agent_dba, agent_pr_gate, agent_valentino
- **Agenti L2**: agent_backend, agent_frontend, agent_sentinel
- **Skills**: `skills/registry.json` v2.15.0 — 35 skill
- **ewctl**: `scripts/pwsh/ewctl.ps1` + moduli
- **GEDI**: `agents/agent_gedi/manifest.json` (16 principi) + Casebook
- **Iron Dome**: `scripts/pwsh/modules/ewctl/ewctl.secrets-scan.psm1`
- **Connection Registry**: `scripts/connections/` — github.sh, ado.sh, server.sh, qdrant.sh, halebopp.sh
- **Task Server**: `scripts/task-server.py` — HTTP server per esecuzione task server-side (S96)

## Runner v2.0 (S96)

Il container `easyway-runner` e il braccio esecutivo server-side degli agenti.

| Aspetto | Dettaglio |
|---------|-----------|
| **Container** | `easyway-runner` (porta 8400, rete `easyway-net`) |
| **Entrypoint** | `agents/entrypoint.ps1` -> boot checks -> `scripts/task-server.py` |
| **Health** | `GET http://easyway-runner:8400/health` |
| **Agenti caricati** | 40 (11 Level 2 LLM) |

### Architettura

```
n8n (QUANDO) --> POST http://easyway-runner:8400/task --> Runner (COME) --> MinIO (DOVE salvare)
```

### Task types registrati

| Task | Script | Descrizione |
|------|--------|-------------|
| `levi-scan` | `scripts/levi-scan.py` | Levi doc guardian scan wiki |
| `gedi-validate` | `scripts/pwsh/agent-gedi.ps1` | GEDI principle validation |
| `wiki-ingest` | `scripts/ingest_wiki.js` | Re-index wiki in Qdrant |
| `health-report` | `scripts/health-report.sh` | Infra health report |
| `exec` | qualsiasi in `/app/scripts/` | Script arbitrario (sandboxed) |

### Esempio

```bash
curl -X POST http://easyway-runner:8400/task \
  -H "Content-Type: application/json" \
  -d '{"type": "levi-scan", "params": {"scope": "wiki"}}'
```

## Dipendenze

- **easyway-ado**: SDK per WI update (futuro Levi 2.0)
- **/opt/easyway/.env.secrets**: secrets per Import-AgentSecrets

## Prodotti estraibili

Da questo repo nascono 4 prodotti standalone: GEDI, Iron Dome, ewctl, Agentic Playbook.
