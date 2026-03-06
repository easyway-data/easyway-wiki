---
title: "Polyrepo Git Workflow - Guida Operativa Completa"
created: 2026-03-05
updated: 2026-03-05
status: active
category: guides
domain: infrastructure
tags: [domain/git, domain/polyrepo, process/workflow, process/governance, process/pr, domain/branch, domain/platform, layer/how-to, audience/dev, audience/agent, privacy/internal, language/it]
entities: [easyway-portal, easyway-wiki, easyway-agents, easyway-infra, easyway-ado]
llm:
  include: true
  pii: none
  chunk_hint: 300-500
type: guide
---

# Polyrepo Git Workflow - Guida Operativa Completa

> Questa guida documenta il giro completo di git per la piattaforma EasyWay polyrepo.
> Destinata a sviluppatori umani e agenti IA che devono operare sui repository.
> Fonte di verita per: commit, branch, PR, deploy, governance.

---

## 0. Mappa Logica — Orientamento Rapido

```
                          +---------------------------+
                          |     ADO (Azure DevOps)    |
                          |  Backlog, PR, CI Pipeline |
                          |  dev.azure.com/EasyWayData|
                          +---------------------------+
                                     |
                    PR create/merge, WI tracking
                                     |
  +----------------------------------------------------------------------+
  |                    SVILUPPO LOCALE (Windows)                         |
  |                    C:\old\easyway\                                   |
  |                                                                      |
  |  portal/    wiki/     agents/    infra/     ado/                     |
  |  (Node.js)  (Markdown) (PS+bash)  (Docker)   (TypeScript)           |
  |    |          |          |          |          |                      |
  |    +----------+----------+----------+----------+                     |
  |              5 REPO GIT INDIPENDENTI                                 |
  |              Ognuno ha il suo .git/                                   |
  |              Commit e PR SEPARATI                                     |
  +----------------------------------------------------------------------+
           |                                        |
      git push                              easyway-ado MCP
      (ADO remote)                          (10+1 tool, stdio)
           |                                        |
  +----------------------------------------------------------------------+
  |                    SERVER OCI (Ubuntu ARM64)                         |
  |                    80.225.86.168 via SSH                              |
  |                                                                      |
  |  ~/easyway-portal/   (git clone, docker compose)                     |
  |  ~/easyway-wiki/     (git clone, RAG ingest source)                  |
  |  ~/easyway-agents/   (git clone, n8n workflows, skills)              |
  |  ~/easyway-infra/    (git clone, deploy.sh, compose overlays)        |
  |                                                                      |
  |  Docker containers:                                                  |
  |    easyway-portal (Node.js app)                                      |
  |    easyway-qdrant (Qdrant v1.16.2, porta 6333)                       |
  |    easyway-memory (RAG search server, porta 8300)                    |
  |    easyway-n8n (orchestrazione, porta 80 via Caddy)                  |
  |    easyway-gitea (git mirror, porta 3100)                            |
  |                                                                      |
  |  Systemd services:                                                   |
  |    hale-bopp-db (:8100)                                              |
  |    hale-bopp-etl-webhook (:3001)                                     |
  |    hale-bopp-argos (:8200)                                           |
  |    rag-search (HTTP :8300)                                           |
  |                                                                      |
  |  Secrets:                                                            |
  |    /opt/easyway/.env.secrets (API keys)                              |
  |    ~/secrets-registry.json (PAT metadata per expiry check)           |
  +----------------------------------------------------------------------+
           |                                        |
      GitHub mirror                          Qdrant RAG
      (push automatico                       (167k+ chunks wiki)
       da CI pipeline)                       (semantic search)
           |
  +----------------------------------------------------------------------+
  |                    GITHUB (mirror pubblico)                          |
  |                                                                      |
  |  easyway-data/easyway-wiki     (Circle 2, source-available)         |
  |  easyway-data/easyway-agents   (Circle 2, source-available)         |
  |  easyway-data/easyway-portal   (Circle 3, private — futuro)         |
  |  hale-bopp-data/hale-bopp-db   (Circle 1, Apache 2.0)              |
  |  hale-bopp-data/hale-bopp-etl  (Circle 1, Apache 2.0)              |
  |  hale-bopp-data/hale-bopp-argos(Circle 1, Apache 2.0)              |
  +----------------------------------------------------------------------+
```

### Flusso tipo: modifica → produzione

