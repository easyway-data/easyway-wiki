---
title: "Il Traduttore e l'Hextech"
date: 2026-03-05
category: deploy
session: S84
tags: [hextech, deploy, multi-agent, gemini, docker, nginx, golden-path]
---

# Il Traduttore e l'Hextech

> Session 84 — 5 Marzo 2026

La sessione inizia raccogliendo i frutti della S83: 4 PR approvate, script multi-agente fixati, guide Nonna Standard completate. Ma il piatto forte e l'UI Hextech che Gemini aveva quasi completato da solo.

## Il Traduttore Universale

Il primo atto e capire perche Gemini non riusciva a completare il ciclo autonomo. Tre problemi strutturali: path con backslash in bash, mount diversi tra Git Bash e WSL, e la confusione tra `Import-AgentSecrets` (infra secrets) e i PAT ADO (file diverso). La soluzione — `_detect_root()` con fallback multi-mount e la variabile `EASYWAY_ROOT` — trasforma gli script da "funzionano solo sulla mia macchina" a "funzionano ovunque". GEDI Case #33: Il Traduttore Universale.

## Il Golden Path

Nasce la documentazione del "Golden Path" — il percorso che ogni agente deve seguire per andare da zero a PR: Setup, Work+Commit, PBI, Push+PR, STOP. Scritto nella guida ADO operations con un dettaglio che non lascia spazio all'interpretazione. Se un agente non riesce a fare tutto leggendo solo la guida, la guida e incompleta.

## L'Hextech in Produzione

La palette Hextech Evolution — deep void, glassmorphism, cyan/gold/violet — prende vita nei CSS del portal. PR #353 mergiata in develop, PR #354 (Release) crea il ponte verso main. Il deploy sul server rivela pero il debito tecnico dell'infrastruttura: container creati a mano fuori compose, variabili d'ambiente sparse in file diversi, il reverse proxy nginx che restituisce Internal Server Error.

Il build funziona. Il container si ricrea. Ma il browser mostra un muro bianco con "Internal Server Error". La lezione: il codice puo essere perfetto, ma se l'infrastruttura non e allineata, non si vede niente.

## Il Backlog Strutturale

Quattro nuovi item nel backlog, tutti correlati: `.env` mancante per compose, container name conflict, nginx reverse proxy, standardizzazione compose vs docker run. Non sono bug — sono debito architetturale accumulato in 84 sessioni di sviluppo rapido. La prossima sessione dedicata all'infra dovra affrontarli tutti insieme.

## Numeri

- 2 PR mergiate (#353 feat->develop, #354 develop->main)
- 1 PBI chiuso (#103 Hextech UI)
- 4 item backlog infra aggiunti
- 1 deploy eseguito (build OK, nginx da fixare)
- 1 GEDI Case (#33 Il Traduttore Universale)
- `.cursorrules` aggiornato con sezione 3b Deploy Flow
