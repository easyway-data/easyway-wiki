# Agent ADO Operations Guide

> How any AI agent (Claude, Cursor, Gemini, Copilot) interacts with Azure DevOps in the EasyWay project.
> Zero prerequisites scontati. Se non sai dove partire, parti da qui.

## Cosa c'e' e dove

| Cosa | Dove | A cosa serve |
|------|------|-------------|
| **easyway-ado** | `C:\old\easyway\ado\` (Windows) / `~/easyway-ado/` (server) | CLI + MCP Server per Azure DevOps |
| **PAT (Personal Access Tokens)** | Server: `/opt/easyway/.env.secrets` / Dev: `C:\old\.env.local` | 4 PAT con scope diversi (vedi tabella sotto) |
| **MCP config Claude Code** | `C:\old\.mcp.json` | Registra easyway-ado come MCP server |
| **MCP config Cursor** | `C:\old\.cursor\mcp.json` | Registra easyway-ado come MCP server |
| **ADO Organization** | `https://dev.azure.com/EasyWayData` | L'organizzazione Azure DevOps |
| **ADO Project** | `EasyWay-DataPortal` | Il progetto dentro l'org |
| **ado-auth.sh** | `easyway-ado/scripts/ado-auth.sh` | PAT routing + B64 + BOM-safe curl wrapper. Auto-detect env (server/local) |

## PAT Routing — Chi fa cosa (OBBLIGATORIO)

**MAI** usare un PAT generico (`ADO_PAT`) per tutto. Ogni azione ha il suo PAT con scope minimo (principio least privilege). Lo script `ado-auth.sh` gestisce il routing automaticamente.

| Azione | PAT Variable | Service Account | Scope ADO |
|--------|-------------|-----------------|-----------|
| PR create/read/complete | `ADO_PR_CREATOR_PAT` | svc-agent-pr-creator | Code R/W, PR Contribute |
| Work Item read/write/link | `ADO_WORKITEMS_PAT` | svc-agent-scrum-master | Work Items R/W, Iterations R |
| Team settings/iterations | `ADO_WORKITEMS_PAT` | svc-agent-scrum-master | Team Admin role required |
| Pipeline/Build | `AZURE_DEVOPS_EXT_PAT` | ew-svc-azuredevops-agent | Code R/W, Build R/Execute |
| General read (briefing) | `AZURE_DEVOPS_EXT_PAT` | ew-svc-azuredevops-agent | General read |
| GitHub mirror/push | `GITHUB_PAT` | ADO-GitHub-Mirror | GitHub org push |

**Dove vivono i PAT**:
- **Server**: `/opt/easyway/.env.secrets` (tutte le variabili sopra)
- **Dev locale**: `C:\old\.env.local` (se presenti, altrimenti servono sul server)
- **Auto-detect**: `ado-auth.sh` cerca automaticamente: env var `ADO_AUTH_ENV_FILE` > server > locale

### Come usare ado-auth.sh (3 modi)

```bash
# 1. Auth only — genera il Base64 header per curl
B64=$(bash ado-auth.sh wi) && curl -H "Authorization: Basic $B64" ...
B64=$(bash ado-auth.sh pr) && curl -H "Authorization: Basic $B64" ...

# 2. ado-curl (raccomandato) — auth + BOM stripping + Content-Type auto
source ado-auth.sh
ado-curl wi POST "https://dev.azure.com/EasyWayData/..." -d '[...]'
ado-curl pr GET  "https://dev.azure.com/EasyWayData/..."

# 3. Sul server con env gia caricato
source /opt/easyway/.env.secrets
source ~/easyway-ado/scripts/ado-auth.sh  # skip file load, usa env
ado-curl wi POST "..." -d '[...]'
```

### Troubleshooting PAT

| Errore | Causa | Soluzione |
|--------|-------|-----------|
| HTTP 401 / content-length: 0 | PAT sbagliato per l'azione | Verifica la tabella routing sopra |
| `unbound variable` | PAT non presente nel file env | Aggiungere la variabile al file env corretto |
| HTTP 203 (redirect HTML) | PAT scaduto | Rinnovare su ADO > User Settings > PAT |
| JSON parse error | BOM in risposta | Usare `ado-curl` invece di `curl` diretto |

## Metodo di approccio: MCP > CLI > Connettori > curl

Quando devi interagire con ADO, usa questo ordine di preferenza:

### 1. MCP Server (preferito - zero auth manuale)

Se il tuo IDE ha il MCP configurato, hai 11 tool pronti:

