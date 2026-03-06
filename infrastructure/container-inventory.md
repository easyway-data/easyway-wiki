---
id: ew-infra-container-inventory
title: Container Inventory — Chi fa cosa sul server
summary: Mappa completa dei container Docker sul server Oracle ARM, con ruolo, immagine, rete e stato. Aggiornare ad ogni modifica infrastrutturale.
status: active
owner: team-platform
created: '2026-03-06'
updated: '2026-03-06'  # Session 93 - Qdrant v1.16.2 upgrade + naming fix
tags: [domain/infrastructure, layer/reference, audience/ops, audience/dev, privacy/internal, language/it, docker, containers, server]
entities: []
llm:
  include: true
  pii: none
  chunk_hint: 300-500
type: reference
---

# Container Inventory — Chi fa cosa sul server

> **Server**: Oracle ARM (Ubuntu), IP `80.225.86.168`
> **Compose dir**: `~/easyway-infra/`
> **Compose files**: `docker-compose.yml` (base) + `docker-compose.apps.yml` + `docker-compose.prod.yml`
> **Compose project**: `easyway-prod`
> **Ultimo aggiornamento**: 2026-03-06 (Session 85)

---

## Progetto `easyway-prod` — Stack Principale

Tutti gestiti da docker compose. Avvio: `cd ~/easyway-infra && docker compose -f docker-compose.yml -f docker-compose.apps.yml -f docker-compose.prod.yml -p easyway-prod up -d`

| Container | Service | Immagine | Ruolo | Porte | Rete | Note |
|---|---|---|---|---|---|---|
| `easyway-portal` | `frontend` | `easyway/frontend:latest` | Frontend PWA (nginx) — sito pubblico, landing page, dashboard | 8080 | easyway-net | Alias DNS: `frontend`. Caddy proxya qui |
| `easyway-api` | `api` | `easyway-prod-api` | Backend API Node.js — gestione utenti, appuntamenti, quote | 3000 | easyway-net | DB mode: sql (SQL Edge x86) o mock (ARM) |
| `easyway-runner` | `agent-runner` | `easyway-prod-agent-runner` | Agent framework — 31+ agenti AI, skills, ewctl | — | easyway-net | ~3.8 GB immagine. Contiene agents + wiki mount |
| `easyway-gateway-caddy` | `caddy` | `caddy:2.8-alpine` | Reverse proxy principale — routing HTTP/HTTPS, basic auth, compressione | 80, 443, 2019 | easyway-net | Caddyfile in `/etc/caddy/Caddyfile`. Admin API su 2019 |
| `easyway-orchestrator` | `n8n` | `n8nio/n8n:1.123.20` | Workflow automation — webhook ADO, notifiche, orchestrazione agenti | 5678 | easyway-net | Backend PostgreSQL. Accessible via `/n8n/` (Caddy) |
| `easyway-memory` | `qdrant` | `qdrant/qdrant:v1.16.2` | Vector database — RAG wiki, knowledge base agenti | 6333 (internal) | easyway-net | API key required. ~168K chunks. Upgraded S93: v1.9.0/v1.12.4 non supportavano storage format `InRamChunkedMmap` |
| `easyway-meta-db` | `postgres` | `postgres:15.10-alpine` | Database relazionale — backend per n8n workflows | — | easyway-net | User: n8n, DB: n8n |
| `easyway-storage` | `azurite` | `azurite:3.29.0` | Azure Blob Storage emulator — storage documenti | 10000-10002 | easyway-net | Usato da agent-runner |
| `easyway-storage-s3` | `minio` | `minio:RELEASE.2024-01-31` | Object storage S3-compatible — backup, file sovereign | 9000, 9001 | easyway-net | Console su :9001. Unhealthy (non critico) |

### Container in standby (profili)

| Container | Service | Profilo | Note |
|---|---|---|---|
| `easyway-gateway-traefik` | `traefik` | `backup` | Reverse proxy backup su porta 8081. Avvio: `--profile backup` |
| `easyway-db` | `sql-edge` | `sql` | SQL Edge per x86 locale. Non disponibile su ARM64 |

---

## Progetto `forgejo` — Git Server Interno

Compose file: `~/EasyWayDataPortal/infra/forgejo/docker-compose.yml` (path legacy, da migrare)

| Container | Service | Immagine | Ruolo | Porte | Rete |
|---|---|---|---|---|---|
| `easyway-atelier` | `atelier` | `forgejo:9.0` | Git forge self-hosted — repo interni, code review | — | forgejo_atelier-net |

