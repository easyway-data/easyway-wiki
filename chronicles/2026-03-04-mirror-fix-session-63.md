---
title: "Session 63 — The Last Mirror"
date: 2026-03-04
category: fix
session: S63
tags: [domain/ci-cd, github-mirror, domain/pipeline, domain/polyrepo, process/cleanup]
---

# Session 63 — The Last Mirror

> *La Fabbrica e chiusa. Ma le finiture no.*

## Il Contesto

La Session 62 aveva chiuso La Fabbrica — polyrepo migration completata, 93 branch eliminati, backlog rinnovato. Tutto allineato. Quasi.

Un ultimo specchio era ancora storto: il GitHubMirror stage nella pipeline del portal puntava ancora a `belvisogi/EasyWayDataPortal`, il vecchio indirizzo personale, il nome di quando il progetto era un esperimento solitario.

## L'Intervento

Due righe. Solo due righe in `azure-pipelines.yml`:
- Riga 240: `belvisogi/EasyWayDataPortal` diventa `easyway-data/easyway-portal`
- Riga 251: stessa cosa, nell'URL del remote HTTPS

Un fix banale in superficie. Ma il significato e profondo: e l'ultimo riferimento al vecchio mondo. Dopo questo, ogni pipeline, ogni remote, ogni URL punta all'organizzazione `easyway-data`. Il progetto non e piu "di qualcuno" — e dell'organizzazione.

## La Build che non Parla

Il handoff segnalava una pipeline fallita (Build #20260304.10). Investigazione locale: `npm ci` OK, `tsc --noEmit` OK, `npm test` OK — 7 suite, 33 test, tutti verdi. La failure e sul server CI, ma la PAT non ha scope Build per leggere i log. Un muro silenzioso.

GEDI avrebbe detto: *"Absence of Evidence is not Evidence of Absence"* — non vedere i log non significa che il problema non esista. Ma anche: *"Known Bug Over Chaos"* — un bug noto documentato e meglio di un fix alla cieca.

## Il Consiglio del Nonno

L'utente dice: *"lascio decidere a te e GEDI"*. E GEDI, per la prima volta, non spinge avanti. Dice: fermati. Hai fatto il necessario. I task rimanenti — pipeline split, container fixes, deploy stages — sono architetturali. Meritano sessioni dedicate con i prerequisiti giusti.

A volte la saggezza non e fare di piu. E sapere quando hai fatto abbastanza bene.

## Artefatti

| Artefatto | Dettaglio |
|---|---|
| PR #282 | `fix(ci): update GitHubMirror URL` — portal, feat→develop |
| PR #283 | `docs(planning): add CI/CD backlog items` — wiki, docs→main |
| WI #44 | `[Session 63] Fix GitHubMirror URL + CI pipeline investigation` |
| GEDI Case #21 | "Closeout vs. Continue: Il Ritmo Giusto" |

## La Lezione

> *Il mirror piu importante non e quello del codice verso GitHub. E quello della sessione verso la documentazione. Se non lo scrivi, non e successo.*