```
1. LOCALE: modifica file in C:\old\easyway\<repo>\
2. LOCALE: ewctl commit (Iron Dome scan)
3. LOCALE: git push origin feat/<branch>
4. ADO:    PR con Work Item linkato
5. ADO:    Approvazione umana + merge
6. CI:     Pipeline ADO → build/test → GitHub mirror
7. SERVER: git fetch origin main && git reset --hard origin/main
8. SERVER: docker compose restart (se necessario)
```

---

## 1. Mappa dei Repository

| Repo | Path locale | Path server | ADO | GitHub | Cerchio |
|---|---|---|---|---|---|
| easyway-portal | `C:\old\easyway\portal\` | `~/easyway-portal/` | `easyway-portal` | easyway-data/easyway-portal | 3 (private) |
| easyway-wiki | `C:\old\easyway\wiki\` | `~/easyway-wiki/` | `easyway-wiki` | easyway-data/easyway-wiki | 2 (source-available) |
| easyway-agents | `C:\old\easyway\agents\` | `~/easyway-agents/` | `easyway-agents` | easyway-data/easyway-agents | 2 (source-available) |
| easyway-infra | `C:\old\easyway\infra\` | `~/easyway-infra/` | `easyway-infra` | easyway-data/easyway-infra | 3 (private) |
| easyway-ado | `C:\old\easyway\ado\` | — | `easyway-ado` | — | 3 (private) |

**Root locale**: `C:\old\easyway\` (NON e un repo git — ogni sottocartella e un repo indipendente).

**Mappa factory**: `C:\old\easyway\factory.yml` — 8 repo, 4 fasi, tutte DONE (Session 62).

---

## 2. Branch Strategy

### 2.1 Branch protetti (MAI commit diretto)

| Branch | Repo | Merge strategy | Note |
|---|---|---|---|
| `main` | tutti | Merge (no fast-forward) | Produzione. Solo via PR |
| `develop` | portal | Merge (no fast-forward) | Integrazione. Solo via PR |

### 2.2 Feature branch

**Naming**: `feat/<descrizione>` o `feat/<PBI-id>-<slug>`

**Esempio**: `feat/levi-4.1-repos-enforcement`, `feat/pr-guardrails`

**Flusso standard**:
```
PORTAL (ha develop):
  develop
    └─ feat/mia-feature
         ├─ commit 1
         └─ PR → develop        ← SEMPRE prima qui
  main
    └─ PR develop→main [Release] ← solo quando si rilascia

WIKI / AGENTS / INFRA / ADO (NO develop):
  main
    └─ feat/mia-feature
         ├─ commit 1
         └─ PR → main           ← direttamente
```

> **ATTENZIONE**: su portal, MAI fare PR direttamente a `main` da un feature branch.
> Il flusso e SEMPRE `feat→develop` poi `develop→main [Release]`.
> Gli altri repo NON hanno `develop` — la PR va direttamente a `main`.

**Eccezione rara**: fix CI/infra urgenti possono andare direttamente `feat→main` su portal (coerente con PR #176-#184), ma solo con approvazione esplicita.

### 2.3 Rebase policy (GEDI Case #22)

- **Feature branch**: OK rebase con `git rebase origin/<target>` + `push --force-with-lease`
- **Branch protetti**: MAI rebase, MAI force-push
- **In dubbio**: merge commit (sempre sicuro)
- Guida dettagliata: `wiki/guides/git-rebase-policy.md`

---

## 3. Come Committare

### 3.1 Regola fondamentale

**OGNI REPO HA IL SUO GIT.** Non esiste un commit che tocca file in repo diversi. Se una modifica tocca agents + wiki, servono 2 commit separati.

### 3.2 Commit tool

```bash
# Preferito — include Iron Dome pre-commit secrets scan
ewctl commit

# Se ewctl non disponibile
git add <file1> <file2>
git commit -m "feat: descrizione breve"
```

### 3.3 Convenzione messaggi

```
<tipo>: <descrizione breve>

[corpo opzionale con dettagli]

