---
id: repo-easyway-portal
title: 'Repo: easyway-portal'
summary: Scheda operativa del repository easyway-portal — il data portal Node.js/Express.
status: active
owner: team-platform
created: '2026-03-05'
updated: '2026-03-05'
tags: [easyway-portal, process/repos, circle-3, domain/portal, layer/application, audience/dev, privacy/internal, language/it]
entities: []
llm:
  include: true
  pii: none
  chunk_hint: 300
type: reference
---

# easyway-portal

Il data portal. Applicazione web business-specific.

## Anagrafica

| Campo | Valore |
|-------|--------|
| **ADO repo** | `easyway-portal` (GUID: `e29d5c59-28d0-4815-8f62-6f5c1465855c`) |
| **GitHub** | Private — non mirrorato |
| **Cerchio** | 3 (Private) |
| **Linguaggio** | Node.js / Express / EJS |
| **Porta** | 3000 (container) |
| **CI/CD** | ADO Pipeline — PreChecks, BuildAndTest, GitHubMirror |
| **Branch protetti** | `main`, `develop` — merge no-ff, min 1 reviewer |
| **Deploy** | `~/easyway-portal` su server, `git fetch + reset --hard` |

## Struttura locale

```
C:\old\easyway\portal\
```

## Dipendenze

- **easyway-infra**: docker-compose, Caddyfile, deploy.sh
- **easyway-wiki**: knowledge base per RAG

## Note operative

- npm audit: 0 vulnerabilita (S70)
- Branch cleanup: da 100+151 a 2+9 (S68)
- Pipeline semplificata: 779 a 230 righe (S58)
