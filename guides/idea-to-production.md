---
title: "Idea to Production — Il Ciclo Agentico Completo"
tags: [sdlc, governance, wiki, ado, github, agenti, prd, discovery]
status: active
created: 2026-03-07
---

# Idea to Production — Il Ciclo Agentico Completo

> **Documento di riferimento esteso:** `agents/EASYWAY_AGENTIC_SDLC_MASTER.md`
> Questa guida è il riassunto operativo. Per la visione completa, architettura L1-L5 e scenari, leggere il master.

**Regola fondamentale: la Wiki è la prima fonte di verità. Tutto parte da qui.**
**Regola sovrana: gli agenti accelerano il ciclo, ma non rimuovono governance.**

---

## Il Ciclo in 5 Fasi (con Gate Umani)

```
IDEA (linguaggio naturale)
  │
  ▼
[1] WIKI BACKLOG ──────────────────── 30 secondi
  planning/initiatives-backlog.md
  │
  ▼
[2] DISCOVERY & PRD ────────────────── Gate umano ✅
  Agent_Discovery con RAG obbligatorio (wiki + repo)
  Output: PRD con Evidence + Confidence score
  │
  ▼
[3] DECOMPOSIZIONE BACKLOG ─────────── Gate umano ✅
  Agent_Backlog_Planner
  WhatIf Preview → conferma umana → Apply su ADO
  Epic → Feature → PBI → Task
  │
  ▼
[4] SPRINT EXECUTION ───────────────── Gate umano ✅ (UAT)
  Agent_Developer / Agent_DevOps
  branch → ewctl commit (Iron Dome) → PR → PrGuardian → UAT → release
  AB# obbligatorio in ogni commit GitHub
  │
  ▼
[5] WIKI + RAG UPDATE
  Guida wiki: status: planned → active
  Sezione Cosa/Perché/Come/Q&A compilata
  RAG ingest → agenti trovano la guida
```

---

## Fase 1 — IDEA → Wiki Backlog

**Trigger**: qualsiasi idea, appunto, bug futuro, miglioramento.
**Azione**: aggiungere SUBITO a `planning/initiatives-backlog.md`.

```markdown
### Potenziale / Da esplorare
- [ ] marginalia: eval integration con Qdrant reale
- [ ] n8n: ADO WI → GitHub Issue sync automatico
```

**MAI perdere un'idea** — se non la scrivi ora, scompare.

---

## Fase 2 — Discovery & PRD (Agent_Discovery)

### Chi lo fa
`Agent_Discovery` (o umano per PRD critici). **RAG obbligatorio** su wiki + repo — nessuna "tabula rasa".

### Tre pilastri dell'analisi
1. **Dependency Discovery** — quali oggetti toccare? (codice esistente, tabelle, runbook)
2. **Tech Stack Advisory** — quale tecnologia? (Sovereign Law: PowerShell/Python/Node/SQL/Qdrant)
3. **Infrastructure Blueprinting** — dove atterrare? (Static/Dynamic/Service Land)

### Template PRD

```markdown
---
title: "Nome Feature"
tags: [prd, <area>]
status: planned
evidence: [wiki/..., repo/...]
confidence: High | Medium | Low
---

## IL RAGIONAMENTO (cuore decisionale)
Perché questa soluzione e non un'altra. Con evidence.

## Cosa
Una frase chiara.

## Perché
Problema che risolve. Chi ne beneficia.

## Come
Approccio tecnico. Alternative considerate.

## Scope
- IN: cosa fa
- OUT: cosa NON fa

## Impatto tecnico
API / DB / Frontend / ETL / Governance coinvolti.

## Acceptance Criteria
- [ ] Criterio 1

## Rollback
Come tornare indietro se va male.
```

### Regola Evidence & Confidence (anti-allucinazione)
- Ogni decisione deve avere **Evidence** (fonte: wiki, repo, runbook)
- Confidence **Low** → validazione umana obbligatoria prima di procedere
- Se manca evidenza → l'agente scrive "Confidence: Low — serve validazione" e si ferma

### Gate umano ✅
Il PRD deve essere letto e approvato dall'umano **prima** di creare qualsiasi WI ADO.