Co-Authored-By: Claude Opus 4.6 <noreply@anthropic.com>
```

**Tipi**: `feat`, `fix`, `docs`, `refactor`, `test`, `chore`

**Esempio reale** (S82):
```
feat: PBI-95 secrets expiry alerting — n8n v2.0 + registry path fix
feat: PBI-90 extract _load_env to _common.sh + env overlays
feat: add ado_rag_resolve MCP tool — semantic search on wiki via Qdrant RAG
docs: add n8n dedicated repo initiative to backlog (S82)
```

### 3.4 Iron Dome pre-commit

Ogni commit passa per Iron Dome che scansiona per secrets leaked:
```
git commit → pre-commit hook → Invoke-SecretsScan.ps1 → PASS/FAIL
```

Se fallisce: il commit NON viene creato. Rimuovere il secret e riprovare. **MAI `--no-verify`.**

### 3.5 Cosa NON committare

- `.env.local`, `.env.secrets`, `secrets-registry.json` (fuori dai repo)
- File con valori secret hardcoded
- `node_modules/`, `dist/`, `.obsidian/`

---

## 4. Come Creare una PR

### 4.1 Prerequisiti (Regola del Palumbo)

**PRIMA di creare una PR**:
1. Verificare/creare il Work Item ADO associato
2. Verificare che NON esista una PR pendente sugli stessi file (Lesson S63b)
3. Verificare di NON essere su un branch protetto

### 4.2 Via easyway-ado MCP

```json
{
  "tool": "ado_pr_create",
  "params": {
    "repo": "easyway-agents",
    "sourceBranch": "feat/mia-feature",
    "targetBranch": "main",
    "workItemId": 95,
    "title": "PBI-95: secrets expiry alerting"
  }
}
```

Guardrail automatici:
- **Palumbo**: rifiuta PR senza Work Item
- **feat→main guard**: rifiuta feature→main su portal (deve passare per develop)
- **Duplicate PR guard**: rifiuta se esiste PR attiva sullo stesso branch

### 4.3 Via curl ADO API

> **IMPORTANTE**: usare il NOME del repo nell'URL, NON il GUID. I GUID troncati causano errore `TF401019`.

```bash
B64=$(bash /c/old/easyway/agents/scripts/ado-auth.sh pr)
REPO="easyway-agents"  # nomi: easyway-portal, easyway-wiki, easyway-agents, easyway-infra, easyway-ado
curl -s -X POST \
  -H "Authorization: Basic $B64" \
  -H "Content-Type: application/json" \
  "https://dev.azure.com/EasyWayData/EasyWay-DataPortal/_apis/git/repositories/$REPO/pullrequests?api-version=7.1" \
  -d '{"sourceRefName":"refs/heads/feat/mia-feature","targetRefName":"refs/heads/main","title":"PBI-95: ...","workItemRefs":[{"id":"95"}]}'
```

### 4.4 PR flusso per repo

| Repo | Flusso PR | Target branch |
|---|---|---|
| portal | `feat→develop` poi `develop→main [Release]` | develop |
| wiki | `feat→main` o `docs/→main` | main |
| agents | `feat→main` | main |
| infra | `feat→main` | main |
| ado | `feat→main` (da feat/pr-guardrails) | main |

### 4.5 PR Readiness Check

Prima di proporre approvazione, SEMPRE verificare:
1. WI linkati (`pullrequests/{id}/workitems`)
2. Policy evaluations (`policy/evaluations?artifactId=...`)
3. Merge status (no conflicts)
4. Reviewer votes

---

## 5. Come Fare Deploy

### 5.1 Deploy standard (portal)

```bash
# 1. Push feature branch
git push origin feat/mia-feature

# 2. Creare PR feat→develop (via easyway-ado o curl)

# 3. Dopo merge in develop, creare PR develop→main [Release]

# 4. SSH al server e aggiornare
ssh ubuntu@80.225.86.168
cd ~/easyway-portal && git fetch origin main && git reset --hard origin/main

