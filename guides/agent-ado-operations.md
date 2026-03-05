# Agent ADO Operations Guide

> How any AI agent (Claude, Cursor, Gemini, Copilot) interacts with Azure DevOps in the EasyWay project.
> Zero prerequisites scontati. Se non sai dove partire, parti da qui.

## Cosa c'e' e dove

| Cosa | Dove | A cosa serve |
|------|------|-------------|
| **easyway-ado** | `C:\old\easyway\ado\` (Windows) / `~/easyway-ado/` (server) | CLI + MCP Server per Azure DevOps |
| **PAT (Personal Access Token)** | `C:\old\.env.local` | Autenticazione verso ADO. Variabile: `ADO_PAT` |
| **MCP config Claude Code** | `C:\old\.mcp.json` | Registra easyway-ado come MCP server |
| **MCP config Cursor** | `C:\old\.cursor\mcp.json` | Registra easyway-ado come MCP server |
| **ADO Organization** | `https://dev.azure.com/EasyWayData` | L'organizzazione Azure DevOps |
| **ADO Project** | `EasyWay-DataPortal` | Il progetto dentro l'org |
| **ado-auth.sh** | `easyway/agents/scripts/ado-auth.sh` | Script bash per generare header Base64 (fallback) |

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

## Ricette comuni

### Creare un PBI e poi una PR

```bash
# 1. Crea il PBI
npx tsx bin/easyway-ado.ts wi create "Product Backlog Item" "Descrizione del lavoro"
# Output: Created #103 - Descrizione del lavoro

# 2. Fai il lavoro sul feature branch
git checkout -b feat/pbi-103-descrizione
# ... modifiche ...
git add . && git commit -m "PBI-103: descrizione"
git push -u origin feat/pbi-103-descrizione

# 3. Crea la PR linkando il WI
npx tsx bin/easyway-ado.ts pr create easyway-wiki feat/pbi-103-descrizione --wi 103
```

### Controllare lo stato prima di lavorare

```bash
# Briefing completo
npx tsx bin/easyway-ado.ts briefing

# Solo PBI aperti
npx tsx bin/easyway-ado.ts wi list --type "Product Backlog Item"

# PR attive (ci sono conflitti? qualcuno sta lavorando sugli stessi file?)
npx tsx bin/easyway-ado.ts pr list
```

### Chiudere un PBI dopo il merge

```bash
npx tsx bin/easyway-ado.ts wi update 103 --state Done
```

## Troubleshooting

| Problema | Causa | Soluzione |
|----------|-------|-----------|
| `401 Unauthorized` | PAT scaduto o mancante | Verifica `C:\old\.env.local` contiene `ADO_PAT=...` valido |
| `TF401019` | GUID troncato nell'URL | Usa il **nome** del repo (`easyway-wiki`), non il GUID |
| `Palumbo enforcement` | PR senza WI | Crea prima il WI con `wi create`, poi passa `--wi <id>` |
| MCP non risponde | Server non buildato | `cd easyway/ado && npm install && npm run build` |
| PowerShell quoting rotto | Caratteri speciali nel body | Usa il CLI invece di curl. Sul serio. |

## Per Gemini / agenti che lavorano da directory esterne

Se lavori da una directory diversa (es. `~/.gemini/antigravity/scratch`):

```bash
# Opzione A: usa path assoluto
npx --prefix C:\old\easyway\ado tsx C:\old\easyway\ado\bin\easyway-ado.ts briefing

# Opzione B: cambia directory
cd C:\old\easyway\ado && npx tsx bin/easyway-ado.ts briefing

# Opzione C: imposta il PAT esplicitamente
export ADO_PAT=$(grep ADO_PAT C:\old\.env.local | cut -d= -f2)
npx --prefix C:\old\easyway\ado tsx C:\old\easyway\ado\bin\easyway-ado.ts wi list
```

## Link utili

- [README easyway-ado](../../ado/README.md)
- [Connection Registry](connection-registry.md)
- [Polyrepo Git Workflow](polyrepo-git-workflow.md)
- [ADO PR Structural Validation](ado-pr-structural-validation.md)
