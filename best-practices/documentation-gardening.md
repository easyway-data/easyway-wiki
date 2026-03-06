---
tags: [domain/docs, layer/process, process/best-practices, process/governance, gardening]
updated: 2026-01-16
owner: team-platform
summary: Metodologia "Gardener Cycle" per la gestione evolutiva della tassonomia documentale. Invece di definire strutture rigide a priori, si usa un approccio bottom-up: scansione della realtà, clustering emergente e raffinamento iterativo a livelli.
status: draft
id: ew-best-practices-documentation-gardening
title: Documentation Gardening 🌱
llm:
  include: true
  pii: none
  chunk_hint: 250-400
  redaction: [email, phone]
entities: []
type: guide
---

[Home](./start-here.md) > [[domains/docs-governance|Docs]] > [[Layer - Process|Process]]

# Documentation Gardening 🌱

> **Principio**: La documentazione è un giardino, non un edificio. Non si costruisce una volta per tutte; si coltiva, si pota e si fa crescere organicamente.

## Il Problema
I tentativi di creare tassonomie documentali perfette "a tavolino" falliscono quasi sempre perché:
1.  È impossibile prevedere tutti i casi d'uso futuri.
2.  I tag diventano obsoleti rapidamente.
3.  Gli utenti usano tag "folksonomici" (liberi) che non combaciano con la struttura rigida.

## La Soluzione: Il Ciclo del Giardiniere (Gardener Cycle)

Invece di imporre la struttura, la facciamo **emergere** dai dati.

### 1. Artifacts
Il sistema si basa su due fonti di verità:

*   **Realtà (`knowledge-graph.json`)**: Generato automaticamente dallo scanner. Contiene lo stato *attuale* dei file, i loro tag grezzi e i riassunti semantici. È "ciò che c'è".
*   **Legge (`tag-master-hierarchy.json`)**: Un file curato che definisce la struttura "To-Be" accettata. È "ciò che dovrebbe essere".

### 2. Il Workflow Iterativo

Il processo avviene a "cipolla" (o a spirale), raffinando la granularità ad ogni passaggio.

#### Step 1: Scan & Discovery 🔍
L'Agent scansiona tutti i documenti e aggiorna il `knowledge-graph.json`.
Raccoglie tutti i tag "loose" (liberi) usati dai developer.

#### Step 2: Analysis & Clustering 🧠
L'Agent confronta i tag trovati con la `Master Hierarchy`.
Identifica i tag "Orfani" (Orphans) e cerca pattern semantici.

> *Esempio*: L'Agent nota che `flyway`, `migration`, `sql-script` appaiono spesso insieme.

#### Step 3: Proposal (Hierarchy Proposal) 🙋‍♂️
L'Agent propone una modifica alla Gerarchia:
> *"Propongo di raggruppare `flyway`, `migration` sotto il nodo `Tech/DB/Migrations`."*

L'essere umano (Human in the Loop) approva o corregge.

#### Step 4: Apply & Prune ✂️
Una volta approvata la nuova struttura:
1.  Il `tag-master-hierarchy.json` viene aggiornato.
2.  (Opzionale) Un agente "Fixer" passa sui file e normalizza i tag (es. sostituisce `sql-script` con `Tech/DB/Migrations`).

## Benefici
*   **Zero Debito Tecnico**: La tassonomia non invecchia mai.
*   **Low Friction**: Gli sviluppatori possono usare tag liberi all'inizio; il sistema li "normalizzerà" dopo.
*   **Searchability**: Il Knowledge Graph diventa sempre più preciso per il Retrieval RAG.

## Struttura Gerarchica (Esempio Target)

L'obiettivo è arrivare gradualmente a una struttura a 3 livelli:
1.  **Macro-Area** (Business, Technology, Ops)
2.  **Domain** (Data, API, UX, Security)
3.  **Specific** (T-SQL, React, OAUTH2)