# 5. Test nel container
docker compose restart
```

**MAI `git pull`** — sempre `git fetch + git reset --hard` (Lesson S55).

### 5.2 Deploy multi-repo (infra)

```bash
# Script centralizzato — Compose Coherence Gate (GEDI Case #25)
ssh ubuntu@80.225.86.168
cd ~/easyway-infra && bash scripts/deploy.sh [env]
```

### 5.3 Deploy agents/wiki

```bash
ssh ubuntu@80.225.86.168
cd ~/easyway-agents && git fetch origin main && git reset --hard origin/main
cd ~/easyway-wiki && git fetch origin main && git reset --hard origin/main
```

### 5.4 PAT scaduto durante deploy

```bash
git remote set-url origin 'https://Tokens:<PAT>@dev.azure.com/EasyWayData/EasyWay-DataPortal/_git/<repo>'
git fetch origin main && git reset --hard origin/main
```

---

## 6. Governance: Regole Assolute

Queste regole NON sono negoziabili. Nessun agente o sviluppatore puo aggirarle.

| # | Regola | Motivo |
|---|---|---|
| G1 | MAI approvare/votare una PR | Azione visibile, richiede ok umano |
| G2 | MAI completare/mergiare una PR senza ok esplicito | Irreversibile |
| G3 | MAX 2 tentativi API ADO per la stessa azione | Anti brute-force |
| G4 | MAI usare `-ExecutionPolicy Bypass` | Security policy |
| G5 | MAI pushare direttamente a main | Branch protection |
| G6 | MAI mergere PR senza verificare i commit | Le PR si chiudono automaticamente |
| G7 | MAI creare PR senza Work Item (Palumbo) | Tracciabilita |
| G8 | MAI creare PR se ce n'e una pendente sugli stessi file | Conflitti merge |
| G9 | MAI `git pull` in deploy — sempre fetch+reset | Consistenza |
| G10 | MAI `--no-verify` per skippare hooks | Iron Dome e un guardrail |
| G11 | MAI `--force` — usare `--force-with-lease` | Safety net |
| G12 | Se `develop` esiste nel repo, PR verso `main` bloccate — passare da `develop` | Auto-detect dinamico |

> **G12 — PR Target Branch Policy (Session 88)**
> Se un repo ha il branch `develop`, le PR verso `main` sono bloccate — devono passare prima da `develop`.
> - **Logica dinamica**: il guard controlla se `develop` esiste nel repo. Se si, solo `develop` e `release/*` possono puntare a `main`.
> - **Enforcement a 3 livelli**:
>   1. Pipeline `BranchPolicyGuard` (`azure-pipelines.yml`) — check `git ls-remote` a runtime
>   2. Modulo `ewctl.vcs.psm1` (`New-AdoPullRequest`) — check API ADO refs prima di creare la PR
>   3. `factory-vcs.json` (`default_target_branch`) — fallback config per script
> - **Best practice**: se vuoi il flusso `feat→develop→main`, crea il branch `develop`. Il sistema lo rileva e attiva l'enforcement automaticamente.

---

## 7. Sessione Tipo: Checklist Operativa

```
1. git branch --show-current          (verificare di essere sul branch giusto)
2. Lavoro sui file
3. ewctl commit                       (Iron Dome pre-commit)
4. git push origin <branch>           (push al remote)
5. Creare PR via easyway-ado o curl   (con WI linkato)
6. Verificare PR readiness            (policy, WI, merge status)
7. Attendere approvazione umana       (MAI auto-approvare)
8. Deploy sul server                  (fetch+reset, MAI pull)
```

### 7.1 Quando la modifica tocca piu repo

**Esempio reale (S82)**: PBI #95 ha toccato agents + wiki.

```
1. cd easyway/agents/
   - Modificare file
   - git add <files>
   - git commit -m "feat: PBI-95 ..."

2. cd easyway/wiki/
   - Modificare file
   - git add <files>
   - git commit -m "docs: ..."

3. Push separati (possono essere paralleli):
   - cd easyway/agents && git push origin <branch>
   - cd easyway/wiki && git push origin <branch>

4. PR separate su ADO (una per repo, stesso WI linkato)
```

---

## 8. PAT Routing

Gli agenti usano PAT segregati per operazioni diverse. MAI indovinare quale PAT usare.

```bash
# Il PAT router decide automaticamente
B64=$(bash /c/old/easyway/agents/scripts/ado-auth.sh <action>)
```

| Action | PAT | Scope |
|---|---|---|
| `pr` | `ADO_PR_CREATOR_PAT` | Code R/W, PR Contribute |
| `wi` | `ADO_WORKITEMS_PAT` | Work Items R/W |
| `build` | `AZURE_DEVOPS_EXT_PAT` | Code R/W, Build Read |
| `general` | `AZURE_DEVOPS_EXT_PAT` | Briefing, branch list |
| `github` | `GITHUB_PAT` | repo Contents:Write |

**Scadenza**: tutti rinnovati 2026-03-03, scadono 2026-06-01 (90gg).
**Alerting**: `Invoke-SecretsScan.ps1` con `secrets-registry.json` — CRITICAL <3gg, HIGH <14gg.

---

## 9. Tool e Connettori Disponibili

### 9.1 easyway-ado MCP (10 tool)

| Tool | Operazione |
|---|---|
| `ado_briefing` | Project overview |
| `ado_wi_get` | Get WI by ID |
| `ado_wi_list` | List active WIs |
| `ado_wi_query` | Execute WIQL |
| `ado_wi_create` | Create PBI/Feature/Epic |
| `ado_wi_update` | Update WI |
| `ado_pr_list` | List PRs |
| `ado_pr_get` | Get PR details |
| `ado_pr_create` | Create PR (con guardrail) |
| `ado_pr_policy` | Check PR policy |
| `ado_rag_resolve` | Semantic search wiki via Qdrant RAG |

**Start MCP**: `npm run mcp:stdio` (da `easyway-ado/`)

### 9.2 Connection Registry

| Connettore | Uso |
|---|---|
| `github.sh healthcheck` | Verifica GitHub auth |
| `ado.sh healthcheck` | Verifica ADO auth |
| `qdrant.sh search "query"` | Semantic search wiki |
| `server.sh exec "cmd"` | Esegui comando su server |
| `halebopp.sh healthcheck` | Verifica servizi HALE-BOPP |
| `healthcheck-all.sh` | Verifica tutti i connettori |

**Env overlay**: `CONN_ENV=server` carica `connections.server.env` (override path/URL per server).

---

## 10. Ricettario Comandi (copia-incolla)

Tutte le operazioni quotidiane, con comandi esatti. Locale = Windows dev. Server = Ubuntu OCI.

### 10.1 SSH al server

```bash
# Da Git Bash / terminal locale
"/c/Windows/System32/OpenSSH/ssh.exe" -i "/c/old/Virtual-machine/ssh-key-2026-01-25.key" ubuntu@80.225.86.168

# Oppure via connector
bash /c/old/easyway/agents/scripts/connections/server.sh exec "uptime"
```

### 10.2 Aggiornare un repo sul server

```bash
# SSH al server, poi:
cd ~/easyway-portal && git fetch origin main && git reset --hard origin/main
cd ~/easyway-agents && git fetch origin main && git reset --hard origin/main
cd ~/easyway-wiki && git fetch origin main && git reset --hard origin/main
cd ~/easyway-infra && git fetch origin main && git reset --hard origin/main
```

### 10.3 Riavviare i container dopo deploy

```bash
# SSH al server, poi:
cd ~/easyway-infra
source /opt/easyway/.env.secrets
OPENAI_API_KEY=placeholder ANTHROPIC_API_KEY=placeholder docker compose up -d

# Riavviare solo un servizio
docker compose restart <service-name>
```

### 10.4 Verificare stato servizi

```bash
# Da locale (via connector)
bash /c/old/easyway/agents/scripts/connections/healthcheck-all.sh

# Da server
docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"
systemctl status hale-bopp-db hale-bopp-etl-webhook hale-bopp-argos
```

### 10.5 Lanciare RAG re-index manuale

```bash
# SSH al server, poi:
source /opt/easyway/.env.secrets
cd ~/easyway-portal
QDRANT_API_KEY=$QDRANT_API_KEY WIKI_PATH=../easyway-wiki node scripts/ingest_wiki.js > /tmp/ingest.log 2>&1

# Verificare chunk count
curl -s "http://localhost:6333/collections/easyway_wiki" \
  -H "api-key: $QDRANT_API_KEY" | \
  python3 -c "import sys,json; d=json.load(sys.stdin); print('chunks:', d['result']['points_count'])"
```

### 10.6 Secrets scan manuale

```bash
# Da locale
pwsh -File C:\old\easyway\agents\agents\skills\security\Invoke-SecretsScan.ps1 \
  -ScanPath C:\old\easyway\agents -RegistryPath C:\old\secrets-registry.json -OutputFormat markdown

# Da server (verifica PAT expiry)
pwsh -File ~/easyway-agents/agents/skills/security/Invoke-SecretsScan.ps1 \
  -ScanPath ~/easyway-portal -RegistryPath ~/secrets-registry.json -Json
```

### 10.7 Creare un Work Item ADO

```bash
# Via MCP (se easyway-ado MCP server attivo)
# Tool: ado_wi_create { type: "Product Backlog Item", title: "...", parentId: 4 }

# Via curl
B64=$(bash /c/old/easyway/agents/scripts/ado-auth.sh wi)
curl -s -X POST \
  -H "Authorization: Basic $B64" \
  -H "Content-Type: application/json-patch+json" \
  "https://dev.azure.com/EasyWayData/EasyWay-DataPortal/_apis/wit/workitems/\$Product%20Backlog%20Item?api-version=7.1" \
  -d '[{"op":"add","path":"/fields/System.Title","value":"Titolo PBI"}]'
```

### 10.8 Creare una PR ADO

```bash
# Via MCP (preferito — ha tutti i guardrail)
# Tool: ado_pr_create { repo: "easyway-agents", sourceBranch: "feat/...", workItemId: 95 }

# Via curl (usare NOME repo, NON GUID — i GUID troncati danno TF401019)
B64=$(bash /c/old/easyway/agents/scripts/ado-auth.sh pr)
REPO="easyway-agents"  # nomi validi: easyway-portal, easyway-wiki, easyway-agents, easyway-infra, easyway-ado
curl -s -X POST \
  -H "Authorization: Basic $B64" \
  -H "Content-Type: application/json" \
  "https://dev.azure.com/EasyWayData/EasyWay-DataPortal/_apis/git/repositories/$REPO/pullrequests?api-version=7.1" \
  -d '{"sourceRefName":"refs/heads/feat/mia-feature","targetRefName":"refs/heads/main","title":"PBI-95: ...","workItemRefs":[{"id":"95"}]}'
```

### 10.9 Copiare un file sul server (senza scp — via cat+SSH)

```bash
# Da locale, iniettare file sul server
cat /c/old/secrets-registry.json | \
  "/c/Windows/System32/OpenSSH/ssh.exe" -i "/c/old/Virtual-machine/ssh-key-2026-01-25.key" \
  ubuntu@80.225.86.168 "cat > ~/secrets-registry.json"
```

### 10.10 Nomi repository per API ADO

> **REGOLA**: usare SEMPRE il **nome** del repo nell'URL API, NON il GUID.
> I GUID troncati (8 char) causano `TF401019: GitRepositoryNotFoundException`.
> I GUID completi (36 char) sono in `factory.yml` — servono solo per ArtifactLink.

**Nomi validi per API** (usare questi):
```
easyway-portal
easyway-wiki
easyway-agents
easyway-infra
easyway-ado
```

**Pattern URL**:
```
https://dev.azure.com/EasyWayData/EasyWay-DataPortal/_apis/git/repositories/<NOME>/...
```

**GUID completi** (solo per ArtifactLink `vstfs:///Git/PullRequestId/...`):
| Repo | GUID completo |
|---|---|
| Project | `d8c907d2-dae1-4db1-9dcf-40c2931b6dc7` |
| easyway-portal | `e29d5c59-28d0-4815-8f62-6f5c1465855c` |
| easyway-wiki | `d055dfa8-db64-4c9d-a6ad-55bff6e8d767` |
| easyway-agents | `fa068c67-b5ee-4b41-aab3-38b0166ffb27` |
| easyway-infra | `fa453541-4227-4837-bd55-8b92f656f87a` |
| easyway-ado | `91b146f8-8037-4823-b673-b50147e1a22f` |

---

## 11. Troubleshooting

### PR rifiutata: "Palumbo mutu non po essere servuto"
Manca il Work Item. Creare prima il WI, poi la PR.

### PR rifiutata: "feat→main not allowed"
Sul portal, feature branch deve andare prima a develop. Cambiare target branch.

### Commit rifiutato da Iron Dome
Secret rilevato nel file. Rimuovere il valore e usare `${VAR}` reference.

### Deploy fallisce: "fatal: not a git repository"
`C:\old\` NON e un repo. Entrare nel repo specifico (`cd easyway/portal/`).

### Deploy fallisce: PAT scaduto
Rinnovare su ADO portal, aggiornare `.env.local` + Variable Group + server credentials. Vedi sezione 5.4.

### Merge conflict su PR
Rebase il feature branch: `git fetch origin main && git rebase origin/main && git push --force-with-lease`

### API ADO fallisce 2 volte
STOP. Non riprovare. Verificare PAT, scope, URL. Chiedere aiuto umano.

### TF401019: GitRepositoryNotFoundException
Stai usando un GUID troncato (8 char) nell'URL API. **Usare il NOME del repo** (`easyway-wiki`, non `d055dfa8`).
L'API accetta sia nomi che GUID completi (36 char), ma i GUID troncati falliscono sempre.

---

> **Riferimenti**:
> - `wiki/guides/git-rebase-policy.md` — Rebase safety (GEDI Case #22)
> - `wiki/guides/agentic-pbi-to-pr-workflow.md` — PBI→PR agentic flow
> - `wiki/guides/connection-registry.md` — Connection Registry
> - `wiki/security/secrets-governance.md` — Secrets Bible
> - `wiki/repos/` — Schede operative per ogni repo
> - `easyway/factory.yml` — Mappa polyrepo
