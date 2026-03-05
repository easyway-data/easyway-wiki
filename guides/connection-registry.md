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

## ADO PAT Routing

The ADO connector delegates authentication to `ado-auth.sh`, which routes
to the correct PAT based on the action scope (principle of least privilege):

| Action    | PAT Variable            | Scope                          |
|-----------|-------------------------|--------------------------------|
| `pr`      | `ADO_PR_CREATOR_PAT`    | Code R/W + PR Contribute       |
| `wi`      | `ADO_WORKITEMS_PAT`     | Work Items R/W                 |
| `build`   | `AZURE_DEVOPS_EXT_PAT`  | Code R/W + Build Read          |
| `general` | `AZURE_DEVOPS_EXT_PAT`  | Briefing, branch list          |

## Future Vision

The `connections.yaml` registry is designed to be consumed by:
- **n8n workflows**: auto-discover available connections
- **Agents**: read registry to know which APIs are available
- **CLI tools**: `ewctl connect <name>` to test/use connections
- **Monitoring**: scheduled healthcheck-all for uptime tracking
