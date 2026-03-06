---
id: ew-initiatives-backlog
title: Initiatives Backlog — Sala d'Attesa
summary: Iniziative e task pending che diventeranno Epic/PBI su ADO quando pronti. Fonte di verità pre-backlog.
status: active
owner: team-platform
created: '2026-03-04'
updated: '2026-03-06'  # Session 86 - n8n repo, Docker cleanup
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
| ~~.cursorrules GitHub refs update~~ | ~~Da fare~~ | ✓ S71: verificato — i ~15 riferimenti sono storici (session notes, chronicles). Refs operativi (pipeline, git remote) gia aggiornati S63 |

## 2. Branch Cleanup — Debito Tecnico ✓ COMPLETATA S68

**Risultato (Session 68)**:

| Repo | Prima (locale+remoto) | Dopo | Eliminati |
|---|---|---|---|
| portal | 100 + 151 | 2 + 9 protetti | **-240** |
| agents | 8 + 11 | 1 + 2 | **-16** |
| wiki | 10 + 15 | 1 + 3 | **-21** |
| infra | 6 + 10 | 1 + 1 | **-14** |

**9 branch portal protetti** da ADO branch policy (ForcePush) — eliminabili solo da admin ADO web.
~~**Dependabot**: 30 vulnerabilita su easyway-portal (1 critical, 22 high) — da fixare.~~ ✓ S70: npm audit fix PR #312 (0 vulns)

## 3. Infrastruttura & CI/CD

