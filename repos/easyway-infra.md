---
id: repo-easyway-infra
title: 'Repo: easyway-infra'
summary: Scheda operativa del repository easyway-infra — Docker Compose, deploy scripts, Caddyfile, server config.
status: active
owner: team-platform
created: '2026-03-05'
updated: '2026-03-05'
tags: [easyway-infra, repos, circle-3, domain/infra, layer/infrastructure, audience/dev, privacy/internal, language/it]
entities: []
llm:
  include: true
  pii: none
  chunk_hint: 300
type: reference
---

# easyway-infra

Infrastruttura: Docker Compose, deploy scripts, Caddyfile, configurazione server.

## Anagrafica

| Campo | Valore |
|-------|--------|
| **ADO repo** | `easyway-infra` (GUID: `fa453541-4227-4837-bd55-8b92f656f87a`) |
| **GitHub** | [easyway-data/easyway-infra](https://github.com/easyway-data/easyway-infra) (private) |
| **Cerchio** | 3 (Private) |
| **Linguaggio** | Docker / bash / YAML |
| **CI/CD** | ADO Pipeline #318 — compose validate + shellcheck + GitHub mirror |
| **Branch protetti** | `main` — merge no-ff |

## Struttura locale

```
C:\old\easyway\infra\
```

## Componenti chiave

- **docker-compose.yml**: stack principale (portal, caddy, qdrant, gitea, n8n)
- **deploy.sh**: `~/easyway-infra/scripts/deploy.sh [env]` — Compose Coherence Gate (GEDI Case #25)
- **Caddyfile**: reverse proxy HTTPS
- **RBAC config**: `config/environments/` — modello env segregation (documentato, non implementato)

## Dipendenze

- Tutti i repo dipendono da infra per il deploy
- **easyway-net**: network Docker condiviso (name-based, non external)

## Note operative

- Server: Ubuntu ARM64 su OCI, IP `80.225.86.168`
- Port hardening: DOCKER-USER chain, pubblici solo 80/443/22
- docker compose build richiede `OPENAI_API_KEY=placeholder ANTHROPIC_API_KEY=placeholder`