---

## Fase 3 — Decomposizione Backlog (Agent_Backlog_Planner)

### Input
PRD validato dall'umano.

### WhatIf Preview (obbligatorio)
L'agente mostra **prima** cosa sta per creare:
```
WhatIf Preview:
- 1 Epic: "Marginalia Qdrant integration"
- 2 Feature: "Qdrant backend", "Eval metrics"
- 5 PBI: ...
Confermi? [y/N]
```
**Apply solo dopo conferma esplicita umana.**

### Gerarchia ADO
```
Epic       ← iniziativa (trimestrale)
  Feature  ← capability (bisettimanale)
    PBI    ← task sviluppabile in 1-3 giorni
      Task ← sotto-attività
```

### Regola Palumbo
**MAI iniziare sviluppo senza WI. MAI creare PR senza WI.**
Il WI ADO deve avere nel description il link alla guida wiki.

### Platform Adapter
Il processo è identico su ADO, GitHub Issues, Atlassian, Forgejo — cambia solo l'adapter.
EasyWay usa ADO come primario; `hale-bopp-data` usa GitHub Issues.

### Gate umano ✅
Backlog Check prima di passare a sviluppo.

---

## Fase 4 — Sprint Execution

### Classificazione task
Prima di iniziare: `AI-automatable` vs `human-only`.

### Flusso sviluppo

```bash
# 1. Branch per PBI
git checkout -b feat/pbi-1234-marginalia-qdrant

# 2. Sviluppo + commit con AB# (obbligatorio su GitHub)
git commit -m "feat: add qdrant eval backend AB#1234"

# 3. Iron Dome (obbligatorio)
ewctl commit   # NON git commit diretto

# 4. Push + PR
git push origin feat/pbi-1234-marginalia-qdrant
# PR body deve contenere: AB#1234
```

### PrGuardian (L3 audit)
- Secrets scan
- Branch policy check (Palumbo Rule, no fast-forward)
- Quality gate

### Scenari speciali
| Scenario | Comportamento |
|---|---|
| PRD rimandato indietro (es. "manca AES-256") | Agent si ferma, richiede PRD update |
| Conflitto ADO (PBI già esiste) | Segnala, propone collegamento — non forza |
| Audit failure (segreto nel commit) | Fix chirurgico (`git reset` / `bfg`) — non cancella solo il file |
| Hotfix urgente | PRD compresso ma Iron Dome sempre attivo |

### Gate umano ✅ — UAT in staging
UAT obbligatorio prima della release su `main`.

---

## Fase 5 — Wiki Aggiornata + RAG

Dopo il merge, **obbligatorio**:

1. Guida wiki: `status: planned` → `status: active`
2. Compilare **Cosa/Perché/Come/Q&A**
3. Aggiungere Troubleshooting con errori reali trovati
4. Se cambia auth/connessione: aggiornare `guides/connection-registry.md`

```bash
# RAG ingest (da SSH server)
source /opt/easyway/.env.secrets
QDRANT_API_KEY=$QDRANT_API_KEY WIKI_PATH=Wiki \
  node scripts/ingest_wiki.js > /tmp/ingest.log 2>&1
```

**Regola**: se un agente non riesce a usare la feature leggendo solo la guida wiki, la guida è incompleta.

---

## Gate Umani — Riepilogo

| Gate | Quando | Chi | Cosa valuta |
|---|---|---|---|
| **PRD Gate** | Dopo Discovery | Umano IT/Business | Correttezza PRD, evidence, scope |
| **Backlog Gate** | Dopo WhatIf Preview | Umano | Struttura WI, stima, priorità |
| **UAT Gate** | Prima di release | Umano | Funzionalità in staging |

**Nessun gate è bypassabile.** Non di notte, non in hotfix, non "solo questa volta".

---

## Esempio Concreto — marginalia eval Qdrant