| Tool | Cosa fa | Esempio |
|------|---------|---------|
| `ado_briefing` | Stato progetto: PR attive, PBI aperti, sprint, repo | - |
| `ado_wi_get` | Dettaglio work item per ID | `id: 97` |
| `ado_wi_list` | Lista WI attivi, filtrabile per tipo | `type: "Product Backlog Item"` |
| `ado_wi_query` | Query WIQL libera | `wiql: "SELECT ... FROM workitems WHERE ..."` |
| `ado_wi_create` | Crea WI (PBI, Feature, Epic) | `type: "Product Backlog Item", title: "..."` |
| `ado_wi_update` | Aggiorna WI (stato, titolo, campi) | `id: 97, state: "Done"` |
| `ado_pr_list` | Lista PR per stato | `status: "active"` |
| `ado_pr_get` | Dettaglio PR con WI linkati | `id: 346` |
| `ado_pr_create` | Crea PR (con Palumbo enforcement: serve WI) | `repo, source, title, workItemId` |
| `ado_pr_policy` | Check policy e reviewer votes | `id: 346` |
| `ado_rag_resolve` | Ricerca semantica sulla wiki | `query: "come si fa deploy?"` |

### 2. CLI (se MCP non disponibile)

```bash
cd C:\old\easyway\ado

# Briefing
npx tsx bin/easyway-ado.ts briefing

# Work Items
npx tsx bin/easyway-ado.ts wi list
npx tsx bin/easyway-ado.ts wi get 97
npx tsx bin/easyway-ado.ts wi create "Product Backlog Item" "Titolo del PBI"
npx tsx bin/easyway-ado.ts wi update 97 --state Done

# Pull Requests
npx tsx bin/easyway-ado.ts pr list
npx tsx bin/easyway-ado.ts pr create easyway-wiki feat/my-branch --wi 97
npx tsx bin/easyway-ado.ts pr policy 343
```

Il CLI trova il PAT automaticamente in questo ordine:
1. Variabile d'ambiente `ADO_PAT`
2. File `.env` nella directory corrente
3. `C:\old\.env.local` (Windows) / `~/.env.local` (Linux)

### 3. Connettori bash (auth gia integrata, best practice per scripting)

I connettori in `easyway-agents/scripts/connections/` gestiscono auth, PAT routing e error handling.
Sono la **best practice** per operazioni da script, cron, pipeline o quando il CLI non funziona.

```bash
# Path connettori
CONN_DIR="C:\old\easyway\agents\scripts\connections"  # Windows
CONN_DIR="~/easyway-agents/scripts/connections"         # Server

# Auth header (PAT routing automatico per azione)
B64=$(bash /c/old/easyway/agents/scripts/ado-auth.sh wi)

# Operazioni comuni con connettore + curl
# Get work item
curl -s "https://dev.azure.com/EasyWayData/EasyWay-DataPortal/_apis/wit/workitems/97?api-version=7.1" \
  -H "Authorization: Basic $B64"

# Update work item state
curl -s "https://dev.azure.com/EasyWayData/EasyWay-DataPortal/_apis/wit/workitems/97?api-version=7.1" \
  -H "Authorization: Basic $B64" \
  -H "Content-Type: application/json-patch+json" \
  -X PATCH \
  -d '[{"op":"replace","path":"/fields/System.State","value":"Active"}]'

# Connettori specializzati
bash connections/ado.sh healthcheck       # Verifica auth
bash connections/ado.sh wi-get 97         # Get WI
bash connections/github.sh pr-list org/repo  # Lista PR GitHub
bash connections/qdrant.sh search "query"    # Ricerca semantica
bash connections/server.sh exec "cmd"        # Comando su server
```

**Perche connettori > curl diretto**: il connettore `ado-auth.sh` seleziona automaticamente il PAT giusto
per l'azione richiesta (wi, pr, build, github) usando il PAT router. Non devi sapere quale PAT usare.

### 4. curl diretto (ultimo resort)

Solo se connettori, CLI e MCP non sono disponibili:

```bash
# Auth manuale (SOLO se ado-auth.sh non disponibile)
PAT="il-tuo-pat"
B64=$(printf ":%s" "$PAT" | base64 -w0)

# Lista PBI attivi
curl -s "https://dev.azure.com/EasyWayData/EasyWay-DataPortal/_apis/wit/wiql?api-version=7.1" \
  -H "Authorization: Basic $B64" \
  -H "Content-Type: application/json" \
  -d '{"query": "SELECT [System.Id] FROM workitems WHERE [System.WorkItemType] = '\''Product Backlog Item'\'' AND [System.State] <> '\''Done'\'' AND [System.State] <> '\''Removed'\''"}'
```

