---
title: "Session 78 — The Installable Engine"
date: 2026-03-05
category: milestone
session: S78
tags: [hale-bopp, packaging, pypi, connector, crlf]
---

# Session 78 — The Installable Engine

> "Un motore che non si installa non e' un prodotto."

La sessione inizia con un obiettivo chiaro: rendere hale-bopp-db un pacchetto Python installabile via pip. Sembra semplice — rinominare una directory, aggiornare qualche import. Ma come sempre, i dettagli contano.

## Il Rename

La directory `app/` diventa `hale_bopp_db/`. Sedici file di import da aggiornare, uno per uno. Il `pyproject.toml` prende forma con metadata PyPI completi: autori, classificatori, URL, licenza SPDX. Un `__version__ = "0.1.0"` nell'`__init__.py` segna il punto di partenza. Il Dockerfile si adegua. La CI impara a fare `pip install -e .[dev,api]`. Il README guadagna una sezione Installation.

Diciassette test su diciassette passano. Il wheel si costruisce pulito.

## Il Connettore

Con hale-bopp-db che gira come servizio systemd (dalla Session 73), mancava un modo standard per interrogarlo dalla Connection Registry. Nasce `halebopp.sh` — un connettore unico per tutti e tre i motori (db, etl, argos) con quattro comandi: healthcheck, health, diff, snapshot. Si registra in `connections.yaml` e `healthcheck-all.sh` lo include nel giro di ronda.

## Il Bug Invisibile

Il momento piu istruttivo della sessione: `.env.local` ha terminazioni CRLF (Windows). La funzione `_load_env` in `github.sh` e `qdrant.sh` faceva `source` del file, e il `\r` finiva nei valori delle variabili. Risultato: il PAT GitHub diventava `ghp_xxx\r`, e le API rispondevano 401 Unauthorized.

Un carattere invisibile che rompe l'autenticazione. La fix e' banale — `tr -d '\r\n'` — ma la lezione e' profonda: quando fai bridge tra Windows e Linux, i line endings ti mordono sempre.

## Tre PR Aperte

La sessione si chiude con tre PR pronte per review: GitHub #1 per il pip packaging, ADO #335 per il connettore e il CRLF fix, ADO #336 per la documentazione wiki. Tre repository, tre canali, un unico filo logico.

Nel backlog entrano due idee nuove: MongoDB come catalogo agenti, e `_common.sh` per consolidare le funzioni condivise tra connettori. Idee che aspetteranno il loro turno nella lista dei desideri.
