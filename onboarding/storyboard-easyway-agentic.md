---
id: ew-storyboard-agentic-evolution
title: Storyboard evolutivo - Da knowledge base classica a continuous improvement agentico (EasyWay)
tags: [domain/onboarding, domain/devx, domain/agentic, domain/vector-db, kb, riflessione, process/retrospective, process/strategy, language/it]
summary: Racconto della crescita agentica di EasyWay, logica evolutiva e tappe per ispirare altri progetti ad abbracciare continuous improvement agentico e semantico.
status: draft
owner: team-platform
updated: '2026-01-06'
llm:
  include: true
  pii: none
  chunk_hint: 250-400
  redaction: [email, phone]
entities: []
type: guide
---

[Home](./start-here.md)

# 📖 Storyboard evolutivo: la storia “agentica” di EasyWay

Una retrospettiva “aperta” degli step fondamentali che hanno trasformato EasyWay DataPortal **da una knowledge base documentale tradizionale** a una piattaforma agentica, self-improving e semantic-driven.  
Questa storia è pensata per essere blueprint per chi vuole replicare (e migliorare!) l’esperienza in altri progetti, evitando errori frequenti e ottimizzando tempi, collaborazione e risultati.

---

## Contesto (repo)
- Obiettivi e principi: `agents/goals.json`
- Orchestrazione e gates: `scripts/ewctl.ps1` (Checklist/DB Drift/KB Consistency/WhatFirstLint)
- Ricette operative (KB): `agents/kb/recipes.jsonl`
- Osservabilita: `agents/logs/events.jsonl`
- Wiki indicizzabile (chunks): `Wiki/EasyWayData.wiki/chunks_master.jsonl` (generato da `Wiki/EasyWayData.wiki/scripts/export-chunks-jsonl.ps1`)
- Standard contesto: `Wiki/EasyWayData.wiki/onboarding/documentazione-contesto-standard.md`

---

## 1. Il punto di partenza: repo vivo, ma "manuale"

- EasyWay nasce con un repo ben organizzato (folder, agent, script, doc, template…).
- Tutto è documentato, ma la fruizione della knowledge base è “umana”: ricerca a mano, navigazione file, FAQ e how-to scritte, ma spesso disperse o ridondanti.
- La collaborazione è efficace tra pochi contributor storici, ma difficile da scalare per nuovi team/agent, onboarding faticoso.

---

## 2. Crescita → massa critica: la knowledge base esplode

- Il progetto cresce: agent, ricette, manifest, edge-case, best practice si moltiplicano,  
  la documentazione e automazione diventano indispensabili.
- Emergono gap tipici:
  - FAQ duplicate, doc non sempre linkata/crossreferenziata, ricette “nascoste”
  - Difficoltà nel troubleshooting rapido e nell’onboarding dei nuovi
- Si introduce maggiore ordine (README ovunque, naming standard, tagging, templates)  
  e si prepara a una “maturità semantica”.

---

## 3. Si riconosce il bisogno di automazione più “smart”

- Gli agent iniziano a produrre log, report, orchestrano pipeline (n8n e CI/CD).
- Nasce la domanda:  
  _Come possiamo far sì che agent e workflow non solo “eseguano”, ma anche imparino, integrino, suggeriscano fix/test/documentazione senza intervento umano continuo?_
- Si comprende che una knowledge base “classica” non basta più: serve capacità di **scoprire, ragionare e auto-migliorarsi**.

---

## 4. La svolta: knowledge base vettoriale

- Introduzione della **vector db** e della pipeline di embedding:
    - Tutta la doc/manifest/ricette/faq viene chunkata e trasformata in embedding numerici, suggerendo pattern, simili, fix, edge-case nuovi.
    - Le regole di inclusion/exclusion (yaml) rendono trasparente cosa è “runtime” e cosa resta “manuale”.
- La base vettoriale viene aggiornata automaticamente (CI-agente-n8n) a ogni modifica della repo.

---

## 5. Continuous improvement e ciclo agentico

