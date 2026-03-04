---
id: ew-initiatives-backlog
title: Initiatives Backlog — Sala d'Attesa
summary: Iniziative e task pending che diventeranno Epic/PBI su ADO quando pronti. Fonte di verità pre-backlog.
status: active
owner: team-platform
created: '2026-03-04'
updated: '2026-03-04'  # Session 62
tags: [planning, backlog, roadmap, initiatives, domain/platform, layer/reference, audience/dev, privacy/internal, language/it]
entities: []
llm:
  include: true
  pii: none
  chunk_hint: 300-500
type: planning
---

# Initiatives Backlog — Sala d'Attesa

> Questa pagina raccoglie le iniziative e i task pending prima che diventino Epic/PBI formali su ADO.
> Aggiornata ad ogni sessione. Fonte di verita per la prossima sessione.

## Come usare questa pagina

1. **Aggiungere**: quando emerge un task/idea durante una sessione, aggiungerlo qui
2. **Promuovere**: quando un'iniziativa e matura, creare Epic/PBI su ADO e segnare "Promosso → PBI #XX"
3. **Archiviare**: quando completata, spostare in fondo nella sezione "Completate"

---

## 1. La Fabbrica — Polyrepo Finalization ✓ COMPLETATA

Phase 3c chiusa Session 62. Tutte le PR merged, tutti i repo su main.

**Residui**:

| Item | Stato | Note |
|---|---|---|
| GitHub mirror wiki — 2 commit dietro ADO | Da sync | Pipeline GitHubMirror non copre wiki/infra, push manuale necessario |
| GitHub mirror infra — 2 commit dietro ADO | Da sync | Idem |
| Branch cleanup portal — ~144 locali + ~195 remoti | Da fare | Debito tecnico enorme accumulato pre-polyrepo |
| GitHub Actions sync-to-ado.yml | Da configurare | Workflow vive in easyway-infra ma GitHub Actions lo cerca nel repo easyway-portal |
| .cursorrules GitHub refs update | Da fare | ~15 riferimenti a `belvisogi/EasyWayDataPortal` da aggiornare a `easyway-data/easyway-portal` |

## 2. Branch Cleanup — Debito Tecnico (Portal)

**Inventario (2026-03-04)**:
- Locali: 144 branch (48 gia merged in main)
- Remoti: 195 branch (50 gia merged in main)
- **Azione**: eliminare i 48+50 branch merged, poi valutare gli unmerged

| Batch | Tipo | Count | Azione |
|---|---|---|---|
| Merged locali | `git branch --merged main` | 48 | `git branch -d <name>` — sicuro |
| Merged remoti | `git branch -r --merged main` | 50 | `git push origin --delete <name>` — richiede approvazione |
| Unmerged locali | `git branch --no-merged main` | ~96 | Valutare caso per caso |
| Unmerged remoti | `git branch -r --no-merged main` | ~145 | Valutare caso per caso |

## 3. Infrastruttura & CI/CD

| Item | Priorita | Note |
|---|---|---|
| Re-enable deploy stages in pipeline | Alta | Disabilitati in S58, pending multi-repo setup con `deploy.sh` |
| Fix 3 container falliti | Alta | Build context punta a `agents/Dockerfile` che non esiste piu nel monorepo |
| Iron Dome hooks ripristino | Media | `ewctl commit` pre-commit hooks da riattivare |
| GitHub PAT `.env.github` e `.env.publish` scaduti | Media | 401 errors, rigenerare |

**Dipendenze**: i container dipendono dal fix dei Dockerfile path dopo polyrepo split.

## 4. Database — Scelta Tecnologica

| Item | Priorita | Note |
|---|---|---|
| Scelta database (PostgreSQL vs SQL Edge vs altro) | Alta | sql-edge ha `profiles: [sql]` in docker-compose.yml, ignorato su ARM64 |
| hale-bopp-db come candidato | Media | Schema governance PostgreSQL, CLI `halebopp`, 17 test, porta 8100 |

**Contesto**: il server e ARM64 (OCI), SQL Edge non supporta ARM nativamente. PostgreSQL e il candidato naturale, allineato con HALE-BOPP.

## 5. Agents Platform

