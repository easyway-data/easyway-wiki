---
title: "Gnosis Framework - Il Sistema Nervoso Agentico"
tags: [gnosis, architettura, agenti, governance, memory, sdlc, sovereign, framework]
status: active
created: 2026-03-07
---

# Gnosis Framework

> Il framework che definisce chi sono gli agenti, come costruiscono, e come ricordano.
> Tre layer indipendenti ma interconnessi, unificati dalla visione Sovereign.

---

## Quadro d'insieme

```
                    GNOSIS FRAMEWORK
    ============================================

    L1  CHI SONO             agentic-architecture-enterprise.md
        Tassonomia, Ruoli, Livelli L1-L5, Governance

    L2  COME COSTRUISCONO    idea-to-production.md
        Ciclo SDLC, Gate umani, Fast/Full Track

    L3  COME RICORDANO       agent-context-truth-memory-ledger.md
        Context Truth, Memory Ledger, Sprint Rooms

    ============================================
    SOVEREIGN VISION         (visione unificante)
    Kortex, Father-Evaluation, Cloni, Antifragilita
```

---

## L1 - Chi Sono gli Agenti

**Documento**: [agentic-architecture-enterprise.md](agentic-architecture-enterprise.md)

Definisce lo standard architetturale enterprise:

- **Agente vs Automazione**: distinzione basata su input, logica, adattabilita
- **Due dimensioni**: Ruolo (Brain/Arm) x Livello (L1-L5)
- **Governance**: RBAC-A, Deterministic Gatekeeping, Budget & Stop-Loss
- **KPI**: Technical, Governance, Financial, Safety
- **Promotion Criteria**: soglie misurabili per salire di livello
- **Anti-pattern**: il "Super Agent" monolitico

Principio cardine:
> L1 e sempre l'ultimo step prima dell'azione. L'orchestrazione dall'alto fornisce intelligenza; l'esecuzione dal basso fornisce sicurezza.

---

## L2 - Come Costruiscono

**Documento**: [idea-to-production.md](idea-to-production.md)

Il ciclo completo dalla nascita di un'idea al codice in produzione:

```
IDEA -> Wiki Backlog -> Discovery & PRD -> ADO Epic/PBI -> Sviluppo -> PR -> Merge -> Wiki aggiornata -> RAG ingest
```

- **5 fasi** con gate umani obbligatori
- **Fast-Track vs Full-Track**: decision tree basato su complessita e rischio
- **Human-as-Agent Protocol**: l'umano partecipa come agente con ruolo definito
- **Wiki Health Monitor**: metriche di salute della knowledge base
- **AB# convention**: commit GitHub con `AB#1234` aggiorna WI ADO automaticamente

Principi cardine:
> La Wiki e la prima fonte di verita. Tutto parte da qui.
> Gli agenti accelerano il ciclo, ma non rimuovono governance.

Documento master esteso: [agentic-sdlc-master.md](agentic-sdlc-master.md)

---

## L3 - Come Ricordano e Apprendono

**Documento**: [agent-context-truth-memory-ledger.md](agent-context-truth-memory-ledger.md)

Risolve il problema della Context Truth frammentata e dell'apprendimento attivo:

- **Context Truth**: dove risiede la verita aggiornata? (.cursorrules, wiki, RAG, manifest)
- **Memory Ledger**: apprendimento attivo (Local Ledger + Global Ledger)
- **Memoria a Due Velocita**: Workspace (short-term) vs Global (long-term)
- **Il Portinaio**: router leggero che smista agenti senza scrivere codice
- **Sprint Room**: workspace isolati con cloni degli agenti
- **Father-Evaluation**: blind review oggettiva da parte degli agenti "padre"

Principio cardine:
> Solo i Principi Cardinali estratti dal lavoro vengono distillati nella Memoria Globale. Il rumore resta murato nel workspace.

---

## Visione Sovereign

La visione unificante che lega i 3 layer. Codename: **Project Kortex**.

> In un mondo dove l'AI e puro autocompletamento, il Framework (la Governance) e Sovrano.
> Le regole non si piegano alla fretta, non bypassano la CI/CD e non barattano mai la Qualita per la Velocita.

### Kortex (Orchestrator L0)
Il "Mediano della Passione" -- non scrive codice, non allucina codice. Conosce solo *chi* sa fare *cosa*.
- Prende in input requisiti di business, epiche, PRD
- Legge le Carte d'Identita degli agenti (manifest + micro-RAG)
- Decide la squadra (Roster) per ogni attivita

