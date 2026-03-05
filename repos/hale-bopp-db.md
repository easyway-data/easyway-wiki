---
id: repo-hale-bopp-db
title: 'Repo: hale-bopp-db'
summary: Scheda operativa del repository hale-bopp-db — schema governance engine per PostgreSQL.
status: active
owner: team-platform
created: '2026-03-05'
updated: '2026-03-05'
tags: [hale-bopp-db, repos, circle-1, domain/database, layer/engine, audience/dev, privacy/public, language/it]
entities: []
llm:
  include: true
  pii: none
  chunk_hint: 300
type: reference
---

# hale-bopp-db

Schema governance engine per PostgreSQL. CLI `halebopp` per diff, deploy, drift detection.

## Anagrafica

| Campo | Valore |
|-------|--------|
| **GitHub** | [hale-bopp-data/hale-bopp-db](https://github.com/hale-bopp-data/hale-bopp-db) |
| **Cerchio** | 1 (Open source, Apache 2.0) |
| **Linguaggio** | Python |
| **Porta** | 8100 |
| **CI/CD** | GitHub Actions |
| **Test** | 17 test |
| **Systemd** | `hale-bopp-db.service` (enabled, auto-restart, bind 127.0.0.1) |

## Struttura locale

Non presente in locale — solo su server e GitHub.

## Dipendenze

- PostgreSQL (target database)
- Parte del trio HALE-BOPP: DB + ETL + ARGOS

## ADO

- Epic #30, PBI #34 (PostgreSQL integration test), PBI #36 (docs quickstart)
- `pip install hale-bopp-db` completato S78 (PBI #35 Done)

## Connettore

- `agents/scripts/connections/halebopp.sh` — healthcheck, diff, snapshot via API