- Da quel momento, ogni domanda — sia umana che agente (n8n, orchestrazione, LLM, copilot) — ottiene risposte intelligenti, semantiche e always-up-to-date.
- Gli agent possono:
  - Suggerire fix, creare snippet di doc, generare PR, segnalare gap/FAQ mancanti, proporre ricette edge-case, e aggiornare best practice
  - Fare refactoring auto-guidato della knowledge base (merge/split/linking)
  - Colmare mismatch tra uso reale e doc, aggiornando di continuo i punti critici
- Si instaura così un ciclo “repo <-> vector db <-> agent”: il sistema impara dai suoi stessi artefatti e dai pattern d’uso

---

## Loop operativo auto-migliorativo (WHAT-first)

### Segnali (input)
- Domande ricorrenti (utente/dev/agent), errori, edge-case, gap di documentazione.
- Esiti gate CI: Checklist/DB Drift/KB Consistency/WhatFirstLint (entrypoint `scripts/ewctl.ps1`).
- Log ed eventi di esecuzione: `agents/logs/events.jsonl` (con `decision_trace_id`).

### Artefatti (output)
- Patch a Wiki/KB/Script (PR) + ricette nuove/aggiornate in `agents/kb/recipes.jsonl`.
- Indice Wiki aggiornato: `Wiki/EasyWayData.wiki/chunks_master.jsonl`.
- Report di consistenza e decision trace ricostruibile.

### Stati e transizioni
1. Rileva segnale -> classifica (bug/doc/gap/automation).
2. Propone azione (WHAT) + impatti + comando idempotente.
3. Applica patch (HOW) con human-in-the-loop quando necessario.
4. Esegue rigenerazioni (es. chunks/inventory) e aggiorna artefatti.
5. Passa i gate e logga evento strutturato.

### Criteri di successo
- Nessuna zona grigia tra sorgenti canoniche e documentazione.
- Ogni pagina ha contesto + comandi + artefatti + gate.
- Ogni cambiamento lascia traccia ricostruibile (log + PR + output machine-readable).

---

## 6. Lezioni apprese / Cosa evitare e cosa replicare

- Non conviene vettorializzare troppo presto: prendi il tempo di strutturare bene il repo/documentazione, naming, template e tagging
- Dai priorità a cross-link, FAQ vive, ricette atomic/testate: ogni item è un “nodo” utile per la ricerca semantica futura
- Automatizza il refresh dell’index vettoriale — la pipeline fa la differenza!
- Separa sempre ambiente operativo (runtime/agent/n8n) da repo di sviluppo: ognuno alimenta/usa il vector db in modo diverso
- Coinvolgi tutta la community nella crescita: non solo agent ma anche contributor ricevono feedback e suggerimenti dal sistema

---

## 7. EasyWay oggi: un ecosistema in miglioramento autonomo

- Ogni nuovo agent, ricetta o fix si integra senza sforzo nel ciclo auto-migliorativo
- Il repo resta fonte verità per dev/contributor, vector db il motore per agent, runtime e AI/collaborazione smart
- La knowledge base è viva: onboarding più rapido, troubleshooting semplificato, agent davvero autonomi

---

## Vuoi “copiare” questa esperienza?

1. Parti dal repo: organizza, tagga, cross-linka, normalizza naming e template
2. Solo con massa critica, imposta la pipeline di vettorializzazione e scegli un vector db/promemoria inclusivo
3. Automatizza il ciclo di update tra repo e vector db, integra n8n/agent/LLM
4. Metti feedback loop e continuous improvement al centro: ogni agent/contributor è “professore” del sistema!

---

**See also / Cross-link:**
- [Knowledge base vettoriale in EasyWay](../ai/knowledge-vettoriale-easyway.md)
- [Proposte cross-link, FAQ, edge-case, automation](proposte-crosslink-faq-edgecase.md)
- [Scripting Best Practice](best-practice-scripting.md)
- [Setup sandbox/Zero Trust](setup-playground-zero-trust.md)



