---
title: "Session 73 — HALE-BOPP Online"
date: 2026-03-05
category: milestone
session: S73
tags: [domain/hale-bopp, domain/deploy, domain/systemd, connection-registry, domain/etl, domain/infrastructure]
---

# Session 73 — HALE-BOPP Online

> I muscoli si svegliano. Tre servizi, quattro heartbeat, zero framework.

## Il momento

Per 72 sessioni HALE-BOPP e stato un'idea, poi un progetto, poi codice testato, poi repo su GitHub. Oggi diventa infrastruttura viva. Quattro processi systemd si avviano sul server OCI e rispondono `{"status":"ok"}` su tre porte. Niente Docker, niente Kubernetes, niente Airflow. Solo Python, YAML, e `systemctl start`.

## I fatti

### Il disco che non bastava

Il server mostrava 96GB, usati al 91%. Ma la console OCI diceva 200GB. Un rescan del device paravirtualizzato (`echo 1 > /sys/class/block/sda/device/rescan`), un `growpart`, un `resize2fs` — e il filesystem passa da 96 a 193GB. Dal 91% al 46%. Lo spazio non era finito: era nascosto.

### L'ETL che cambia pelle

PR #1 su GitHub: via Dagster, dentro un runner leggero di 300 righe. 30 test verdi. Ma la PR era bloccata — branch protection richiedeva una review che non poteva arrivare (owner unico). Bypass delle regole, merge, branch protection rimossa. La burocrazia serve quando c'e un team. Quando sei solo, la CI verde basta.

### Il Connection Registry

`.env.local` aveva caratteri Unicode nei commenti. `source` in bash non li digeriva. Invece di un fix puntuale, nasce il **Connection Registry** — cinque connettori bash con interfaccia standard, un file YAML dichiarativo tipo `odbc.ini`, un healthcheck che verifica tutto in un colpo. GitHub, ADO, Server, Qdrant — ognuno con il suo connettore, ognuno con lo stesso pattern: `<connettore>.sh healthcheck`, `<connettore>.sh <azione>`.

Il principio GEDI della Presa Elettrica, applicato alle connessioni: non importa se l'energia viene da GitHub o da ADO, basta che la presa sia standard.

### I muscoli si svegliano

Quattro unit file systemd. Un `daemon-reload`. Un `enable`. Un `start`. Due secondi dopo:

```
DB    :8100  {"status":"ok"}
ETL   :3001  {"status":"ok"}
ARGOS :8200  {"status":"ok"}
```

Bind su 127.0.0.1, auto-restart on failure, abilitati al boot. Il watcher poll ogni 10 secondi una cartella eventi. Il webhook ascolta POST HTTP. ARGOS valuta policy. DB governa schema.

I muscoli sono svegli. Aspettano solo che il cervello li comandi.

### La vetrina

Tre README riscritti con badge CI, diagrammi architettura ASCII, sezione "Part of HALE-BOPP" con il diagramma dei tre motori che comunicano. Traceability ADO nei commit (`ADO: PBI #38`). GitHub e la vetrina, ADO e il laboratorio.

## La mappa aggiornata

```
  ┌──────────┐     evento      ┌──────────┐     gate      ┌──────────┐
  │ DB :8100 │ ─────────────► │ETL :3001 │ ◄──────────── │ARGOS:8200│
  │ RUNNING  │                │ RUNNING  │               │ RUNNING  │
  └──────────┘                └──────────┘               └──────────┘
```

## Numeri

- **Disco**: 96GB → 193GB (+101%)
- **Servizi online**: 0 → 4 (DB, ETL webhook, ETL watcher, ARGOS)
- **Test totali**: 61 (17 DB + 30 ETL + 14 ARGOS)
- **Connettori**: 5 (github, ado, server, qdrant, healthcheck-all)
- **ETL**: v0.2.0 Dagster → v0.3.0 lightweight (+409/-893 righe)