### Clean-Room Architecture
- **Sprint Room**: workspace isolato per ogni PBI/Feature
- **Cloni**: copie temporanee e vergini degli agenti, con semafori di sincronizzazione
- **Father-Evaluation**: i Padri giudicano SOLO l'output finale (blind review, anti-bias)

### Il flusso Sovereign
1. Arriva la Task (PRD/PBI)
2. **Kortex** assegna il Roster
3. Si apre la **Sprint Room** con i Cloni
4. I Cloni lavorano, accumulano memoria locale
5. Output finale (PR/PRD) presentato ai **Padri**
6. I Padri approvano o stroncano (Blind Review)
7. Solo il "succo" distillato diventa memoria globale

---

## Rischi e Mitigazioni (GEDI Case #42)

### Rischio 1: Documentare il futuro a scapito del presente

La visione Sovereign (Kortex, Sprint Rooms, Father-Evaluation) e L4/L5. Oggi il sistema e solidamente a L2-L3. Il rischio e spendere troppo tempo a documentare il futuro invece di consolidare il presente.

**Principi GEDI attivati**: G14 (Start Small), G2 (Quality over Speed), G5 (Pragmatic Action), G6 (Quality Leap)

**Mitigazione: Sovereign Maturity Gate**

La visione Sovereign resta come documento di *destinazione* (stato: planned). Ma ogni componente Sovereign puo essere *implementato* solo quando il prerequisito L2-L3 e **measured & stable**.

Prerequisiti concreti prima di avviare qualsiasi componente Sovereign:
1. `process.workspace-brainstorm` operativo (skill obbligatoria L3+)
2. `wiki-queries.yaml` in produzione con metriche reali
3. Wiki Health Monitor con verdict != DEGRADED per almeno 5 sessioni consecutive

La visione **tira** il sistema avanti senza forzarlo. Se L2 non regge, Sovereign aspetta. Se L2 funziona, Sovereign accelera. Questo trasforma l'ambizione da rischio a motore.

### Rischio 2: Volume di legge scritta che diventa freno

MEMORY.md, regole, checklist, casebook crescono senza potatura. Il volume impressionante di governance scritta puo diventare peso morto se nessuno lo verifica attivamente.

**Principi GEDI attivati**: G8 (Absence of Evidence), G15 (Known Bug over Chaos), G10 (Testudo Formation)

**Mitigazione: Wiki Pruning Cycle**

Un meccanismo a feedback negativo — piu il sistema cresce, piu il pruning lo contiene:

1. **marginalia** scanna la wiki e produce metriche (link rotti, pagine orfane, tag mancanti)
2. **wiki-queries.yaml** definisce le query di salute (ridondanze, contraddizioni, pagine non aggiornate da N giorni)
3. Ogni ~10 sessioni, una **sessione di potatura** dedicata: rimuovere/archiviare doc stantie, unificare regole sovrapposte, aggiornare MEMORY.md (prossima: S108)
4. **Levi** enforcea la struttura — se una pagina non ha frontmatter, tag, o link, la segnala

Il volume diventa antifragile: lo stress (crescita) attiva il meccanismo correttivo (pruning) invece di accumularsi come entropia.

---

## Stato implementativo

| Componente | Stato | Note |
|-----------|-------|------|
| Tassonomia L1-L5 | attivo | Agenti classificati in manifest.json |
| RBAC-A | attivo | rbac-master.json |
| Ciclo SDLC (L2) | attivo | idea-to-production.md operativo |
| AB# convention | attivo | hale-bopp-data org connessa ad ADO |
| Memory Ledger | planned | LOCAL_LEDGER.md per agente |
| Context Truth unificata | planned | Single source da definire |
| Kortex (L0) | planned | Portinaio/Router |
| Sprint Rooms | planned | Workspace isolati |
| Father-Evaluation | planned | Blind review |
| Wiki Health Monitor | planned | wiki-queries.yaml |
| process.workspace-brainstorm | planned | Skill obbligatoria L3+ |

---

## Relazioni con altri documenti

| Documento | Relazione |
|-----------|-----------|
| [agentic-sdlc-master.md](agentic-sdlc-master.md) | Documento master esteso per L2 |
| [repo-semaphore.md](repo-semaphore.md) | Meccanismo di concorrenza tra sessioni |
| `agents/platform-operational-memory.md` | Memoria operativa corrente |
| `planning/initiatives-backlog.md` | Backlog iniziative (prossimi step) |
| GEDI Casebook | Consultazioni architetturali documentate |

---

## Skill mancante critica

**`process.workspace-brainstorm`** — Obbligatorio L3+, prima di ogni Full-Track.
- v1: warning se non eseguito
- v2: blocking (non si procede senza brainstorm)
- Da aggiungere a `agents/skills/registry.json`
