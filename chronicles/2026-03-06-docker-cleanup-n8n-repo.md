---
title: "Session 86 — La Grande Pulizia e il Nono Repo"
date: 2026-03-06
category: infrastructure
session: S86
tags: [domain/docker, process/cleanup, domain/n8n, easyway-n8n, domain/infrastructure, domain/monitoring]
---

# Session 86 — La Grande Pulizia e il Nono Repo

> "Prima di costruire, sgomberare il cantiere."

## Il Problema

Un collega aveva bypassato il deploy workflow usando SCP per caricare file direttamente sul server. L'indagine conseguente ha rivelato uno stato del server ben peggiore del previsto: 16 container (di cui 6 orfani), 42 volumi (33 orfani), 17 immagini (6 inutilizzate), 10.78 GB di build cache. Il server era un cantiere mai pulito dopo mesi di rinominazioni compose (easywaydataportal → easyway → easyway-cert → easyway-dev → easyway-infra → easyway-prod).

## La Pulizia

Methodicamente, partendo dal meno rischioso:
1. **5 container zombie** `easyway-cert-*` — resti del vecchio progetto monorepo
2. **33 volumi orfani** — accumulati da 6 cambi di project name compose
3. **6 immagini orfane** — vecchie versioni (qdrant v1.9, n8n 1.31, agent-runner cert/dev/infra)
4. **10.78 GB build cache** — layer intermedi mai puliti
5. **easyway-seq** — Seq log viewer che girava da 6 giorni senza ricevere un singolo log

Risultato: da 80 GB usati (42%) a **64 GB (33%)**. 16 GB recuperati. Da 42 volumi a 9. Da 16 container a 10.

## Il Nono Repo

Con la pulizia completata, l'attenzione si e spostata sulla prevenzione. Due workflow n8n per monitorare il server:

- **docker-health-report**: report giornaliero (disco, RAM, container status, volumi orfani, build cache). Alert se soglie superate.
- **container-census-watchdog**: confronta container attivi con whitelist censita. Segnala intrusi.

Con 8 workflow esistenti + 2 nuovi = 10 — abbastanza per giustificare un repo dedicato. Nasce **easyway-n8n**, il nono repository dell'ecosistema:

```
easyway-n8n/
  workflows/
    common/     → Error Handler, Send Email
    business/   → MinIO ingest, Sentinel
    infra/      → Health Report, Census Watchdog  (NEW)
    templates/  → Master, Pipeline Parent, Agent Composition
  scripts/      → validate-workflows.sh
```

Circle 3, private ADO, PR #357 merged, PBI #105, factory.yml aggiornato a 9 repo.

## Lezione

Il volume sprawl non fa rumore. 33 volumi orfani si accumulano silenziosamente perche ogni `docker compose -p <nuovo-nome> up` crea un nuovo set senza rimuovere il vecchio. La regola: prima di cambiare project name, sempre `docker compose -p <vecchio-nome> down -v`. E per il futuro, il watchdog n8n preverra che succeda di nuovo.

## Numeri Finali

| Metrica | Prima | Dopo |
|---------|-------|------|
| Disco usato | 80 GB (42%) | 64 GB (33%) |
| Container | 16 | 10 |
| Volumi | 42 | 9 |
| Immagini | 17 | 10 |
| Repository | 8 | 9 |