**ATTENZIONE con curl diretto**: quoting fragile (specialmente in PowerShell), rate limit, nessuna validazione, PAT hardcoded. Usalo solo se non hai alternative.

## Regole operative (NON negoziabili)

1. **MAI creare PR senza Work Item** (Regola del Palumbo) - prima crea/trova il WI, poi la PR
2. **MAI approvare/mergiare PR** senza ok esplicito dell'utente umano
3. **MAI pushare direttamente a main** - sempre feature branch + PR
4. **Portal: feat -> develop -> main** (3 livelli). Tutti gli altri repo: feat -> main (2 livelli)
5. **Nomi repo nell'API**: usare `easyway-wiki`, `easyway-agents`, etc. MAI usare GUID
6. **Merge strategy**: sempre no-fast-forward su branch protetti
7. **MAX 2 tentativi API** per la stessa azione - se fallisce, fermati e chiedi aiuto

## Golden Path: da zero a PR (flusso completo per agenti autonomi)

Questo e' il flusso che OGNI agente deve seguire. Non inventare script, usa questi tool.

### Fase 0: Setup (una volta per sessione)

```bash
# Assicurati di essere nel repo giusto
cd C:/old/easyway/portal    # o wiki, agents, infra, ado

# Allinea al branch target
git fetch origin develop && git checkout develop && git pull  # portal
git fetch origin main && git checkout main && git pull        # tutti gli altri

# Crea feature branch
git checkout -b feat/nome-descrittivo
```

### Fase 1: Lavora (codice, CSS, docs...)

Fai le modifiche. Poi commit con ewctl:

```powershell
# ewctl e' PowerShell (NON bash). Chiamalo cosi:
pwsh -Command "cd C:/old/easyway/portal; Import-Module C:/old/easyway/agents/tools/ewctl/ewctl.psd1; ewctl commit"
```

Se ewctl non funziona, usa git diretto (Iron Dome gira come pre-commit hook):

```bash
cd C:/old/easyway/portal
git add -A
git commit -m "feat(ui): descrizione della modifica"
```

### Fase 2: Crea PBI (Regola del Palumbo — PRIMA del push)

```bash
# Connettore bash (gestisce $ nell'URL e auth automatica)
bash C:/old/easyway/agents/scripts/connections/ado.sh wi-create pbi "Titolo del PBI" --tags "UI; S84"

# Output: Created #103 [Product Backlog Item] - Titolo del PBI
```

NON usare curl diretto per creare WI — il `$` nel tipo viene mangiato dalla shell.
NON usare `Import-AgentSecrets` — carica secrets infra, non PAT ADO.

### Fase 3: Push + PR

```bash
# Push
git push -u origin feat/nome-descrittivo

# PR con curl (specificare repo, source, target, WI)
B64=$(bash C:/old/easyway/agents/scripts/ado-auth.sh pr)

# Per portal: feat -> develop
curl -s -X POST \
  "https://dev.azure.com/EasyWayData/EasyWay-DataPortal/_apis/git/repositories/easyway-portal/pullrequests?api-version=7.1" \
  -H "Authorization: Basic $B64" \
  -H "Content-Type: application/json" \
  -d '{"sourceRefName":"refs/heads/feat/nome-descrittivo","targetRefName":"refs/heads/develop","title":"Titolo PR","description":"PBI #103","workItemRefs":[{"id":"103"}]}'

# Per wiki/agents/infra/ado: feat -> main
curl -s -X POST \
  "https://dev.azure.com/EasyWayData/EasyWay-DataPortal/_apis/git/repositories/easyway-wiki/pullrequests?api-version=7.1" \
  -H "Authorization: Basic $B64" \
  -H "Content-Type: application/json" \
  -d '{"sourceRefName":"refs/heads/feat/nome-descrittivo","targetRefName":"refs/heads/main","title":"Titolo PR","description":"PBI #103","workItemRefs":[{"id":"103"}]}'
```

### Fase 4: STOP — non andare oltre

Dopo aver creato la PR, FERMATI. NON:
- Approvare la PR (serve ok umano)
- Mergiare la PR (serve ok umano)
- Completare la PR (serve ok umano)

Comunica all'utente: PR creata, PBI linkato, pronta per review.

## Ricette rapide

