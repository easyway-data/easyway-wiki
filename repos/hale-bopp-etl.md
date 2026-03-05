---
id: repo-hale-bopp-etl
title: 'Repo: hale-bopp-etl'
summary: Scheda operativa del repository hale-bopp-etl — lightweight ETL runner event-driven.
status: active
owner: team-platform
created: '2026-03-05'
updated: '2026-03-05'
tags: [hale-bopp-etl, repos, circle-1, domain/etl, layer/engine, audience/dev, privacy/public, language/it]
entities: []
llm:
  include: true
  pii: none
  chunk_hint: 300
type: reference
---

# hale-bopp-etl

Lightweight ETL runner event-driven. File watcher + webhook trigger, pipeline YAML.

## Anagrafica

| Campo | Valore |
|-------|--------|
| **GitHub** | [hale-bopp-data/hale-bopp-etl](https://github.com/hale-bopp-data/hale-bopp-etl) |
| **Cerchio** | 1 (Open source, Apache 2.0) |
| **Linguaggio** | Python |
| **Porta** | 3001 (webhook) |
| **CI/CD** | GitHub Actions |
| **Test** | 30 test |
| **Systemd** | `hale-bopp-etl-webhook.service` + `hale-bopp-etl-watcher.service` |

## Dipendenze

- Parte del trio HALE-BOPP: DB + ETL + ARGOS

## ADO

- Epic #30 — v0.3.0 completato S73
