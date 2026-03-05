---
id: repo-hale-bopp-argos
title: 'Repo: hale-bopp-argos'
summary: Scheda operativa del repository hale-bopp-argos — policy gating engine rule-based.
status: active
owner: team-platform
created: '2026-03-05'
updated: '2026-03-05'
tags: [hale-bopp-argos, repos, circle-1, domain/policy, layer/engine, audience/dev, privacy/public, language/it]
entities: []
llm:
  include: true
  pii: none
  chunk_hint: 300
type: reference
---

# hale-bopp-argos

Policy gating engine rule-based. Valutazione regole per data quality, schema compliance, deployment gates.

## Anagrafica

| Campo | Valore |
|-------|--------|
| **GitHub** | [hale-bopp-data/hale-bopp-argos](https://github.com/hale-bopp-data/hale-bopp-argos) |
| **Cerchio** | 1 (Open source, Apache 2.0) |
| **Linguaggio** | Python |
| **Porta** | 8200 |
| **CI/CD** | GitHub Actions |
| **Test** | 14 test |
| **Systemd** | `hale-bopp-argos.service` (enabled, auto-restart, bind 127.0.0.1) |

## Dipendenze

- Parte del trio HALE-BOPP: DB + ETL + ARGOS

## ADO

- Epic #30 — completato S73

## Note

- Candidato conversione da servizio HTTP a libreria Python (parcheggiato, servizio funziona)
