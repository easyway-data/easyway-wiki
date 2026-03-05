---
title: "Polyrepo Git Workflow - Guida Operativa Completa"
created: 2026-03-05
updated: 2026-03-05
status: active
category: guides
domain: infrastructure
tags: [git, polyrepo, workflow, governance, pr, branch, domain/platform, layer/how-to, audience/dev, audience/agent, privacy/internal, language/it]
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
main (o develop per portal)
  └─ feat/mia-feature
       ├─ commit 1
       ├─ commit 2
       └─ PR → develop (portal) o main (wiki/agents/infra/ado)
```

**Nota**: fix CI/infra urgenti possono andare direttamente `feat→main` (coerente con PR #176-#184).

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

```bash
B64=$(bash /c/old/easyway/agents/scripts/ado-auth.sh pr)
curl -s -X POST \
  -H "Authorization: Basic $B64" \
  -H "Content-Type: application/json" \
  "https://dev.azure.com/EasyWayData/EasyWay-DataPortal/_apis/git/repositories/<repo-id>/pullrequests?api-version=7.1" \
  -d '{"sourceRefName":"refs/heads/feat/mia-feature","targetRefName":"refs/heads/main","title":"PBI-95: ..."}'
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

## 10. Troubleshooting

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

---

> **Riferimenti**:
> - `wiki/guides/git-rebase-policy.md` — Rebase safety (GEDI Case #22)
> - `wiki/guides/agentic-pbi-to-pr-workflow.md` — PBI→PR agentic flow
> - `wiki/guides/connection-registry.md` — Connection Registry
> - `wiki/security/secrets-governance.md` — Secrets Bible
> - `wiki/repos/` — Schede operative per ogni repo
> - `easyway/factory.yml` — Mappa polyrepo
