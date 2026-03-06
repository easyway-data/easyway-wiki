---
title: "Session 66 — The Coherence Gate"
date: 2026-03-04
category: infrastructure
session: S66
tags: [domain/docker, domain/deploy, network, coherence-gate, domain/gedi, process/cleanup]
---

# Session 66 — The Coherence Gate

La rete spezzata e stata riparata. Il cancello di coerenza e stato eretto.

Session 66 e la sessione della validazione. Tre PR create nella sessione precedente (S65) per fixare il network split Docker — gia merged prima dell'apertura. Il compito: dimostrare che il fix funziona in produzione.

## Il Deploy che Valida

`deploy.sh prod` — il nuovo orchestratore multi-repo — ha eseguito per la prima volta il Compose Coherence Gate in ambiente reale:

1. **Testudo check**: 4 repo presenti, docker disponibile
2. **Coherence check**: `docker compose config` merge dei 3 overlay files, nessun `external: true`, rete `easyway-net` presente
3. **Stack up**: 10 servizi su un'unica rete `easyway-net`

Il warning "network easyway-net exists but was not created for project" e benigno — e il segnale che il pattern `name:` funziona correttamente, la rete e condivisa e non duplicata.

## La Pulizia

Dopo il deploy, quattro reti orfane giacevano inutilizzate — fantasmi di configurazioni passate:

- `easyway-prod_easyway-net` (la rete che aveva causato lo split)
- `easyway-dev_easyway-net`
- `easyway_easyway-net`
- `easywaydataportal_easyway-net` (dal vecchio nome del repo)

Caddy e stato disconnesso dalla rete orfana. Seq, ultimo container rimasto sulla vecchia rete, e stato migrato. Tutte e quattro le reti sono state rimosse.

Zero reti orfane. Zero container orfani. Un'infrastruttura pulita.

## Lezione

Il Compose Coherence Gate (GEDI Case #25) non e solo un check — e un contratto. Dice: "prima di mettere in piedi lo stack, dimostra che la configurazione e coerente." Un principio semplice che avrebbe evitato ore di debugging nella sessione precedente.

Come diceva il nonno: "Batti il ferro finche e caldo."

---

*Sessione 66 — dove il ferro battuto diventa struttura.*