```
1. IDEA → initiatives-backlog.md:
   "eval dovrebbe usare Qdrant reale, non solo TF-IDF"

2. DISCOVERY (Agent_Discovery con RAG):
   - RAG su wiki: trova marginalia/eval.py, Qdrant collection easyway_wiki
   - Evidence: guides/qdrant-ops.md, marginalia/eval.py
   - Confidence: High
   - PRD: guides/marginalia-eval-qdrant.md
     → Cosa: backend Qdrant opzionale, fallback TF-IDF
     → Scope OUT: non rimuove zero-dep offline mode
     → IL RAGIONAMENTO: offline users non hanno Qdrant → dual backend

3. Gate PRD ✅ → umano approva

4. BACKLOG (WhatIf Preview):
   "Sto per creare: 1 Feature, 2 PBI. Confermi?"
   → PBI #1456 "marginalia: eval Qdrant backend"
   → PBI #1457 "marginalia: eval docs + test Qdrant"

5. Gate Backlog ✅ → umano approva

6. SVILUPPO:
   git checkout -b feat/pbi-1456-eval-qdrant
   git commit -m "feat: qdrant eval backend AB#1456"
   ewctl commit → Iron Dome ok → PR

7. Gate UAT ✅ → test su vault reale

8. MERGE → wiki update:
   guides/marginalia-eval-qdrant.md → status: active
   RAG ingest → agenti trovano la guida
```

---

## Stato implementazione (Human-as-Agent Protocol)

> GEDI Case #41 — Il processo è operativo oggi con umani. L'automazione viene dopo.

| Fase | Agente target | Stato oggi |
|---|---|---|
| Discovery & PRD | Agent_Discovery | **Human-as-Agent** (Claude Code + template PRD) |
| Backlog Planner | Agent_Backlog_Planner | **Human** + MCP `ado_wi_create` + WhatIf manuale |
| Developer | Agent_Developer | **Human** + `ewctl commit` |
| PrGuardian | Agent_PrGuardian | **Parziale** — Iron Dome + `ado_pr_poll_status` |
| Wiki update | — | **Obbligatorio manuale** post-merge |

"Human-as-Agent" = seguire lo stesso template PRD, stessi gate, stessa Evidence/Confidence — anche se lo fa un umano o Claude Code.

---

## Fast-Track vs Full-Track (GEDI Case #41)

**Decision tree — 3 domande, ≤30 secondi:**

```
Fast-Track SE tutte e tre vere:
  □ Produzione è down OPPURE vulnerabilità di sicurezza attiva?
  □ Modifica ≤ 1 file, ≤ 50 righe?
  □ Zero impatto su API contract, DB schema, altri team?

Full-Track SE anche solo una vera:
  □ Nuova feature o capability
  □ Modifica DB schema o API contract
  □ Impatto cross-team o cross-repo
  □ Nuova dipendenza esterna
  □ Stima > 1 giorno di lavoro
```

**Fast-Track non azzera governance**: Iron Dome sempre attivo, AB# obbligatorio, PRD compresso (3 righe: Cosa / Perché / Rollback).

---

## Wiki Health Monitor (GEDI Case #41)

La wiki è il sistema nervoso degli agenti. Se degrada silenziosamente, Agent_Discovery produce PRD con evidence sbagliata — con Confidence: High.

**Monitoraggio settimanale** (cron o n8n sul server):

```bash
marginalia eval snapshot ~/easyway-wiki/guides/ wiki-queries.yaml \
  snap-$(date +%Y%m%d).json --top-k 5
marginalia eval compare snap-previous.json snap-$(date +%Y%m%d).json
# Se verdict=DEGRADED → alert
```

**CI warning** (non bloccante): se una PR tocca codice ma nessun file wiki, il check mostra:
```
⚠️  Wiki sync: nessuna guida aggiornata in questa PR.
```

---

## Regole Assolute

1. **Wiki prima** — nessun PBI senza PRD o backlog entry
2. **RAG obbligatorio in Discovery** — nessuna tabula rasa
3. **Evidence + Confidence** — ogni decisione PRD
4. **WhatIf Preview** — prima di creare ADO WI
5. **Palumbo Rule** — nessuna PR senza WI (`AB#` obbligatorio)
6. **Iron Dome** — `ewctl commit` sempre, mai `git commit` diretto
7. **Wiki dopo** — nessun merge senza aggiornare la guida
8. **RAG vive** — la wiki non aggiornata = agenti ciechi
