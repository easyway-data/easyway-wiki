---
title: "Session 82 — The Cartographer"
date: 2026-03-05
category: infrastructure
session: S82
tags: [mcp, rag, secrets, documentation, cursorrules, polyrepo, governance]
---

# Session 82 — The Cartographer

> "Non basta costruire. Bisogna lasciare la mappa."

La sessione 82 e' quella in cui la piattaforma ha smesso di essere sapere implicito e ha iniziato a documentare se stessa per chi verra' dopo — umano o IA.

## Il contesto

Dopo 81 sessioni, il sistema funzionava ma la conoscenza viveva frammentata: in MEMORY.md, in .cursorrules (2237 righe), nella testa degli agenti, nei commit message. Un'IA nuova avrebbe dovuto indovinare. Non e' accettabile.

## Le tre costruzioni

**ado_rag_resolve** — l'undicesimo tool MCP. Ora un agente puo' chiedere "come si fa il deploy?" e ricevere chunk dalla wiki via Qdrant. Shell-out a `qdrant.sh` per riusare l'infrastruttura SSH esistente. 5 test, 23 totali verdi.

**Secrets expiry alerting** — `secrets-registry.json` con metadata PAT (nomi, date, no valori). n8n workflow v2.0 con branch parallelo: se un PAT scade entro 14 giorni, crea automaticamente un Task ADO. Il sistema si prende cura di se stesso.

**_common.sh env overlays** — la funzione `_load_env()` era copiata in 3 connettori. Ora vive in `_common.sh` con auto-detect (Windows/Linux/server) e overlay `CONN_ENV=server`. DRY applicato.

## La mappa

Ma il vero prodotto della sessione e' la documentazione:

- **polyrepo-git-workflow.md** (~500 righe): mappa ASCII dell'architettura, 11 sezioni, 10 ricette copia-incolla, 7 troubleshooting. Un'IA che legge questa guida sa esattamente cosa fare.
- **.cursorrules v3.0 "Index Edition"**: da 2237 a ~230 righe. Non contiene piu' le regole — contiene l'indice. Punta alla wiki, ai tool MCP, ai connettori, ai workflow n8n. La conoscenza vive dove puo' essere aggiornata.

## La lezione

*"Hai messo le logiche di ADO prima di modificare?"* — la domanda dell'utente che ha guidato l'intera revisione. Prima i prerequisiti (Regola del Palumbo, verifica WI, verifica PR pendenti), poi i comandi. Prima capire, poi agire. La guida rispetta quest'ordine.

## Per chi viene dopo

Tre punti di orientamento:
1. **`.cursorrules`** — caricato al boot, indice rapido, dice dove cercare
2. **`wiki/guides/polyrepo-git-workflow.md`** — la guida operativa completa
3. **`platform-operational-memory.md`** — la storia, sessione per sessione, per capire il perche'

E quando hai un dubbio: **chiedi a GEDI** (manifest in `agents/agent_gedi/manifest.json`, casebook in `GEDI_CASEBOOK.md`). Il grillo parlante non blocca mai, ma illumina.
