---
id: ew-initiatives-backlog
title: Initiatives Backlog — Sala d'Attesa
summary: Iniziative e task pending che diventeranno Epic/PBI su ADO quando pronti. Fonte di verità pre-backlog.
status: active
owner: team-platform
created: '2026-03-04'
updated: '2026-03-04'
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

## 1. La Fabbrica — Polyrepo Finalization (Phase 3c)

| Item | Stato | Note |
|---|---|---|
| PR #274 portal docs/session-60-refs-update → develop | Pending approval | Approvare PRIMA di #273 |
| PR #273 portal develop → main (Release S60) | Pending approval | Approvare DOPO #274 |
| PR #275 wiki docs/session-60-rename-refs → main | Pending approval | |
| PR #276 agents initial-import → main | Merged (S61) | GitHub mirror fatto |

**Criterio di completamento**: tutte le PR merged, tutti i repo allineati ADO ↔ GitHub.

## 2. Infrastruttura & CI/CD

| Item | Priorita | Note |
|---|---|---|
| Re-enable deploy stages in pipeline | Alta | Disabilitati in S58, pending multi-repo setup con `deploy.sh` |
| Fix 3 container falliti | Alta | Build context punta a `agents/Dockerfile` che non esiste piu nel monorepo |
| Fix GitHubMirror URL in portal pipeline | Alta | Riga 251: ancora `belvisogi/EasyWayDataPortal` → `easyway-data/easyway-portal` |
| Pipeline split per-repo | Media | Oggi 1 pipeline portal (semplificata) + 1 vecchia in infra (monstre). Servono pipeline dedicate per wiki, agents, infra |
| GitHub branch protection rules | Media | Portare regole ADO su GitHub: require PR, require review, no force push, status checks required, CODEOWNERS |
| Pipeline trigger optimization | Bassa | Oggi ogni push a develop triggera full build+test anche per docs-only changes. Path filter o skip CI |
| Iron Dome hooks ripristino | Media | `ewctl commit` pre-commit hooks da riattivare |
| GitHub PAT `.env.github` e `.env.publish` scaduti | Media | 401 errors, rigenerare |

**Dipendenze**: i container dipendono dal fix dei Dockerfile path dopo polyrepo split.
**Sessione dedicata**: pipeline split + GitHub governance richiedono pianificazione architetturale (GEDI).

## 3. Database — Scelta Tecnologica

| Item | Priorita | Note |
|---|---|---|
| Scelta database (PostgreSQL vs SQL Edge vs altro) | Alta | sql-edge ha `profiles: [sql]` in docker-compose.yml, ignorato su ARM64 |
| hale-bopp-db come candidato | Media | Schema governance PostgreSQL, CLI `halebopp`, 17 test, porta 8100 |

**Contesto**: il server e ARM64 (OCI), SQL Edge non supporta ARM nativamente. PostgreSQL e il candidato naturale, allineato con HALE-BOPP.

## 4. Agents Platform

| Item | Priorita | Note |
|---|---|---|
| GEDI Casebook commit (Case #17, #18) | Bassa | Locale in `agents/agent_gedi/`, ora pushabile dopo merge PR #276 |
| Agent runner L2/L3 test post-polyrepo | Media | Verificare che i path `Import-AgentSecrets` funzionino ancora |
| Skills registry update post-polyrepo | Media | `agents/skills/registry.json` — path references da verificare |

## 5. HALE-BOPP

| Item | Priorita | ADO | Note |
|---|---|---|---|
| hale-bopp-db Sprint 1 completamento | Media | PBI #34-#37 | 17 test verdi, schema governance funzionante |
| hale-bopp-etl rewrite | Bassa | — | Rimuovere Dagster, custom runner ~300 righe |
| hale-bopp-argos conversione | Bassa | — | Da servizio HTTP a libreria Python |

**Org GitHub**: `hale-bopp-data` (Apache 2.0, full open source)

## 6. Knowledge Lifecycle — Level 2 (Automazione)

| Item | Priorita | Note |
|---|---|---|
| Skill `session.closeout` automatica | Bassa | n8n trigger su PR merge per verificare aggiornamento wiki |
| RAG re-index automatico post wiki update | Bassa | Qdrant ingest dopo merge su easyway-wiki |
| PRD → Epic → Wiki → RAG pipeline | Futura | Flusso completo documentato in MEMORY.md |

---

## Completate (Archivio)

| Item | Completato | Sessione |
|---|---|---|
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
