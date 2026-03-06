# Connection Registry

Central registry of all external service connections used by the EasyWay + HALE-BOPP platform.
Inspired by `odbc.ini` — a single place to know what connects where, with what credentials, and how.

## Location

```
easyway-agents/scripts/connections/
  connections.yaml      # declarative registry (machine-readable)
  github.sh             # GitHub API connector
  ado.sh                # Azure DevOps connector
  server.sh             # OCI server SSH connector
  qdrant.sh             # Qdrant vector store connector
  halebopp.sh           # HALE-BOPP services connector (via SSH)
  healthcheck-all.sh    # verify all connections
  README.md             # usage documentation
```

## Registered Connections

| Name          | Type     | Base URL / Host                       | Auth Method | Connector     |
|---------------|----------|---------------------------------------|-------------|---------------|
| **GitHub**    | REST API | `https://api.github.com`              | PAT token   | `github.sh`   |
| **ADO**       | REST API | `https://dev.azure.com/EasyWayData`   | PAT B64     | `ado.sh`      |
| **Qdrant**    | REST API | `http://localhost:6333`               | API key     | `qdrant.sh`   |
| **Server**    | SSH      | `80.225.86.168`                       | SSH key     | `server.sh`   |
| **HALE-BOPP** | REST API | `http://127.0.0.1:{8100,3001,8200}`   | None (SSH)  | `halebopp.sh` |
| **OpenRouter**| REST API | `https://openrouter.ai/api/v1`        | Bearer      | _(planned)_   |

## Secrets Management

All secrets live in `/c/old/.env.local` (never committed to any repository).
Connectors use a robust KEY=VALUE parser that handles Unicode comments safely.

## Standard Interface

Every connector follows the **Electrical Socket Pattern** (GEDI principle):
same interface, swap the provider.

```bash
# Every connector has healthcheck
bash connections/github.sh healthcheck    # -> OK (HTTP 200)
bash connections/ado.sh healthcheck       # -> OK (HTTP 200)
bash connections/server.sh healthcheck    # -> OK -- 03:20:45 up 38 days...
bash connections/qdrant.sh healthcheck    # -> OK (HTTP 200)
bash connections/halebopp.sh healthcheck # -> db (:8100) — OK ...

# Check all at once
bash connections/healthcheck-all.sh
```

## GitHub Connector Examples

```bash
# Authentication test
bash connections/github.sh whoami

# PR operations
bash connections/github.sh pr-list   hale-bopp-data/hale-bopp-etl
bash connections/github.sh pr-status hale-bopp-data/hale-bopp-etl 1
bash connections/github.sh pr-merge  hale-bopp-data/hale-bopp-etl 1
```

## ADO Connector Examples

```bash
# PR and Work Items
bash connections/ado.sh pr-list
bash connections/ado.sh wi-get 60

# Raw API call
bash connections/ado.sh api GET "EasyWay-DataPortal/_apis/git/repositories?api-version=7.1"
```

## Server Connector Examples

```bash
# Run command on server
bash connections/server.sh exec "df -h /"
bash connections/server.sh disk
bash connections/server.sh docker
bash connections/server.sh services
```

## HALE-BOPP Connector Examples

```bash
# Check all 3 services (db, etl, argos) via SSH
bash connections/halebopp.sh healthcheck

# Check single service
bash connections/halebopp.sh health db
bash connections/halebopp.sh health etl
bash connections/halebopp.sh health argos

# Schema operations (via db :8100 API)
bash connections/halebopp.sh diff "postgresql://user:pass@localhost/mydb" desired.json
bash connections/halebopp.sh snapshot "postgresql://user:pass@localhost/mydb"
```

## ADO PAT Routing

The ADO connector delegates authentication to `ado-auth.sh`, which routes
to the correct PAT based on the action scope (principle of least privilege):

| Action    | PAT Variable            | Scope                          |
|-----------|-------------------------|--------------------------------|
| `pr`      | `ADO_PR_CREATOR_PAT`    | Code R/W + PR Contribute       |
| `wi`      | `ADO_WORKITEMS_PAT`     | Work Items R/W                 |
| `build`   | `AZURE_DEVOPS_EXT_PAT`  | Code R/W + Build Read          |
| `general` | `AZURE_DEVOPS_EXT_PAT`  | Briefing, branch list          |

## PAT Status & Credentials

All PATs live in `C:\old\.env.local` (Windows dev) or `/opt/easyway/.env.secrets` (server).
**Mai committare. Mai hardcodare.**

| Variable | Scopo | Stato (S88) | Note |
|----------|-------|-------------|------|
| `AZURE_DEVOPS_EXT_PAT` | General (build, branch list, briefing) | **OK** | Verificato S88 — 200 su projects, code, build, policy |
| `ADO_PR_CREATOR_PAT` | PR create/update (Code R/W + PR Contribute) | OK | Usato da `ado-auth.sh pr` |
| `ADO_WORKITEMS_PAT` | Work Items R/W | OK | Usato da `ado-auth.sh wi` |
| `AZURE_DEVOPS_WI_PAT` | Alias legacy per Work Items | OK | Stesso scope di `ADO_WORKITEMS_PAT` |
| `GITHUB_PAT` | GitHub API (mirror, PR, orgs) | OK | Scadenza Jun 2 2026 |

**Alias**: `ADO_PAT` = alias di `AZURE_DEVOPS_EXT_PAT` (usato da agent scripts con `$env:ADO_PAT`).

**Come rinnovare un PAT ADO**:
1. `dev.azure.com/EasyWayData` > User Settings > Personal Access Tokens
2. Trovare il token, rigenerare con stessi scope
3. Aggiornare `C:\old\.env.local` con il nuovo valore
4. Se usato su server: aggiornare anche `/opt/easyway/.env.secrets`
5. Se usato in pipeline: aggiornare Variable Group `EasyWay-Secrets` in ADO

**Dettaglio scope per PAT**: vedi `memory/ado-governance.md` nel repo memory.

## Environment Overlays (S82)

All connectors use `_common.sh` for shared helpers (`_load_env`, `_log_error`, `_classify_error`).

**Auto-detect env file** (order of precedence):
1. `CONN_ENV_FILE` explicit override
2. `/c/old/.env.local` (Windows dev)
3. `$HOME/.env.local` (Linux user)
4. `/opt/easyway/.env.secrets` (server system)

**Overlay mechanism**: set `CONN_ENV=<env>` to load `connections.<env>.env` that overrides base values.

```bash
# Run qdrant.sh with server-specific URLs
CONN_ENV=server bash connections/qdrant.sh healthcheck

# connections.server.env contains:
# QDRANT_URL=http://localhost:6333
# RAG_HTTP_PORT=8300
```

Available overlays:
- `connections.server.env` — server-local URLs (Qdrant, RAG search)

Create new overlays by adding `connections.<env>.env` files.

## Future Vision

The `connections.yaml` registry is designed to be consumed by:
- **n8n workflows**: auto-discover available connections
- **Agents**: read registry to know which APIs are available
- **CLI tools**: `ewctl connect <name>` to test/use connections
- **Monitoring**: scheduled healthcheck-all for uptime tracking
- **Cursor/Claude**: `.cursorrules` auto-generated from wiki via n8n (planned)
