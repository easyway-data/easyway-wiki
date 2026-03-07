---
title: "Agent Context Truth & Memory Ledger"
tags: [gnosis, memory, context-truth, ledger, agenti, architettura, L3]
status: active
created: 2026-03-07
layer: "Gnosis L3 - Come ricorda e apprende"
source: "#inbox-easyway/easyway/AGENT_MEMORY_CONTEXT_TRUTH_ANALYSIS.md"
---

# Agent Context Truth & Memory Ledger

> **Layer Gnosis**: L3 - Come il sistema ricorda e apprende
> **Relazioni**: vedi [gnosis-framework.md](gnosis-framework.md) per il quadro completo

---

## 1. Lo Stato dell'Arte (Cosa abbiamo oggi)

L'Intelligenza e la Memoria degli agenti in EasyWay sono strutturate su 3 pilastri:

- **Global RAG (Wiki + Repo)**: Archiviazione centralizzata interrogata via Qdrant. Fornisce l'*Evidence* per la *Discovery*.
- **Platform Operational Memory**: `platform-operational-memory.md` come "Single Source of Truth" per le regole operative (Git flow, server config, policy).
- **Agent System Prompts / Skills**: Identity Rules e Skill isolate per agente (es. `agent_valentino/PROMPTS.md`, guardrails).

## 2. Il Problema della "Context Truth"

La **Context Truth** (la Verita del Contesto) rischia di frammentarsi o diventare stantia.
Se un agente commette un errore o una regola di business cambia, dove risiede la verita aggiornata?

- Nel `.cursorrules`?
- Nel `platform-operational-memory.md`?
- Nel payload RAG di Qdrant (che necessita di re-indexing)?
- Nel `manifest.json` dell'agente?

**Il Rischio (Allucinazione Architetturale):** Se la verita e sparpagliata, un agente che fa Discovery potrebbe pescare una regola obsoleta da un file Markdown vecchio (High Evidence, ma Low Truth), aggirando di fatto la Governance.

## 3. L'Idea del "Memory Ledger"

Il Memory Ledger e il concetto di **Apprendimento Attivo (Autofeedback)**. Invece di essere entita "stateless" che ad ogni run ripartono da zero, gli agenti dovrebbero accumulare debito informativo positivo.

### Modello Decentralizzato
1. **Ledger Locale**: Ogni agente ha un file `LOCAL_LEDGER.md` nella sua cartella.
2. **Scrittura (Feedback Loop)**: Se una PR viene rifiutata (o se l'umano corregge), l'agente annota la lezione imparata nel Ledger prima di spegnersi.
3. **Lettura (Context Injection)**: Al run successivo, prima di eseguire il WhatIf o scrivere codice, legge il suo Ledger. *Non fara piu lo stesso errore.*

### Modello Centralizzato
1. **Global Ledger via Synapse/Chronicler**: I fallimenti e i feedback vengono consolidati in un `LESSONS_LEARNED.md` globale.
2. **Sincronizzazione Qdrant**: Questo file viene indicizzato. Se Valentino impara una regola sui button React, anche l'Agente Sviluppatore ne beneficera dal RAG.

## 4. La Soluzione: "Il Portinaio" e la "Memoria a due velocita"

### A) Il "Portinaio" (Lightweight Agent Router)

Non serve un agente super-intelligente. Serve un **Portinaio**:

1. **Identikit (Carte d'Identita)**: I `manifest.json` e i `PROMPTS.md` di tutti gli agenti vengono caricati in un layer leggero.
2. **Ruolo**: Il Portinaio agisce come **PM Tecnico**. Quando arriva un'attivita, interroga le carte d'identita e risponde: *"Per questa attivita servono: Valentino (Design), Discovery (Requisiti) e PrGuardian (Review)"*.
3. **Vantaggio Antifragile**: Non ha allucinazioni sul codice perche non scrive codice. Conosce solo *chi* sa fare *cosa*.

### B) Memoria a Due Velocita (Workspace vs Global)

Per evitare che i log intasino la memoria globale (creando entropia e conflict truth):

1. **Memoria di Lavoro (Short-term, Branch/Workspace)**:
   - Attivita isolata in un branch/workspace.
   - Folder di history/memory specifica.
   - Gli agenti scrivono e leggono trial & error *solo li dentro*.

2. **Memoria Stratificata (Long-term, Globale)**:
   - I log completi restano "murati" nel workspace (e vengono archiviati alla chiusura del PBI).
   - Solo i **Principi Cardinali** estratti vengono distillati e promossi nella **Memoria Globale** (RAG di piattaforma).

## 5. Il Paradigma "Sprint Room" e la Clonazione degli Agenti

### 5.1 La "Sprint Room" (Workspace Isolato)

Quando il Portinaio decide quali agenti servono, viene istanziata una **Sprint Room** temporanea:

- Non entrano gli agenti "Main" (i padri), ma vengono generate **Copie (Cloni)** temporanee.
- Tutti gli errori, le discussioni, i tentativi falliti restano intrappolati nei log della Room.

### 5.2 Threading, Identificativi e Semafori

All'interno della Sprint Room:

- Ogni Clone e conversazione ha un **ID univoco** (es. `MSG-VALENTINO-01`).
- **Semafori di Attesa**: Il Clone-Developer non puo iniziare finche il semaforo del Clone-Valentino (Design System) non diventa VERDE.
- Routing dei segnali tramite orchestrazione della Room (multi-threading agentico orchestrato).

### 5.3 Objective Father-Evaluation (La Revisione dei "Padri")

Una volta che la Sprint Room dichiara di aver concluso:

- I **Padri** (Agenti con la vera Global Context Truth) si svegliano per vagliare il lavoro.
- **Isolamento Cognitivo (Blind Review)**: I Padri NON hanno accesso alla memoria temporanea della Room.
- **Oggettivita Pura**: Valutano ESCLUSIVAMENTE l'output finale (PR, PRD).
- **Vantaggio Antifragile**: Se i Cloni sono scesi a compromessi o hanno coperto errori reciproci (bias di co-design), il Padre -- freddo e oggettivo -- stronca il lavoro se non rispetta le Policy Master.

Solo quando i Padri approvano, l'innovazione entra in produzione e il "succo" distillato diventa memoria globale.

---

## Relazioni con gli altri Layer Gnosis

| Layer | Documento | Connessione |
|-------|-----------|-------------|
| L1 | [agentic-architecture-enterprise.md](agentic-architecture-enterprise.md) | Definisce i livelli L1-L5 che il Memory Ledger supporta |
| L2 | [idea-to-production.md](idea-to-production.md) | Il ciclo SDLC che produce le lezioni da memorizzare |
| L3 | **Questo documento** | Come il sistema ricorda e impara |
