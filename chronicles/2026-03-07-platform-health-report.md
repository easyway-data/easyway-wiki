---
title: "Il Check-up e la Lista dei Desideri"
date: 2026-03-07
category: infrastructure
session: S102
tags: [health-check, monitoring, n8n, backlog, platform, ado-api, deploy]
---

# Il Check-up e la Lista dei Desideri

Session 102 e stata breve ma chirurgica. Come un medico che passa per un controllo di routine e finisce per aggiornare la cartella clinica.

## Il Deploy del Dottore

Prima mossa: portare `ado-api.py` sul server. Un `git fetch && git reset --hard` e il wrapper Python era operativo. Verificato con `--help` — 9 subcomandi pronti all'uso, zero quoting issues.

## I PAT Stanno Bene

Il sospetto di S101 — `ADO_PR_CREATOR_PAT` che restituiva HTTP 203 — si e rivelato un falso allarme. Tutti e 3 i PAT rispondono HTTP 200, incluso l'endpoint PR specifico. Probabilmente un glitch transitorio, ma il controllo andava fatto.

## La Radiografia della Piattaforma

Il cuore della sessione: un health check completo del server. Il referto:

- **Disco 38%**, RAM 2.7G/23G, load 0.02 — il server respira bene, uptime 41 giorni
- **9 container su 10 sani** — l'unico "unhealthy" e MinIO (storage-s3), che funziona ma il suo healthcheck Docker fallisce
- **Qdrant green** ma con un calo punti: da 167,970 a 124,891 (-43k). Da investigare — probabilmente un re-index parziale
- **ADO Agent**: il processo gira (dal 27 Feb) ma il servizio systemd e inattivo. Funziona, ma non sopravvivrebbe a un reboot
- **23GB di Docker reclaimabili** tra immagini e build cache

## La Lista dei Desideri

Da questo check-up e nata un'idea: perche non automatizzarlo? Il `docker-health-report.json` che avevamo in easyway-n8n copriva solo Docker. Il nuovo **Platform Health Report** coprirebbe 6 aree in parallelo: system stats, Docker, service endpoints, Qdrant, PAT validity, ADO Agent.

Messo nel backlog con priorita alta. Il template `docker-health-report.json` resta come base, superato dal nuovo scope.

## Il Formato che Resta

Una piccola conquista di processo: il formato tabellare del health report (Componente | Stato | Note + findings numerati) diventa lo standard. Ogni volta che il medico passa, il referto ha sempre la stessa struttura. Leggibile, confrontabile, azionabile.

---

> *"La salute non e tutto, ma senza salute tutto e niente." — Schopenhauer. Vale anche per i server.*
