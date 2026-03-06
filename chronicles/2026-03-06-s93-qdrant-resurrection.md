---
title: "Session 93 — La Resurrezione di Qdrant e il Mazzo di Chiavi v2"
date: 2026-03-06
category: infrastructure
session: S93
tags: [qdrant, docker, naming, pat-routing, ado-auth, cursorrules, n8n]
---

# Session 93 — La Resurrezione di Qdrant e il Mazzo di Chiavi v2

> "Quando il cervello della piattaforma smette di rispondere, non si panica.
> Si riavvia, si rinomina, e si scrive perche' e' successo."

## Il Contesto

Qdrant — la memoria RAG di EasyWay — era crashato. Il container `qdrant` non rispondeva
e il servizio di ricerca semantica era fuori gioco. La sessione si apre con un triage infrastrutturale.

## Atto I — Qdrant v1.16.2 + Naming Convention

Il container viene riportato online con Qdrant v1.16.2, e il naming Docker
viene allineato alla convenzione stabilita in S92:

- **Service name** = DNS interno (non si tocca)
- **container_name** = etichetta operativa (`easyway-memory` per Qdrant)

Il docker-compose viene aggiornato, la PR #400 mergiata e deployata.
E' anche il primo deploy che usa lo step 0 di Testudo: "crea PR prima di tentare il deploy".

## Atto II — ado-auth.sh Auto-detect

Lo script `ado-auth.sh` viene potenziato con auto-detect dell'ambiente:
se gira sul server OCI, legge i PAT da `/opt/easyway/.env.secrets`;
se gira in locale, da `$HOME/.env.local`. Un solo script, due ambienti.

PR #402, mergiata.

## Atto III — Container Inventory e PAT Routing Guide

Creati due artefatti di governance:
- **container-inventory.md**: censimento di tutti i container Docker con service name, container_name, porte, network
- **yaml-docker-standard v1.1.0**: aggiornamento della guida naming Docker
- **PAT routing guide**: documentazione dei 4 PAT ADO e del routing tramite `ado-auth.sh`

PR #401, mergiata.

## Atto IV — easyway-n8n .cursorrules

Il nono repo (easyway-n8n, creato in S86) riceve il suo `.cursorrules` con la sezione
Source of Truth che identifica ADO come remote primario. PR #403 creata, in attesa di merge.

## Lezioni

1. **ado-auth.sh e' il SOLO modo per ottenere PAT** — MAI `$ADO_PAT` inline
2. **Docker naming**: service name = DNS (non toccare), container_name = etichetta operativa
3. **Testudo blocca checkout non-main sul server** — creare PR PRIMA di tentare deploy

## Numeri

| # | Azione | PR | Stato |
|---|--------|-----|-------|
| 1 | Qdrant v1.16.2 + naming easyway-memory | #400 | Merged + deployed |
| 2 | Testudo step 0 "crea PR" | #400 | Merged |
| 3 | ado-auth.sh auto-detect env | #402 | Merged |
| 4 | container-inventory + yaml-docker-standard v1.1.0 + PAT routing guide | #401 | Merged |
| 5 | easyway-n8n .cursorrules | #403 | Da approvare |
