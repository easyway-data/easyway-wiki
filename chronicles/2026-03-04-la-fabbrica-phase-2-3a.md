---
title: "La Fabbrica — Phase 2 Infra + Phase 3a Il Grande Cleanup"
date: 2026-03-04
category: milestone
session: 57
tags: [la-fabbrica, domain/polyrepo, domain/migration, domain/infrastructure, process/cleanup]
---

# La Fabbrica — Phase 2 Infra + Phase 3a Il Grande Cleanup

> "Ogni minuto che passa e' un'occasione per rivoluzionare tutto completamente" — Vanilla Sky

## Il giorno in cui il monorepo divenne il portale

Session 57 segna il punto di non ritorno della migrazione polyrepo La Fabbrica.

### Phase 2: easyway-infra
Estratti **1434 file** (224K+ righe) di infrastruttura dal monorepo:
- 11 Docker Compose stacks
- Caddyfile, azure-pipelines.yml
- Scripts infra/ops/linux/ci
- Config, release, forgejo, terraform

Repo creato su ADO e GitHub (easyway-data/easyway-infra).

### Phase 3a: Il Grande Cleanup
Rimossi **1420 file** (231K righe) dal monorepo — tutto il contenuto gia estratto:
- Wiki/ (522+ file) → easyway-wiki
- agents/, control-plane/, scripts/ (672 file) → easyway-agents
- File legacy, temp, docs obsoleti

Il monorepo ora contiene solo il cuore: `portal-api/`, `apps/portal-frontend/`, `packages/`, `tests/`, `db/`, `docs/`, `etl/`.

### GitHub
Tutti e 3 i repo pushati su GitHub org `easyway-data`:
- easyway-wiki
- easyway-agents
- easyway-infra

### Numeri della giornata
| Metrica | Valore |
|---|---|
| File estratti (infra) | 1,434 |
| File rimossi (cleanup) | 1,420 |
| Righe rimosse | 231,625 |
| PR create | 5 (#259-#263) |
| Repo su GitHub | 3 |
| PBI creati | 1 (#42) |

### GEDI dice
Consultato per Phase 3: principio **Start Small** applicato.
Cleanup sicuro oggi, CI/CD migration domani.
Il monorepo respira. La Fabbrica avanza.

### Prossimo step
Phase 3b: rename `EasyWayDataPortal` → `easyway-portal`, CI/CD migration, server multi-repo.
