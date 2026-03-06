---
title: "Session 95 — Il Sesto Senso e le Sentinelle"
date: 2026-03-06
category: infrastructure
session: S95
tags: [gemini, connections, docker-health, pat-health, cron, watchdog, gedi]
---

# Session 95 — Il Sesto Senso e le Sentinelle

> "Una piattaforma che non sa quando si rompe, e gia' rotta.
> Una piattaforma che se ne accorge da sola, e viva."

## Il Contesto

Sessione 94 aveva lasciato quattro PR da approvare e un backlog chiaro:
Gemini API, Docker health monitoring, PAT watchdog. S95 si apre con il
closeout S94 e poi attacca il backlog con decisione.

## Atto I — Il Sesto Senso (Gemini)

Google Gemini entra nella piattaforma: Free tier, 44 modelli disponibili,
2.5 Pro e 2.5 Flash pronti all'uso. La key viene aggiunta a `.env.secrets`,
l'entry in `connections.yaml`, e la guida wiki aggiornata — Regola Connessioni
rispettata (3 posti aggiornati).

Un test end-to-end conferma: "Quale pianeta e' il piu' vicino al sole?"
Risposta: "Mercurio." Gemini parla.

## Atto II — Le Sentinelle (GEDI Case #39)

GEDI viene invocato per una decisione architetturale: come implementare
il Docker Health Daily Report? Il workflow n8n esiste ma non puo' girare
perche' il container non ha il Docker socket.

Tre opzioni: cron host (A), n8n SSH node (B), Docker socket mount (C).
GEDI raccomanda A — cron sul server host. Motivazione:
- Docker socket in container = rischio sicurezza (C eliminata)
- Credenziali SSH nella UI n8n = non riproducibile (B rimandata)
- Cron + script bash = zero dipendenze, riproducibile, coerente con pat-health-check

Due sentinelle vengono installate:

| Sentinella | Script | Cron | Output |
|---|---|---|---|
| Docker Health | `docker-health-report.sh` | Daily 07:00 | `/opt/easyway/reports/docker-health-report.json` |
| PAT Watchdog | `pat-health-check.sh` | Ogni 6h | `/opt/easyway/reports/pat-health-check.json` |

Il primo run rivela: `easyway-runner` mai avviato (Created), MinIO unhealthy.
Tutti i 4 PAT ADO + GitHub healthy.

## Lezioni

- `easyway-ado` non era clonato sul server — aggiunto per pat-health-check
- Windows line endings (`\r\n`) rompono bash sul server — sempre `sed -i "s/\r$//"` dopo upload
- WI Task in ADO usa stato "To Do" / "In Progress" / "Done" (non "Active")
- `%24Task` URL-encoded per `$Task` in curl (bash expande `$T`)

## Riepilogo PR

| PR | Repo | Contenuto | Stato |
|---|---|---|---|
| #407 | easyway-wiki | S94 closeout + Gemini connection registry | Merged |
| #408 | easyway-agents | Gemini in connections.yaml | Merged |
| #409 | easyway-ado | pat-health-check.sh | Merged |
| #410 | easyway-infra | docker-health-report.sh | Merged |

**WI**: #115