| Item | Priorita | Note |
|---|---|---|
| Re-enable deploy stages in pipeline | Media | deploy.sh prod funzionante (S65-66), valutare se serve anche stage pipeline o basta deploy.sh |
| Fix 3 container falliti | Alta | Build context punta a `agents/Dockerfile` che non esiste piu nel monorepo |
| ~~Fix GitHubMirror URL in portal pipeline~~ | ~~Alta~~ | ✓ Completato PR #282 (S63) |
| ~~Pipeline split per-repo~~ | ~~Media~~ | ✓ S71: PR #316-#318, pipeline registrate e verdi. Wiki (normalize+lint), agents (manifest+shellcheck), infra (compose validate+shellcheck). GitHub mirror da server secrets |
| ~~GitHub branch protection rules~~ | ~~Media~~ | ✓ S71: wiki + agents (public) protetti (no delete, force push per mirror CI). Portal + infra (private) richiedono GitHub Pro |
| Pipeline trigger optimization | Bassa | Oggi ogni push a develop triggera full build+test anche per docs-only changes. Path filter o skip CI |
| ~~Iron Dome hooks ripristino~~ | ~~Media~~ | ✓ S69: pre-commit bash hook + setup-hooks.sh + bugfix single-line scan. PR #310 |
| Iron Dome modularizzazione | Media | **PBI #94** — Pattern registry YAML, `.iron-dome.json` per-repo config, parametri esclusione. Candidato per spin-off open-source (vedi §6 HALE-BOPP) |
| ~~GitHub PAT scaduto~~ | ~~Media~~ | ✓ Rinnovato S68 — `ADO-GitHub-Mirror` token, credential store su server, 3 posti (da 7) |
| ~~PAT scope: aggiungere Build (Read)~~ | ~~Media~~ | ✓ Completato S66 — briefing ora mostra pipeline runs |
| ~~Rimuovere `version` obsoleto dai docker-compose~~ | ~~Bassa~~ | ✓ Completato PR #299 (S66) |
| ~~Dependabot: 3 vulnerabilita high su easyway-infra~~ | ~~Alta~~ | ✓ Completato PR #298 (S66) — minimatch 0 vulns |
| ~~CI deploy gates: disabilitare in pipeline~~ | ~~Alta~~ | ✓ PR #300 merged (S66, GEDI Case #26) |
| ewctl.ado-pr.psm1 refactoring | Media | Estrarre logica ArtifactLink+conflict da Create-ReleasePR.ps1 in modulo condiviso (GEDI Case #24) |
| ~~easyway-ado: Phase 0-2 completate~~ | ~~Media~~ | ✓ Phase 0-2 (S75-S76). CLI 10 comandi, PAT routing, MCP stub 10 tool, notification formatter, Palumbo enforcement. Residuo: Phase 3-5 (MCP full, npm publish). Vedi [use cases](use-cases/easyway-ado.md) |
| ~~easyway-ado: feat→main guard~~ | ~~Media~~ | ✓ S77: check strutturale in `prCreate` — blocca feature→main, eccezioni per develop→main e release/hotfix |
| ~~easyway-ado: MAX 2 retry nel client HTTP~~ | ~~Bassa~~ | ✓ S77: retry loop in `ado-client.ts` — max 2 tentativi, 500ms pausa, solo su errori di rete |
| ~~easyway-ado: MCP safety-by-design comments~~ | ~~Bassa~~ | ✓ S77: commento strutturale in `mcp/index.ts` — pr vote/complete non esposti di proposito |
| ~~n8n workflow repo dedicato (`easyway-n8n`)~~ | ~~Media~~ | ✓ S86: repo creato, PR #357 merged, PBI #105. 10 workflow (2 common, 2 business, 2 infra, 4 template). Circle 3 private ADO. Wiki repos/ scheda + factory.yml (9 repo) |
| ~~Docker compose: .env server mancante~~ | ~~Alta~~ | Risolto S85: `~/easyway-infra/.env` gia creato in S84 con tutte le variabili (verificato). Nessuna azione necessaria |
| ~~Docker compose: container name conflict~~ | ~~Alta~~ | Risolto S85: tutti i container sono sotto compose (progetto `easyway-prod`). 5 container `easyway-cert-*` zombie da rimuovere + `easyway-seq` orfano. Wiki: `infrastructure/container-inventory.md` |
| **n8n job: container census (watchdog)** | Media | S85-S86: template creato in `easyway-n8n/workflows/infra/container-census-watchdog.json`. Da importare in n8n e attivare. Prerequisito: Docker socket montato nel container n8n o SSH |
| **n8n job: Docker Health Daily Report** | **Alta** | S86: template creato in `easyway-n8n/workflows/infra/docker-health-report.json`. Da importare in n8n e attivare. Prerequisito: Docker socket montato nel container n8n o SSH. Soglie: disco>70%, cache>5GB, container unhealthy, volumi orfani |
| ~~Caddy reverse proxy: Internal Server Error~~ | ~~Alta~~ | Risolto S85: verificato — HTTP 200 dall'esterno e internamente. Container `easyway-portal` ha alias DNS `frontend` su `easyway-net`. Caddy reverse_proxy funziona correttamente |
| **Deploy workflow: compose vs docker run** | Media | S84: il container portal era stato creato con `docker run` (immagine `easyway/frontend:latest`), non con compose (che genera `easyway-infra-frontend:latest`). Il deploy ha richiesto `docker tag` + `docker rm` + `docker run` manuale. Standardizzare: o tutto compose o tutto docker run con script. Documentare in `.cursorrules` Step 3 il metodo effettivo |
| easyway-ado: Phase 4 — guardrails configurabili `.guardrails.yml` | Bassa | GEDI Case #33: Tier 1 (Palumbo, safety-by-design) resta hardcoded forever. Tier 2 (feat→main, duplicate PR, branch exceptions) configurabile via YAML quando ci saranno 5+ regole di flusso. Trigger: secondo progetto/team che usa easyway-ado |

**Dipendenze**: i container dipendono dal fix dei Dockerfile path dopo polyrepo split.
**S71**: ADO branch protection configurata su wiki, agents, infra (min reviewers + merge strategy + WI linking). GitHub branch protection pending.

## 4. Database — Scelta Tecnologica

| Item | Priorita | Note |
|---|---|---|
| Scelta database (PostgreSQL vs SQL Edge vs altro) | Alta | sql-edge ha `profiles: [sql]` in docker-compose.yml, ignorato su ARM64 |
| hale-bopp-db come candidato | Media | Schema governance PostgreSQL, CLI `halebopp`, 17 test, porta 8100 |
| MongoDB per catalogo agenti | Media | Document store per agent registry (manifest JSON a struttura variabile). Valutare quando catalogo supera ~20 agenti. Alternativa leggera: JSON file o collection Qdrant |

**Contesto**: il server e ARM64 (OCI), SQL Edge non supporta ARM nativamente. PostgreSQL e il candidato naturale, allineato con HALE-BOPP.

## 5. Agents Platform

| Item | Priorita | Note |
|---|---|---|
| GEDI Casebook commit (Case #17, #18) | Bassa | **PBI #92** — Locale in `agents/agent_gedi/`, ora pushabile dopo merge PR #276 |
| Agent runner L2/L3 test post-polyrepo | Media | Verificare che i path `Import-AgentSecrets` funzionino ancora |
| Skills registry update post-polyrepo | Media | `agents/skills/registry.json` — path references da verificare |
| **Levi 2.0 — Doc Guardian polyrepo** | **Alta** | S79: modernizzare path, `md:fix` multi-repo (wiki/agents/infra), `session:closeout` automatizza checklist. Usa easyway-ado SDK per WI update. **Accoppiata Levi+Wiki repos/** (S81): Levi enforcea frontmatter delle schede repo (tag, cerchio, campi obbligatori) — dogfooding del pattern "Pagine Gialle" su noi stessi prima dei clienti |
| GEDI spin-off open-source | Media | **PBI #93** — GEDI come advisory ethical framework standalone. Blue ocean, zero competitor. Packaging: manifest + casebook + integration guide. Circle 1 |
| **Levi prodotto Obsidian** | Media | Levi come plugin/CLI standalone per studenti su Obsidian vault: frontmatter enforcement, link integrity, tag taxonomy, RAG-ready chunking. Target: vault accademici, tesi, appunti. Prodotto estraibile Circle 1 (open-source) |
| Agent message queuing pattern | Media | Pattern per agenti che processano messaggi utente in coda mentre eseguono tool. 3 livelli: (1) coda semplice — iniettare al turno successivo (~50 righe SDK), (2) reazione immediata — loop check tra tool call, (3) vero parallelismo — multi-thread/multi-agent. Candidato Agentic Playbook |
| **Valentino: browser capability (Playwright MCP)** | Media | S85: dare a Valentino la capacita di navigare e analizzare siti web come un browser subagent. Due opzioni: (1) **MCP Playwright** (`@playwright/mcp` — navigate, screenshot, click, get_dom) — piu semplice, self-hosted, sovereign. (2) **browser-use** (Python, `pip install browser-use`) — piu potente, controllo browser completo. Preferire opzione 1 (MCP) per coerenza con architettura MCP gia in uso (easyway-ado). Repo: `microsoft/playwright-mcp`. Gira su server Oracle ARM = sovereign, nessun dato esce. Prerequisito: Valentino manifest update + config MCP |

## 6. HALE-BOPP

| Item | Priorita | ADO | Note |
|---|---|---|---|
| ~~hale-bopp-db Sprint 1 completamento~~ | ~~Media~~ | ~~PBI #34-#37~~ | Done S72 — 17 test, GitHub Actions CI |
| ~~hale-bopp-etl rewrite~~ | ~~Bassa~~ | ~~PBI #38~~ | Done S73 — v0.3.0 lightweight runner, 30 test, PR #1 merged |
| ~~hale-bopp deploy su server~~ | ~~Alta~~ | — | Done S73 — 4 systemd services, tutti running |
| ~~3 repo GitHub con README vetrina~~ | ~~Media~~ | — | Done S73 — badge CI, architettura ASCII, traceability ADO |
| hale-bopp-argos conversione | Bassa | — | Da servizio HTTP a libreria Python (parcheggiato, servizio funziona) |
| **hale-bopp-iron-dome** (NUOVO) | Media | — | Pre-commit secrets scanner modulare. Spin-off da `ewctl.secrets-scan.psm1`. Pattern registry YAML, per-repo config, multi-language (PS+bash). Apache 2.0 |
| PostgreSQL integration test | Media | — | DB + ETL + ARGOS end-to-end con Postgres reale |

**Org GitHub**: `hale-bopp-data` (Apache 2.0, full open source)
**Services running**: DB :8100, ETL webhook :3001, ETL watcher (poll), ARGOS :8200 — systemd, auto-restart, bind 127.0.0.1
**Nota**: Iron Dome nasce come modulo interno EasyWay (S49) e matura come hook universale (S69). Candidato naturale per open-source: problema universale, zero dipendenze business.

## 6b. Connection Registry & Agent Multi-Platform (S73)

| Item | Priorita | Note |
|---|---|---|
| ~~Connection Registry creato~~ | ~~Alta~~ | Done S73 — `agents/scripts/connections/` con github.sh, ado.sh, server.sh, qdrant.sh, connections.yaml |
| ~~.env.local Unicode fix~~ | ~~Alta~~ | Done S73 — rimossi caratteri box-drawing, parser robusto KEY=VALUE |
| halebopp.sh connettore aggiunto | Done S78 | healthcheck 3 servizi via SSH, diff/snapshot via API. Fix CRLF `_load_env` in github.sh e qdrant.sh |
| Connection Registry: `_common.sh` + env overlays | Media | **PBI #90** — Estrarre `_load_env` in `_common.sh` (sourced da tutti). Supportare `connections.{env}.yaml` overlays (dev/prod). Obiettivo: un solo punto parametrizzabile |
| Agent multi-platform (ADO + GitHub) | Media | **PBI #91** — Agenti esistenti (pr_gate, review) parlano solo ADO. Estendere via Electrical Socket Pattern: stessa interfaccia, connettore diverso. I connettori github.sh/ado.sh sono il layer di astrazione |
| ADO-GitHub traceability convention | Bassa | Commit msg: `ADO: PBI #XX`. PR body: link GitHub. WI description: link PR GitHub. Automatizzabile in github.sh |
| OpenRouter connector | Bassa | `connections/openrouter.sh` — completa il registry |

**Wiki**: `guides/connection-registry.md` documenta il pattern.

## 7. Security & Secrets Management

| Item | Priorita | Note |
|---|---|---|
| Full RBAC env segregation | Media | Implementare modello `.env.discovery/.planner/.executor` da `infra/config/environments/README.md` — oggi tutto in unico `.env.local` |
| PAT resolver function `Get-AdoPat -Role <role>` | Bassa | Elimina hardcoding nomi variabile PAT negli script; shell wrapper per bash |
| Secrets Registry expiry alerting | **Alta** | **PBI #95** — `Invoke-SecretsScan.ps1` esiste ma non e automatizzato — n8n cron? **GEDI Case #28**: PAT scaduto ha rotto mirror per giorni senza allarme |
| SSH key GitHub sul server | Media | **PBI #96** — Eliminare PAT da `~/.git-credentials`, usare deploy key SSH — zero scadenza, zero rotazione |
| ADO Variable Groups PAT scope | Bassa | Nessuno dei 4 PAT ha scope `Variable Groups: Read & Manage` — aggiornare ADO Library richiede portale web manuale |
| ADO Build Queue PAT scope | Bassa | Nessuno dei 4 PAT ha scope `Build: Read & Execute` — non possiamo lanciare pipeline via API, solo dal portale |
| GitHub PAT rotation calendar | Media | Token `ADO-GitHub-Mirror` scade **Jun 2 2026** — impostare reminder 2 settimane prima |
| **Server hardening: RBAC + blocco SCP/SFTP + deploy user** | **Alta** | **PBI #104** (Epic #62 Security) — S85: collega ha bypassato deploy workflow via SCP. Implementare RBAC 4-tier, blocco SCP/SFTP, utente deploy con ForceCommand, sudoers limitato. Wiki aggiornata: `security/threat-analysis-hardening.md` (Scenario 2b) + `infra/security-framework.md` (sezione 3). GEDI: victory_before_battle + testudo_formation |

**Contesto**: GEDI Case #19 (S62) — `.env.local` ha 4 PAT separati per scope (bene), ma serviti da un unico file leggibile da tutti i processi. Il modello RBAC multi-file in `infra/config/environments/` e documentato ma non implementato.

## 8. Sfide Ecosistemiche — Aree di Attenzione

> Emerse durante review architetturale post-polyrepo (S69). Rischi sistemici che richiedono attenzione proattiva.

| Item | Priorita | Note |
|---|---|---|
| CI/CD cross-repo integration testing | Alta | 8 repo interconnessi. Se API portal cambia, n8n workflow si rompe? Serve versionamento semantico dei contratti API + test integrazione tra repo. deploy.sh (Electrical Socket) e il bridge, ma non valida compatibilita |
| GitHub mirror drift prevention | Media | wiki e infra sync dietro ADO. Mirror GitHub devono essere **read-only** tranne bot sync. Se qualcuno pusha direttamente su GitHub mirror → drift. Aggiungere branch protection rules su GitHub: solo bot puo pushare a main |
| Secrets management evolution | Media | `/opt/easyway/.env.secrets` e un file unico con God-Tokens (DEEPSEEK, GITEA, QDRANT). Post-polyrepo, container diversi servono sottoinsiemi diversi. Valutare: HashiCorp Vault / Azure Key Vault per rimpiazzare `.env.secrets` locale quando architettura cresce |

**Dipendenze**: ~~CI/CD testing dipende da pipeline split (§3)~~. Pipeline split completato S71. Secrets evolution dipende da RBAC segregation (§7).

## 9. Knowledge Lifecycle — Level 2 (Automazione)

| Item | Priorita | Note |
|---|---|---|
| Skill `session.closeout` automatica | Bassa | n8n trigger su PR merge per verificare aggiornamento wiki |
| RAG re-index automatico post wiki update | Bassa | **PBI #97** — Qdrant ingest dopo merge su easyway-wiki. Rate limiting in ingest_wiki.js. Agent scopetta come cron |
| MCP tool ado_rag_resolve in easyway-ado | Media | **PBI #98** — Tool MCP che usa rag-search (porta 8300) per risolvere WI da linguaggio naturale |
| **n8n → .cursorrules auto-gen** | Media | S82: .cursorrules generato dalla wiki via n8n. Trigger: post-merge wiki. n8n legge sezioni chiave wiki, genera .cursorrules snello, committa su portal. Elimina drift tra wiki e cursorrules. Prerequisito: n8n repo dedicato (>10 workflow) |
| PRD → Epic → Wiki → RAG pipeline | Futura | Flusso completo documentato in MEMORY.md |
| **Levi per Nuove Pagine Gialle** | Media | Levi come doc-guardian per directory/catalogo aziende — scan qualita schede, frontmatter enforcement, link integrity, auto-tag per categorie. Pattern: ogni scheda e' un .md con frontmatter strutturato → Levi garantisce coerenza e completezza del catalogo |

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
| **Wiki repos/ section: 8 schede repo + _index.md + tag taxonomy** | 2026-03-05 | S81 |
| **ADR: Python per ADO batch API** | 2026-03-05 | S81 |
| **ADO backlog cleanup: 12 PBI chiusi + 9 PBI creati sotto Feature vuote** | 2026-03-05 | S81 |
| **pip install hale-bopp-db (PBI #35)** | 2026-03-05 | S78 |
| **PR guardrails: duplicate/feat-to-main/safety (PBI #88)** | 2026-03-05 | S77 |
| **halebopp.sh connector + CRLF fix (PBI #89)** | 2026-03-05 | S78 |
| **Pipeline split per-repo (wiki, agents, infra) — 3 pipeline verdi** | 2026-03-04 | S71 |
| **ADO branch protection su wiki, agents, infra** | 2026-03-04 | S71 |
| **npm audit fix portal: 0 vulnerabilita (PR #312)** | 2026-03-04 | S70 |
| **PR Readiness Check in Get-ADOBriefing (PR #313)** | 2026-03-04 | S70 |
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
