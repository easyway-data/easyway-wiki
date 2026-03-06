---
title: "Session 68 — La Grande Potatura"
date: 2026-03-04
category: infrastructure
session: 68
tags: [branch-cleanup, github-mirror, domain/pat, domain/security, credential-store]
---

# Session 68 — La Grande Potatura

> *"Un albero con 291 rami secchi non puo crescere. Prima si pota, poi si innaffia."*

## Il Problema

Quattro repository, 291 branch accumulati in mesi di sviluppo. Il portal da solo: 100 locali + 151 remoti. Una foresta di rami morti che rallentava ogni `git fetch`, confondeva ogni `git branch`, nascondeva il lavoro vero. Il debito tecnico piu visibile e fastidioso di tutti.

In parallelo: il GitHub mirror rotto da giorni per un PAT scaduto, e nessun allarme che lo segnalasse. I 4 repo non sincronizzati con GitHub. Credenziali sparse in 7 posti diversi sul server.

## La Soluzione

### Branch Cleanup — Chirurgia su 4 Repo

| Repo | Prima (locale+remoto) | Dopo | Eliminati |
|---|---|---|---|
| portal | 100 + 151 | 2 + 9 protetti | **-240** |
| agents | 8 + 11 | 1 + 2 | **-16** |
| wiki | 10 + 15 | 1 + 3 | **-21** |
| infra | 6 + 10 | 1 + 1 | **-14** |

9 branch portal protetti da ADO branch policy — eliminabili solo da admin ADO web. Il resto: potatura completa, merged verificati, nessuna perdita di lavoro.

### GitHub Mirror — Tutti e 4 i Repo in Sync

PAT `ADO-GitHub-Mirror` rinnovato (scadenza Jun 2 2026). Remote `github` configurato su server con credential store. Push forzato su tutti e 4 i repo: portal, agents, wiki, infra. Mirror funzionante.

### Credential Store — Da 7 a 3

Credenziali Git sul server consolidate: da 7 posizioni sparse a 3 gestite. `~/.git-credentials` con credential store nativo. Meno superficie d'attacco, meno confusione.

## La Lezione

GEDI Case #28: *"Un PAT scaduto ha rotto il mirror per giorni senza che nessuno se ne accorgesse."* Secrets alerting promosso da priorita Bassa ad Alta nel backlog. Token rotation calendar aggiunto: reminder 2 settimane prima della scadenza.

## I Numeri

- **291 branch eliminati** su 4 repo
- 4 GitHub mirror sincronizzati
- 1 PAT rinnovato
- 7 → 3 posizioni credenziali sul server
- 1 GEDI Case: #28 (secrets alerting)
