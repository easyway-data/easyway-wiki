---
id: ew-initiatives-backlog
title: Initiatives Backlog — Sala d'Attesa
summary: Iniziative e task pending che diventeranno Epic/PBI su ADO quando pronti. Fonte di verità pre-backlog.
status: active
owner: team-platform
created: '2026-03-04'
updated: '2026-03-04'  # Session 68
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
| ~~GitHub mirror wiki~~ | ~~Da sync~~ | ✓ Pushed S68 — remote `github` configurato su server con credential store |
| ~~GitHub mirror infra~~ | ~~Da sync~~ | ✓ Pushed S68 — idem |
| ~~Branch cleanup portal~~ | ~~Da fare~~ | ✓ S68: da 100+151 a 2+9 branch. 9 residui protetti da ADO policy |
| GitHub Actions sync-to-ado.yml | Da configurare | Workflow vive in easyway-infra ma GitHub Actions lo cerca nel repo easyway-portal |
| .cursorrules GitHub refs update | Da fare | ~15 riferimenti a `belvisogi/EasyWayDataPortal` da aggiornare a `easyway-data/easyway-portal` |

## 2. Branch Cleanup — Debito Tecnico ✓ COMPLETATA S68

**Risultato (Session 68)**:

| Repo | Prima (locale+remoto) | Dopo | Eliminati |
|---|---|---|---|
| portal | 100 + 151 | 2 + 9 protetti | **-240** |
| agents | 8 + 11 | 1 + 2 | **-16** |
| wiki | 10 + 15 | 1 + 3 | **-21** |
| infra | 6 + 10 | 1 + 1 | **-14** |

**9 branch portal protetti** da ADO branch policy (ForcePush) — eliminabili solo da admin ADO web.
**Dependabot**: 30 vulnerabilita su easyway-portal (1 critical, 22 high) — da fixare.

## 3. Infrastruttura & CI/CD