| Item | Priorita | Note |
|---|---|---|
| GEDI Casebook commit (Case #17, #18) | Bassa | Locale in `agents/agent_gedi/`, ora pushabile dopo merge PR #276 |
| Agent runner L2/L3 test post-polyrepo | Media | Verificare che i path `Import-AgentSecrets` funzionino ancora |
| Skills registry update post-polyrepo | Media | `agents/skills/registry.json` — path references da verificare |

## 6. HALE-BOPP

| Item | Priorita | ADO | Note |
|---|---|---|---|
| hale-bopp-db Sprint 1 completamento | Media | PBI #34-#37 | 17 test verdi, schema governance funzionante |
| hale-bopp-etl rewrite | Bassa | — | Rimuovere Dagster, custom runner ~300 righe |
| hale-bopp-argos conversione | Bassa | — | Da servizio HTTP a libreria Python |

**Org GitHub**: `hale-bopp-data` (Apache 2.0, full open source)

## 7. Security & Secrets Management

| Item | Priorita | Note |
|---|---|---|
| Full RBAC env segregation | Media | Implementare modello `.env.discovery/.planner/.executor` da `infra/config/environments/README.md` — oggi tutto in unico `.env.local` |
| PAT resolver function `Get-AdoPat -Role <role>` | Bassa | Elimina hardcoding nomi variabile PAT negli script; shell wrapper per bash |
| Secrets Registry expiry alerting | Bassa | `Invoke-SecretsScan.ps1` esiste ma non e automatizzato — n8n cron? |

**Contesto**: GEDI Case #19 (S62) — `.env.local` ha 4 PAT separati per scope (bene), ma serviti da un unico file leggibile da tutti i processi. Il modello RBAC multi-file in `infra/config/environments/` e documentato ma non implementato.

## 8. Knowledge Lifecycle — Level 2 (Automazione)

| Item | Priorita | Note |
|---|---|---|
| Skill `session.closeout` automatica | Bassa | n8n trigger su PR merge per verificare aggiornamento wiki |
| RAG re-index automatico post wiki update | Bassa | Qdrant ingest dopo merge su easyway-wiki |
| PRD → Epic → Wiki → RAG pipeline | Futura | Flusso completo documentato in MEMORY.md |

## 9. Archivio — Materiale Recuperabile

Fonte: `C:\old\EasyWayDataPortal-archive\` (127 file, 1.2 MB, no git)

| Item | Priorita | Fonte | Note |
|---|---|---|---|
| mvp_wiki_dq revival — skill DQ per wiki | Media | `archive/mvp_wiki_dq/` | 6 script PS (scorecard, orphans, gap, tags, links, graph-view). Complementari al RAG Qdrant. Candidati per skill in easyway-agents |
| Legacy DDL SQL preservation | Bassa | `archive/old/db/_ARCHIVED_DDL/` | Schema originale EasyWay DataPortal (tables, SP, provisioning). Rilevante per scelta database §4 |
| UX specs agentiche | Bassa | `archive/old/wiki-loose/Wiki/UX/` | `agentic-ux-guidelines.md` + `agentic-ux.md` — bozze UX per agent-console |

**Scartabili**: `.bak` files (backup superati), CSV/JSONL duplicati (artefatti generati), artifacts (log snippets)

---

## Completate (Archivio)

| Item | Completato | Sessione |
|---|---|---|
| **La Fabbrica Phase 3c COMPLETATA** | 2026-03-04 | S62 |
| PR #278 infra GitHub mirror target update | 2026-03-04 | S62 |
| PR #275 wiki rename refs merged | 2026-03-04 | S62 |
| PR #273, #274 portal rename refs merged | 2026-03-04 | S62 |
| factory.yml aggiornato con stato reale | 2026-03-04 | S62 |
| Tutti i repo locali sync su main | 2026-03-04 | S62 |
| easyway-agents import 672 file su main | 2026-03-04 | S61 |
| easyway-agents GitHub mirror sync | 2026-03-04 | S61 |
| ADO repo rename EasyWayDataPortal → easyway-portal | 2026-03-03 | S60 |
| Server folder rename ~/EasyWayDataPortal → ~/easyway-portal | 2026-03-03 | S60 |
| CI/CD pipeline semplificato (779 → 230 righe) | 2026-03-02 | S58 |
| Phase 2 easyway-infra extraction | 2026-03-01 | S57 |
| Phase 3a monorepo cleanup (-1420 file) | 2026-03-01 | S57 |
| Phase 0 easyway-wiki extraction | 2026-02-28 | S54 |

---

> *GEDI ricorda: "Non ne parliamo, risolviamo" (Velasco) — questa pagina esiste per agire, non per accumulare.*
