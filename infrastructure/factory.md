---
id: ew-infra-factory
title: factory.yml — La Mappa dell'Ecosistema
summary: Spiegazione del file factory.yml che descrive tutti i repository, le relazioni, il layout server e la storia della migrazione polyrepo.
status: active
owner: team-platform
created: '2026-03-03'
updated: '2026-03-06'
tags: [domain/infrastructure, layer/reference, audience/dev, privacy/internal, language/it, easyway-infra, process/repos]
entities: []
llm:
  include: true
  pii: none
  chunk_hint: 300
type: reference
---

# factory.yml — La Mappa dell'Ecosistema

## Cos'e

`factory.yml` e il **manifesto infrastrutturale** di EasyWay: un singolo file YAML che descrive l'intero ecosistema di 9 repository, le loro relazioni, il layout sul server, lo stato dei mirror GitHub e la storia della migrazione da monorepo a polyrepo.

E la risposta alla domanda: *"quanti repo abbiamo, dove stanno, cosa contengono, come si deployano?"*

## Perche esiste

EasyWay e nato come un monorepo (`EasyWayDataPortal`) e tra le sessioni 53-62 e stato smembrato in 8 repo separati (poi 9 con easyway-n8n in S86). Senza una mappa centralizzata:

- Nessuno sapeva quanti repo c'erano realmente
- I path locali e server divergevano
- La migrazione procedeva senza un punto di riferimento unico
- Gli agenti AI non avevano modo di scoprire l'ecosistema

`factory.yml` risolve tutto questo. Un agente che legge questo file sa immediatamente dove trovare ogni componente, come deployarlo, e qual e la sua storia.

## Dove vive

| Posizione | Ruolo |
|-----------|-------|
| `easyway-wiki/infrastructure/factory.yml` | **Source of truth** — sotto version control |
| `C:\old\easyway\factory.yml` | Copia locale di lavoro (directory ombrello) |

La copia locale e comoda perche la directory `C:\old\easyway\` contiene tutti i repo come sottodirectory — avere `factory.yml` al livello superiore permette di navigare l'ecosistema.

## Cosa contiene

### 1. Sezione `factory`
Metadati del progetto: nome, filosofia, ID progetto ADO, URL org.

### 2. Sezione `repos`
Per ogni repository:
- **path**: dove si trova in locale (relativo a `C:\old\easyway\`)
- **remote**: URL ADO (o GitHub per HALE-BOPP)
- **ado_guid**: identificativo univoco del repo su ADO
- **description**: cosa fa
- **contains**: lista directory/file principali
- **deploy**: se il repo genera artefatti deployabili
- **deploy_target**: dove viene deployato (container, volume, server)
- **phase**: fase della migrazione polyrepo in cui e stato estratto

### 3. Sezione `server`
Layout del server Oracle ARM:
- Path base, secrets, SSH key, host
- Struttura directory su server
- Layout HALE-BOPP separato

### 4. Sezione `migration`
Storia completa della migrazione polyrepo:
- 4 fasi (0: Wiki, 1: Agents, 2: Infra, 3: Portal standalone)
- Sotto-fasi (3a: cleanup, 3b: CI/CD, 3c: rename+refs)
- Step completati con riferimenti a PR

### 5. Sezione `github_mirrors`
Stato di sincronizzazione tra ADO (source of truth) e GitHub (mirror):
- Visibilita (public/private)
- Stato sync (in_sync, behind, primary)

## I 9 Repository

| Repo | Cerchio | Ruolo |
|------|---------|-------|
| easyway-portal | 3 | Backend API + Frontend PWA |
| easyway-wiki | 2 | Documentazione, guide, RAG knowledge base |
| easyway-agents | 2 | 40+ agenti AI, skills, control plane |
| easyway-infra | 3 | Docker Compose, Caddy, CI/CD, deploy |
| easyway-ado | 2 | SDK TypeScript, CLI, MCP Server per ADO |
| easyway-n8n | 3 | Workflow n8n: orchestrazione e monitoring |
| hale-bopp-db | 1 | Schema governance PostgreSQL |
| hale-bopp-etl | 1 | Data orchestration YAML pipelines |
| hale-bopp-argos | 1 | Policy gating e data quality |

## Come si usa

**Per un agente AI**: leggere `factory.yml` per capire l'ecosistema, poi navigare al repo specifico.

**Per un umano**: consultare quando serve sapere "dove sta X?" o "come si deploya Y?".

**Per lo scrum master**: verificare che tutti i repo siano allineati e i deploy coerenti.

```bash
# Leggere la mappa
cat wiki/infrastructure/factory.yml

# Elenco repo con deploy target
grep -A2 "deploy_target" wiki/infrastructure/factory.yml
```

## Relazione con altri documenti

- **[repos/](../repos/_index.md)** — scheda operativa per-repo (tech stack, CI, connessioni)
- **[product-map.md](../vision/product-map.md)** — vista strategica (cosa estrarre, maturity)
- **[container-inventory.md](container-inventory.md)** — mappa container sul server
- **[connection-registry](../guides/connection-registry.md)** — connettori per interagire con i servizi

factory.yml e il livello "infrastruttura", repos/ il livello "operativo", product-map il livello "strategico".