```bash
# Briefing progetto
bash C:/old/easyway/agents/scripts/connections/ado.sh healthcheck
npx --prefix C:/old/easyway/ado tsx C:/old/easyway/ado/bin/easyway-ado.ts briefing

# Lista PBI aperti
bash C:/old/easyway/agents/scripts/connections/ado.sh wi-search "UI"

# Stati validi per un tipo WI (evita errori 400)
bash C:/old/easyway/agents/scripts/connections/ado.sh wi-states pbi

# Aggiorna stato WI
bash C:/old/easyway/agents/scripts/connections/ado.sh wi-update 103 --state Done

# Get WI dettaglio
bash C:/old/easyway/agents/scripts/connections/ado.sh wi-get 103
```

## Troubleshooting

| Problema | Causa | Soluzione |
|----------|-------|-----------|
| `401 Unauthorized` | PAT scaduto o mancante | Verifica `C:\old\.env.local` contiene `ADO_PAT=...` valido |
| `TF401019` | GUID troncato nell'URL | Usa il **nome** del repo (`easyway-wiki`), non il GUID |
| `Palumbo enforcement` | PR senza WI | Crea prima il WI con `wi create`, poi passa `--wi <id>` |
| MCP non risponde | Server non buildato | `cd easyway/ado && npm install && npm run build` |
| PowerShell quoting rotto | Caratteri speciali nel body | Usa il CLI invece di curl. Sul serio. |

## Per agenti esterni (Gemini, Copilot, o qualsiasi AI da directory esterna)

Se lavori da una directory diversa (es. `~/.gemini/antigravity/scratch`), devi sapere 3 cose:

### 1. Path: SEMPRE forward slash con bash

```bash
# SBAGLIATO - backslash vengono mangiati da bash
bash C:\old\easyway\agents\scripts\ado-auth.sh wi        # -> "No such file"

# CORRETTO - forward slash
bash "C:/old/easyway/agents/scripts/ado-auth.sh" wi      # -> funziona
bash /c/old/easyway/agents/scripts/ado-auth.sh wi        # -> funziona (Git Bash)
bash /mnt/c/old/easyway/agents/scripts/ado-auth.sh wi    # -> funziona (WSL)
```

### 2. Git Bash vs WSL: mount diversi

| Shell | Mount di `C:\` | Esempio path |
|-------|---------------|-------------|
| **Git Bash / MSYS** | `/c/` | `/c/old/.env.local` |
| **WSL** | `/mnt/c/` | `/mnt/c/old/.env.local` |
| **PowerShell** | `C:\` o `C:/` | `C:/old/.env.local` |

Gli script `ado-auth.sh` e `_common.sh` cercano automaticamente in tutti e 3 i mount.
Per forzare un root specifico: `export EASYWAY_ROOT=/mnt/c/old`.

### 3. Env var EASYWAY_ROOT (opzionale, raccomandato)

Se il tuo ambiente non trova automaticamente i file, imposta `EASYWAY_ROOT`:

```bash
# In WSL
export EASYWAY_ROOT=/mnt/c/old

# In Git Bash
export EASYWAY_ROOT=/c/old

# Poi tutto funziona normalmente
B64=$(bash "$EASYWAY_ROOT/easyway/agents/scripts/ado-auth.sh" wi)
bash "$EASYWAY_ROOT/easyway/agents/scripts/connections/ado.sh" healthcheck
```

### 4. CLI: il metodo piu sicuro da qualsiasi directory

```bash
# Opzione A: cd prima
cd C:/old/easyway/ado && npx tsx bin/easyway-ado.ts briefing

# Opzione B: npx con prefix
npx --prefix C:/old/easyway/ado tsx C:/old/easyway/ado/bin/easyway-ado.ts briefing

# Opzione C: imposta il PAT esplicitamente
export ADO_PAT=$(grep ADO_PAT C:/old/.env.local | cut -d= -f2)
```

### 5. Import-AgentSecrets (PowerShell) NON carica i PAT ADO

`Import-AgentSecrets.ps1` carica i secrets infra (Deepseek, Gitea, Qdrant) da `/opt/easyway/.env.secrets`.
I PAT ADO stanno in `C:\old\.env.local` — file diverso. Per i PAT ADO usa `ado-auth.sh` o il CLI.

## Link utili

- [README easyway-ado](../../ado/README.md)
- [Connection Registry](connection-registry.md)
- [Polyrepo Git Workflow](polyrepo-git-workflow.md)
- [ADO PR Structural Validation](ado-pr-structural-validation.md)
