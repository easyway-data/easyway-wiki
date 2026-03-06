---
id: repo-easyway-n8n
title: 'Repo: easyway-n8n'
summary: Scheda operativa del repository easyway-n8n — workflow n8n per orchestrazione, monitoring e automazione.
status: active
owner: team-platform
created: '2026-03-06'
updated: '2026-03-06'
tags: [easyway-n8n, repos, circle-3, domain/infra, layer/orchestration, audience/dev, privacy/internal, language/it]
entities: []
llm:
  include: true
  pii: none
  chunk_hint: 300
type: reference
---

# easyway-n8n

Workflow n8n per la piattaforma EasyWay. Orchestrazione, monitoring infrastruttura, automazione business.

## Anagrafica

| Campo | Valore |
|-------|--------|
| **ADO repo** | `easyway-n8n` (GUID: `ceed9f24-5a10-44d8-8147-8b7ed07a22bd`) |
| **GitHub** | Nessuno (Circle 3 — private) |
| **Cerchio** | 3 (Private ADO) |
| **Linguaggio** | JSON (n8n workflow format) |
| **CI/CD** | Da configurare (validate-workflows.sh) |
| **Branch protetti** | `main` |

## Struttura locale

```
C:\old\easyway\n8n\
```

## Componenti chiave

| Componente | Path | Descrizione |
|-----------|------|-------------|
| **Common** | `workflows/common/` | Workflow riusabili (Global Error Handler, Send Email) |
| **Business** | `workflows/business/` | Logica business (ingest MinIO, sentinel ingestion) |
| **Infra** | `workflows/infra/` | Monitoring Docker (health report, container census) |
| **Templates** | `workflows/templates/` | Pattern composizione (Master, Pipeline Parent, agent example) |
| **Scripts** | `scripts/` | Validazione JSON, import/export |

## Workflow attivi

| Workflow | Categoria | Descrizione |
|----------|-----------|-------------|
| `Global_Error_Handler` | common | Gestione errori globale |
| `Send_Email` | common | Invio email riusabile |
| `ingest-file-to-minio` | business | Ingest file verso MinIO storage |
| `sentinel-ingestion` | business | Ingestion dati sentinel |
| `docker-health-report` | infra | Report giornaliero salute Docker (disco, RAM, container, volumi) |
| `container-census-watchdog` | infra | Confronto container vs whitelist censita |

## Deploy

Volume mount nel container `easyway-orchestrator`:

```bash
# Sul server
cd ~/easyway-n8n && git fetch origin main && git reset --hard origin/main
```

Il container n8n monta `~/easyway-n8n/workflows` come `/home/node/workflows`.

## Connessioni

- **Server Docker**: Execute Command (richiede Docker socket montato o SSH)
- **MinIO**: `easyway-storage-s3` container (porta 9000)
- **ADO webhook**: variabile `WEBHOOK_ALERT_URL` in n8n environment

## Vedi anche

- [container-inventory](../infrastructure/container-inventory.md) — whitelist container usata dal watchdog
- [easyway-infra](easyway-infra.md) — Docker Compose che definisce il container n8n
