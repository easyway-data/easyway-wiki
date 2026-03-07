---
title: "Gnosis Framework e il Wiki Health Monitor"
date: 2026-03-07
category: milestone
session: S98
tags: [gnosis, framework, marginalia, wiki-health, gedi, sovereign, architettura]
---

# Gnosis Framework e il Wiki Health Monitor

> Session 98 — 7 Marzo 2026

## Il momento

C'era un problema che cresceva da settimane: i documenti sull'architettura agentica vivevano sparsi — nell'inbox, nella wiki, nella memoria operativa, nei brainstorming. Tre layer di pensiero (chi sono gli agenti, come costruiscono, come ricordano) esistevano come isole separate. Oggi li abbiamo unificati.

## Cosa e successo

La sessione e iniziata con un handoff denso dalla S97. La lista era lunga, ma il filo conduttore era chiaro: dare forma al sistema nervoso.

**Il Framework Gnosis** e nato come documento unificante — non un'altra pagina wiki, ma il punto di ingresso che collega tutto:
- L1: chi sono gli agenti (tassonomia, livelli, governance)
- L2: come costruiscono (ciclo SDLC, gate umani, fast/full track)
- L3: come ricordano (context truth, memory ledger, sprint rooms)
- Sovereign: la visione che tira avanti il sistema

**GEDI ha parlato.** Quando abbiamo chiesto come rendere antifragili i rischi del framework, il Grillo Parlante ha acceso 7 principi e prodotto due mitigazioni strutturali: il Sovereign Maturity Gate (non costruire L4 finche L2 non e stabile e misurato) e il Wiki Pruning Cycle (la wiki che si auto-pota ogni 10 sessioni). Case #42 — uno dei piu densi.

**Il Wiki Health Monitor e operativo.** 20 query, primo snapshot: 607 documenti indicizzati, coverage 100%, recall 73%. Il battito cardiaco della wiki ora si misura. Da qui in poi, ogni sessione potra confrontare il suo snapshot con questo baseline per vedere se il sistema nervoso migliora o degrada.

**Il semaforo.** Una cosa semplice ma che mancava: `.semaphore.json` per ogni repo, verde/giallo/rosso, per evitare che due sessioni sullo stesso terminale si pestino i piedi.

## Il pezzo che resta

marginalia e stato pushato a hale-bopp-data con 48 test verdi. wi-create ora parla Epic e Feature. Ma la vera conquista e che adesso possiamo *misurare* la salute della wiki — e quindi misurare se il framework Gnosis funziona davvero o e solo carta.

Come ha detto GEDI: *"L'antifragilita non si applica solo al codice — si applica alla governance stessa."*