---

## Container Orfani — PULITI (S86)

Tutti i container orfani sono stati rimossi nella Session 86:
- 5 container `easyway-cert-*` (progetto monorepo legacy)
- 1 container `easyway-seq` (Seq log viewer — nessun log in ingresso, inutilizzato)
- 6 immagini orfane rimosse, 33 volumi orfani rimossi, 10.78 GB build cache pulita
- **Totale recuperato: ~16 GB**

---

## Volumi — Stato

### Attivi (usati da easyway-prod)
- `easyway-prod_qdrant-data` — dati vettoriali RAG (~168K chunks)
- `easyway-prod_postgres-data` — database n8n
- `easyway-prod_n8n-data` — workflow n8n
- `easyway-prod_caddy_data/config/logs` — certificati e log Caddy
- `easyway-prod_minio-data` — object storage
- `easyway-prod_azurite-data` — blob storage

### Orfani — PULITI (S86)
Tutti i 33 volumi orfani sono stati rimossi (cert, dev, monorepo legacy, snapshot, forgejo duplicati, anonimi).

---

## Network

| Nome | Driver | Usata da |
|---|---|---|
| `easyway-net` | bridge | Tutti i container easyway-prod |
| `easyway-cert_easyway-net` | bridge | Container cert legacy (da rimuovere) |
| `easyway-cert_easyway-cert-net` | bridge | Container cert legacy (da rimuovere) |
| `forgejo_atelier-net` | bridge | Forgejo |
| `forgejo_default` | bridge | Forgejo (auto) |
| `ubuntu_gitlab` | bridge | Legacy GitLab (da rimuovere) |

---

## Come aggiornare questa pagina

Dopo ogni modifica ai container (deploy, aggiunta servizio, rimozione), aggiornare questa pagina con il comando:

```bash
# SSH sul server, poi:
docker ps -a --format "table {{.Names}}\t{{.Status}}\t{{.Image}}" | sort
docker volume ls
docker network ls
```

---

## Best Practice — Prevenire il Volume Sprawl

### Incidente S85: 33 volumi orfani, ~1.5 GB sprecati

Ogni volta che si cambia il **project name** (`-p`) o la **working directory** di compose, Docker crea un nuovo set di volumi con prefisso diverso senza eliminare i vecchi. Esempio: `easywaydataportal_qdrant-data` → `easyway_qdrant-data` → `easyway-prod_qdrant-data` = 3 copie degli stessi dati.

### Regole

1. **MAI cambiare il project name** senza pulire prima i vecchi volumi:
   ```bash
   # PRIMA di cambiare -p o working dir:
   docker compose -p <vecchio-progetto> down  # NON -v se vuoi tenere i dati
   docker volume ls | grep <vecchio-prefisso>  # verifica cosa resta
   docker volume rm <vecchio-prefisso>_*       # pulisci
   ```

2. **MAI creare container con `docker run`** — usare SEMPRE compose. Un container creato con `docker run` non e tracciato, non ha labels compose, e genera volumi anonimi.

3. **Censire ogni container** in questa pagina. Se un container non e qui, non dovrebbe esistere.

4. **Pulizia periodica** — controllare mensilmente:
   ```bash
   # Volumi orfani
   docker volume ls -f dangling=true
   # Immagini inutilizzate
   docker image prune -a --dry-run
   # Build cache
   docker builder prune --dry-run
   ```

5. **n8n watchdog** (backlog S85): workflow automatico che confronta `docker ps -a` con la whitelist censita e segnala anomalie.

### Stato risorse server (aggiornato S86)

| Risorsa | Valore |
|---|---|
| Disco totale | 193 GB |
| Disco usato | 64 GB (33%) |
| Disco libero | **130 GB** |
| RAM totale | 24 GB |
| RAM disponibile | **20 GB** |
| Container attivi | 10 |
| Volumi attivi | 9 |
| Immagini Docker | 11 (9.4 GB, zero orfane) |

**Pulizia S86**: rimossi 5 container orfani, 33 volumi, 6 immagini, 10.78 GB build cache. Totale recuperato: ~16 GB.

---

## Vedi anche

- [Security Framework (RBAC)](../infra/security-framework.md) — chi puo gestire i container
- [Threat Analysis](../security/threat-analysis-hardening.md) — container security hardening
- [Deploy Workflow](../guides/polyrepo-git-workflow.md) — come deployare correttamente
