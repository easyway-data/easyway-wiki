---
title: "Il Wrapper e il Fantasma dell'ArtifactLink"
date: 2026-03-07
category: milestone
session: S101
tags: [ado-api, python, artifactlink, bug-investigation, tooling]
---

# Il Wrapper e il Fantasma dell'ArtifactLink

*Session 101, 7 marzo 2026*

La sessione inizia con un obiettivo chiaro: creare un wrapper Python per le API Azure DevOps.
Il motivo e semplice e doloroso: ogni volta che un agente costruisce una chiamata ADO in bash,
il JSON viene stritolato dal quoting della shell. Il `$` nei tipi Work Item diventa variabile,
le virgolette nidificate si mangiano a vicenda, e via SSH il disastro raddoppia.

## Il Wrapper

`ado-api.py` nasce in un'ora. Nove subcomandi, zero dipendenze esterne, stessa mappa PAT di
`ado-auth.sh`. La differenza e strutturale: in Python un dizionario diventa JSON con `json.dumps()`,
senza passare per l'inferno delle shell escape. Il tipo `Product Backlog Item` non e piu una bomba
a orologeria — e una stringa in un dict.

```
python3 ado-api.py wi create "Fix login bug" --type Bug --tags "frontend;S101"
python3 ado-api.py pr create easyway-wiki feat/branch main "titolo" --wi 123
```

La filosofia e la dualita: Python per le operazioni complesse (JSON, linking, batch), bash per
le operazioni semplici (auth, healthcheck, one-liner). Non si sostituisce `ado-auth.sh` — lo si
complementa.

## Il Fantasma

Ma il vero colpo di scena arriva con il Task #2: investigare perche `wi-link-pr` restituisce
400 "already exists" quando il Work Item mostra 0 relations.

Un fantasma. Il link c'e ma non si vede.

L'indagine rivela tre root cause che si sommano come strati di un palinsesto:

**Strato 1: La variabile fantasma.** In bash, `$expand` non e un parametro URL — e una variabile
shell. Vuota. La query `?$expand=relations` diventa `?=relations`, e ADO restituisce educatamente
il Work Item... senza relations. Zero. Il parametro e stato mangiato in silenzio.
La fix: `%24expand` (URL-encoded). In Python il problema non esiste — `$` in una f-string e
letterale.

**Strato 2: Il link bidirezionale.** Quando crei una PR con `workItemRefs`, ADO non si limita
a collegare la PR al WI. Crea silenziosamente un ArtifactLink sul WI, puntando alla PR. Pipeline
e commit (con `AB#`) fanno lo stesso. Il WI #122, che sembrava vergine, aveva in realta 45
ArtifactLinks nascosti — 16 PR, decine di commit e build.

**Strato 3: Il case sensitivity.** ADO salva gli URL con `%2f` minuscolo. Noi li costruiamo con
`%2F` maiuscolo. Il check-before-add confrontava stringhe diverse per lo stesso link. La fix:
normalizzare tutto a lowercase prima del confronto.

Tre bug che si sommano per creare un fantasma perfetto: il primo nasconde i dati, il secondo li
duplica, il terzo li rende incomparabili.

## L'Inbox

La sessione si chiude con il triage dell'inbox wiki. Tre documenti da verificare, tre esiti
diversi: uno non esiste piu (SOVEREIGN_AGENTIC_FRAMEWORK_VISION — gia rimosso), uno e valido
e resta (INFRASTRUCTURE_AUDIT — baseline operativa), uno era gia stato consolidato senza che
nessuno aggiornasse l'inbox (ANALISI_SDLC + USECASE → MASTER v2.0).

## La Lesson

Due bug che si sommano creano un fantasma: il primo nasconde i dati, il secondo li rende
incomparabili. Quando un sistema dice "esiste" ma non lo vedi, il problema e quasi sempre
nella query di lettura, non nella scrittura.

La dualita Python/bash non e un compromesso — e un'architettura. Ogni strumento ha il suo
dominio naturale. Forzare bash a fare JSON complesso e come usare un martello per avvitare.
Forzare Python a fare `source .env && exec` e ugualmente innaturale. La forza sta nel sapere
quando usare quale.