| Item | Priorita | Note |
|---|---|---|
| Re-enable deploy stages in pipeline | Media | deploy.sh prod funzionante (S65-66), valutare se serve anche stage pipeline o basta deploy.sh |
| Fix 3 container falliti | Alta | Build context punta a `agents/Dockerfile` che non esiste piu nel monorepo |
| ~~Fix GitHubMirror URL in portal pipeline~~ | ~~Alta~~ | ✓ Completato PR #282 (S63) |
| Pipeline split per-repo | Media | Oggi 1 pipeline portal (semplificata) + 1 vecchia in infra (monstre). Servono pipeline dedicate per wiki, agents, infra |
| GitHub branch protection rules | Media | Portare regole ADO su GitHub: require PR, require review, no force push, status checks required, CODEOWNERS |
| Pipeline trigger optimization | Bassa | Oggi ogni push a develop triggera full build+test anche per docs-only changes. Path filter o skip CI |
| ~~Iron Dome hooks ripristino~~ | ~~Media~~ | ✓ S69: pre-commit bash hook + setup-hooks.sh + bugfix single-line scan. PR #310 |
| Iron Dome modularizzazione | Media | Pattern registry YAML, `.iron-dome.json` per-repo config, parametri esclusione. Candidato per spin-off open-source (vedi §6 HALE-BOPP) |
| ~~GitHub PAT scaduto~~ | ~~Media~~ | ✓ Rinnovato S68 — `ADO-GitHub-Mirror` token, credential store su server, 3 posti (da 7) |
| ~~PAT scope: aggiungere Build (Read)~~ | ~~Media~~ | ✓ Completato S66 — briefing ora mostra pipeline runs |
| ~~Rimuovere `version` obsoleto dai docker-compose~~ | ~~Bassa~~ | ✓ Completato PR #299 (S66) |
| ~~Dependabot: 3 vulnerabilita high su easyway-infra~~ | ~~Alta~~ | ✓ Completato PR #298 (S66) — minimatch 0 vulns |
| CI deploy gates: disabilitare in pipeline | Alta | ✓ PR #300 (S66, GEDI Case #26) — da mergiare |
| ewctl.ado-pr.psm1 refactoring | Media | Estrarre logica ArtifactLink+conflict da Create-ReleasePR.ps1 in modulo condiviso (GEDI Case #24) |

**Dipendenze**: i container dipendono dal fix dei Dockerfile path dopo polyrepo split.
**Sessione dedicata**: pipeline split + GitHub governance richiedono pianificazione architetturale (GEDI).

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
| **hale-bopp-iron-dome** (NUOVO) | Media | — | Pre-commit secrets scanner modulare. Spin-off da `ewctl.secrets-scan.psm1`. Pattern registry YAML, per-repo config, multi-language (PS+bash). Apache 2.0 |

**Org GitHub**: `hale-bopp-data` (Apache 2.0, full open source)
**Nota**: Iron Dome nasce come modulo interno EasyWay (S49) e matura come hook universale (S69). Candidato naturale per open-source: problema universale, zero dipendenze business.

## 7. Security & Secrets Management

| Item | Priorita | Note |
|---|---|---|
| Full RBAC env segregation | Media | Implementare modello `.env.discovery/.planner/.executor` da `infra/config/environments/README.md` — oggi tutto in unico `.env.local` |
| PAT resolver function `Get-AdoPat -Role <role>` | Bassa | Elimina hardcoding nomi variabile PAT negli script; shell wrapper per bash |
| Secrets Registry expiry alerting | **Alta** | `Invoke-SecretsScan.ps1` esiste ma non e automatizzato — n8n cron? **GEDI Case #28**: PAT scaduto ha rotto mirror per giorni senza allarme |
| SSH key GitHub sul server | Media | Eliminare PAT da `~/.git-credentials`, usare deploy key SSH — zero scadenza, zero rotazione |
| ADO Variable Groups PAT scope | Bassa | Nessuno dei 4 PAT ha scope `Variable Groups: Read & Manage` — aggiornare ADO Library richiede portale web manuale |
| ADO Build Queue PAT scope | Bassa | Nessuno dei 4 PAT ha scope `Build: Read & Execute` — non possiamo lanciare pipeline via API, solo dal portale |
| GitHub PAT rotation calendar | Media | Token `ADO-GitHub-Mirror` scade **Jun 2 2026** — impostare reminder 2 settimane prima |

**Contesto**: GEDI Case #19 (S62) — `.env.local` ha 4 PAT separati per scope (bene), ma serviti da un unico file leggibile da tutti i processi. Il modello RBAC multi-file in `infra/config/environments/` e documentato ma non implementato.

## 8. Sfide Ecosistemiche — Aree di Attenzione

> Emerse durante review architetturale post-polyrepo (S69). Rischi sistemici che richiedono attenzione proattiva.

| Item | Priorita | Note |
|---|---|---|
| CI/CD cross-repo integration testing | Alta | 8 repo interconnessi. Se API portal cambia, n8n workflow si rompe? Serve versionamento semantico dei contratti API + test integrazione tra repo. deploy.sh (Electrical Socket) e il bridge, ma non valida compatibilita |
| GitHub mirror drift prevention | Media | wiki e infra sync dietro ADO. Mirror GitHub devono essere **read-only** tranne bot sync. Se qualcuno pusha direttamente su GitHub mirror → drift. Aggiungere branch protection rules su GitHub: solo bot puo pushare a main |
| Secrets management evolution | Media | `/opt/easyway/.env.secrets` e un file unico con God-Tokens (DEEPSEEK, GITEA, QDRANT). Post-polyrepo, container diversi servono sottoinsiemi diversi. Valutare: HashiCorp Vault / Azure Key Vault per rimpiazzare `.env.secrets` locale quando architettura cresce |

**Dipendenze**: CI/CD testing dipende da pipeline split (§3). Secrets evolution dipende da RBAC segregation (§7).

## 9. Knowledge Lifecycle — Level 2 (Automazione)

| Item | Priorita | Note |
|---|---|---|
| Skill `session.closeout` automatica | Bassa | n8n trigger su PR merge per verificare aggiornamento wiki |
| RAG re-index automatico post wiki update | Bassa | Qdrant ingest dopo merge su easyway-wiki |
| PRD → Epic → Wiki → RAG pipeline | Futura | Flusso completo documentato in MEMORY.md |

## 10. Archivio — Materiale Recuperabile

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
| **Branch cleanup 4 repo: -291 branch** | 2026-03-04 | S68 |
| **GitHub PAT rinnovato + credential store server** | 2026-03-04 | S68 |
| **GitHub mirror 4 repo pushed** | 2026-03-04 | S68 |
| **CI deploy gates disabled + GEDI Case #26** | 2026-03-04 | S66 |
| **Dependabot minimatch 3 CVEs fixed (PR #298)** | 2026-03-04 | S66 |
| **docker-compose version removed (PR #299)** | 2026-03-04 | S66 |
| **PAT Build Read scope aggiunto** | 2026-03-04 | S66 |
| **Docker network coherence fix + Compose Coherence Gate** | 2026-03-04 | S65-66 |
| **deploy.sh prod validated + orphan cleanup** | 2026-03-04 | S66 |
| **WI auto-linking in Create-ReleasePR.ps1** | 2026-03-04 | S64 |
| **GEDI Cases #24-#25 documented** | 2026-03-04 | S65 |
| **GitHubMirror URL fix PR #282** | 2026-03-04 | S63 |
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
