---
title: "Session 67 — Il Mazzo di Chiavi"
date: 2026-03-04
category: infrastructure
session: 67
tags: [pat, security, gedi, pipeline, deploy, ado-auth]
---

# Session 67 — Il Mazzo di Chiavi

> *"Un mazzo di chiavi senza etichette e come una spada senza impugnatura — prima o poi ti tagli."* — GEDI Case #27

## Il Problema

Cinque chiavi (PAT) in un file, cinque scope diversi, nessuna mappa. L'agente indovina quale usare e sbaglia. 401. Retry. Sbaglia di nuovo. L'utente dice: "chiedi a GEDI".

GEDI illumina: il problema non e tecnico, e di design. Un mazzo di chiavi senza etichette viola tre principi: Measure Twice (prima sapere, poi agire), Tangible Legacy (deve essere comprensibile tra 10 anni), Electrical Socket (l'interfaccia deve essere standard).

## La Soluzione

`ado-auth.sh` — un router che dato un'azione (`pr`, `wi`, `build`, `general`, `github`) restituisce il token giusto. Niente indovinelli. Una riga: `B64=$(ado-auth wi)`.

L'utente chiede: "quale opzione e la piu sicura?" Tre alternative: script+mappa, auto-router URL, solo mappa. GEDI analizza: l'auto-router e il meno sicuro (URL mal parsato = PAT con scope troppo ampio). Lo script+mappa vince: scelta esplicita al livello decisionale, esecuzione pulita al livello operativo.

Primo test reale: ArtifactLink WI #50 a PR #303 — successo al primo colpo.

## Il Pipeline

Il vero protagonista della sessione: il GitHub Mirror. Build su main fallisce da 5 run. Non per il codice (Build/Lint/Test tutti verdi), ma per un PAT GitHub scaduto nel variable group ADO.

Fix in due mosse:
1. `continueOnError: true` — il mirror non blocca piu la pipeline (principio Testudo: uno scudo ammaccato non fa cadere la formazione)
2. PAT aggiornato nella variable group

Release PR #303 (S66 CI fix) + PR #305 (S67 mirror fix) merged. Deploy prod completato: 10 servizi up.

## I Numeri

- 5 PR create in sessione: #303, #304, #305 (portal) — tutte merged
- 3 PR S66 verificate merged: #301 (agents), #302 (wiki)
- 1 tool creato: `ado-auth.sh`
- 1 GEDI Case: #27
- 27 casi totali nel Casebook
- Build #108: partiallySucceeded (PAT GitHub da verificare nel prossimo build)
