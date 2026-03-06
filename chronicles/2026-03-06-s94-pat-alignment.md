---
title: "Session 94 — L'Allineamento dei Chiavi e il Pattern LLM"
date: 2026-03-06
category: governance
session: S94
tags: [pat-router, ado-auth, gedi, llm-integration, closeout, antifragile]
---

# Session 94 — L'Allineamento dei Chiavi e il Pattern LLM

> "Due strumenti che fanno la stessa cosa devono farla allo stesso modo.
> Altrimenti non hai ridondanza — hai confusione."

## Il Contesto

Session 93 aveva creato `ado-auth.sh` con auto-detect server/locale e 4 PAT
specializzati. Ma `pat-router.ts` — il fratello TypeScript usato da n8n — aveva
ancora una discovery chain diversa. GEDI viene invocato: Case #38, "Allineamento
delle Chiavi".

## Atto I — Closeout S93

La sessione si apre con il completamento del closeout S93: chronicle, platform
memory, sessions-history. WI #113 viene chiuso a Done. Il fix Palumbo viene
applicato a PR #403 (linkaggio WI mancante).

## Atto II — PAT Router Alignment (GEDI Case #38)

GEDI identifica il rischio: due componenti (`ado-auth.sh` e `pat-router.ts`)
con catene di discovery diverse = comportamento imprevedibile.

Azioni:
- `pat-router.ts` allineato alla stessa catena di `ado-auth.sh`
- Rimosso `cwd/.env` dalla discovery chain — rischio leakage in repo committabili
- PR #405 su easyway-ado

## Atto III — pat-health-check.sh Antifragile

Creato un watchdog per i 4 PAT ADO:
- Verifica scadenza, scope, connectivity per ciascun PAT
- Output JSON (`--json`) per integrazione n8n futura
- Allarme proattivo prima che un PAT scada silenziosamente

## Atto IV — LLM Integration Pattern Guide

Documentata la guida per integrare LLM nella piattaforma:
- Pattern: connections.yaml + OpenRouter/DeepSeek/Anthropic
- Ordine di preferenza: OpenRouter > DeepSeek > Anthropic diretto
- Ricette per n8n, curl, Node.js

PR #404 su easyway-wiki.

## Lezioni

- `.env.local` non ha PAT ADO (rimossi S88) — tutte le operazioni ADO da locale passano via SSH
- `pat-router.ts` e `ado-auth.sh` DEVONO avere la stessa catena di discovery
- `cwd/.env` rimosso dalla chain — troppo rischioso in directory committabili
- Per linkare WI a PR via API: usare `ADO_WORKITEMS_PAT` (PATCH su WI con ArtifactLink)

## Riepilogo PR

| PR | Repo | Contenuto | Stato |
|---|---|---|---|
| #403 | easyway-n8n | .cursorrules Source of Truth | Da approvare |
| #404 | easyway-wiki | S93 closeout + LLM integration guide | Da approvare |
| #405 | easyway-ado | pat-router.ts alignment + pat-health-check.sh | Da approvare |
| #406 | easyway-agents | GEDI Case #38 | Da approvare |

**WI**: #114 (S94 tracking)
