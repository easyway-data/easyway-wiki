---
id: repo-easyway-agents
title: 'Repo: easyway-agents'
summary: Scheda operativa del repository easyway-agents — piattaforma agenti L2/L3, skills, ewctl, GEDI, Iron Dome.
status: active
owner: team-platform
created: '2026-03-05'
updated: '2026-03-05'
tags: [easyway-agents, repos, circle-2, domain/agents, layer/platform, audience/dev, privacy/internal, language/it]
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

## Dipendenze

- **easyway-ado**: SDK per WI update (futuro Levi 2.0)
- **/opt/easyway/.env.secrets**: secrets per Import-AgentSecrets

## Prodotti estraibili

Da questo repo nascono 4 prodotti standalone: GEDI, Iron Dome, ewctl, Agentic Playbook.
