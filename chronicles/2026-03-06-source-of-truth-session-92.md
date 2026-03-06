---
title: "Session 92 — La Source of Truth e la Presa con Due Etichette"
date: 2026-03-06
category: governance
session: S92
tags: [domain/ado, source-of-truth, testudo, docker-naming, domain/gedi, cursorrules]
---

# Session 92 — La Source of Truth e la Presa con Due Etichette

> "Una presa con due etichette funziona — basta che tutti sappiano quale leggere.
> Ma una presa con due voltaggio diversi fa saltare il circuito."
> — GEDI Case #37

## Il Contesto

Un agente esterno (non Claude) aveva tentato di creare una PR su GitHub invece che su ADO.
Motivo: nessun `.cursorrules` diceva esplicitamente dove fosse il remote primario.
Le informazioni c'erano, sparse — ma mancava la dichiarazione inequivocabile in cima.

## La Regola Source of Truth

Aggiunta in tutti e 5 i `.cursorrules` (portal, wiki, agents, infra, ado):

> **Il remote primario e Azure DevOps** (`dev.azure.com/EasyWayData`).
> PR, branch, CI/CD: TUTTO su ADO.
> **GitHub e SOLO un mirror read-only.**

Posizione: sezione 0 (Non-Negoziabili) nel portal, sezione "Source of Truth" negli altri.
Un agente che apre qualsiasi repo EasyWay ora legge subito dove deve andare.

## La Pagina Prodotto Testudo

Session 90-91 avevano creato il sistema e la guida operativa.
Mancava la **pagina prodotto** — l'overview architetturale per chi vuole capire *cosa* e Testudo
e *perche* esiste, senza leggere 239 righe di ricette operative.

Creata `wiki/security/testudo-formation.md`:
- Architettura 4 badge con diagramma ASCII
- Ogni badge dettagliato (HUMAN_SEAL, PR_MERGED, TESTS_GREEN, MAIN_ONLY)
- Scenari di attacco e quale badge blocca
- Principi GEDI incarnati (G10, G11, G8, G4, G3, G16)
- Limiti noti e roadmap

Aggiornato `security/index.md`: link Testudo, use case "deploy", maturity model 5 stelle.

## La Presa con Due Etichette

Tentando `docker compose restart easyway-portal`, il comando fallisce.
Il servizio compose si chiama `frontend`, non `easyway-portal`. Il `container_name` e `easyway-portal`,
ma Docker Compose opera sui nomi servizio.

L'analisi rivela 6 servizi su 10 con naming disallineato tra compose e container.
Peggio: Qdrant ha `easyway-cortex` nel base e `easyway-memory` nel prod.

GEDI consultato (Case #37). Raccomandazione: Opzione B+ — documenta la dualita come design
intenzionale, fixa l'incoerenza Qdrant, rinomina nel backlog con impact analysis
su Caddyfile + env vars + depends_on.

## Deploy sul Server

Tutte le PR mergiate, Release PR #397 (develop->main) per il portal completata.
4 repo server aggiornati a main. Frontend rebuild OK, portal live con Hextech UI.

Qdrant (easyway-cortex) ha avuto errore di avvio — problema preesistente, non legato a S92.

## PR e WI

- **WI #112**: S92 ADO Source-of-Truth rule and Testudo product page
- **PR #392** (wiki): Testudo product page + ADO source-of-truth
- **PR #393** (portal): ADO source-of-truth + Hextech UI fixes
- **PR #394** (agents): ADO source-of-truth in .cursorrules
- **PR #395** (infra): ADO source-of-truth in .cursorrules
- **PR #396** (ado): ADO source-of-truth in .cursorrules
- **PR #397** (portal): Release develop->main

## Lezioni

1. **Source of Truth esplicita**: se non c'e una riga chiara che dice "il remote e ADO", gli agenti vanno dove vogliono
2. **Docker naming duale**: servizio compose = DNS Docker, container_name = identita operativa. Documentare la differenza
3. **GEDI Case #37**: G1 Measure Twice — documentare prima, rinominare dopo. G16 Presa Elettrica — il nome servizio E l'interfaccia
