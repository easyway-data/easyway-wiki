---
title: "Levi 2.1 e il Hierarchies Game — Quando il Doc Guardian impara a giocare a scacchi"
date: 2026-03-06
category: milestone
session: 96
tags: [levi, hierarchies, doc-guardian, runner, minio, antifragile, gedi]
---

# Levi 2.1 e il Hierarchies Game

Session 96 inizia col pragmatismo di sempre: MinIO era unhealthy, il runner dormiva in un loop infinito, e Antigravity continuava a creare PR senza Work Item. Tre problemi, tre guardrail antifragili.

## MinIO — Lo Storage Sovrano

Il container MinIO era rotto — immagine Bitnami senza `curl`, password con `!` che esplodeva nelle shell layers. La fix: immagine ufficiale, healthcheck con `mc ready`, e un container sidecar `minio-init` che crea i bucket al boot. Idempotente. Antifragile. Tre bucket pronti: documents, reports, backups.

## Runner v2.0 — Il Braccio che Agisce

Il container `easyway-runner` esisteva da sempre ma dormiva. Letteralmente: `for() { Start-Sleep 1000 }`. GEDI ha suggerito di evolverlo — non spegnerlo. Nasce il Task Server: un HTTP dispatcher Python su porta 8400 che accetta task da n8n, da curl, da chiunque. 40 agenti caricati. 5 tipi di task registrati. Il pattern: n8n decide QUANDO, il runner decide COME, MinIO decide DOVE salvare.

## Antigravity e la Regola del Palumbo — Due Volte

PR #413 e #414: Antigravity (Gemini) ha creato le PR senza linkare i Work Item. La policy ADO bloccava il merge. Seconda violazione consecutiva. La root cause? GEMINI.md era VUOTO — l'agente non aveva regole. Il portal .cursorrules aveva G1-G12 ma mancava G13. Come un estintore chiuso a chiave.

Fix antifragile: G13 aggiunta a tutti e 6 i .cursorrules, GEMINI.md compilato con governance completa, tool `pr-autolink-wi` pronto all'uso. GEDI Case #40 documentato.

## Levi 2.1 — Il Hierarchies Game

Il momento piu bello della sessione. Levi non si limita piu a dire "link rotto" — ora gioca il gioco delle gerarchie. Costruisce un indice globale di tutti i file .md nel polyrepo e quando trova un link che non risolve, cerca il file target ovunque e suggerisce il path corretto.

Ma il vero salto sono i 4 giri:

1. **Tag Health**: 501 tag, di cui 388 senza namespace. Case mismatch tra DOMAIN/ e domain/. La tassonomia ha bisogno di pulizia.
2. **Link Topology**: 2034 link risolti. Hub: index.md (326 uscenti). Authority: start-here.md (210 entranti). 149 orfani.
3. **Cross-Repo**: wiki linka agents (3), portal (8), ado (1). La mappa delle dipendenze.
4. **Clusters**: 176 file senza domain/. Il cluster piu grande e domain/docs (157). La struttura emerge dai dati.

Tutto in 477ms. Pronto per Obsidian, per n8n, per il RAG, per qualsiasi agente che vuole capire la topologia della wiki.

## La Lezione

"Un tool che esiste ma che nessuno sa di dover usare e come un estintore chiuso a chiave." E i 4 giri del Hierarchies Game insegnano che la documentazione non e una lista di file — e un grafo vivente con hub, autorita, orfani e cluster. Levi ora lo vede.
