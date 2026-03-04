---
title: "La Fabbrica Phase 3b — CI/CD Migration + Deploy Multi-Repo"
date: 2026-03-04
session: 58
category: infrastructure
tags: [la-fabbrica, phase-3b, ci-cd, deploy, gedi]
---

# La Fabbrica Phase 3b — CI/CD Migration + Deploy Multi-Repo

## Session 58 — Continuazione da Session 57

### Cosa

Semplificazione del monorepo EasyWayDataPortal: rimozione infra estratta,
pipeline CI/CD ridotto per portal-api only, deploy script multi-repo.

### Perché

Il monorepo conteneva ancora 32 file infra (docker-compose, Caddyfile, terraform)
già estratti in easyway-infra (Phase 2). Il pipeline CI/CD aveva 779 righe con
7 wiki lint jobs, agent governance, orchestrator — tutti broken post-extraction.
Il deploy sul server richiedeva una strategia multi-repo.

### Come

1. **Infra cleanup** (PR #265 → develop):
   - Rimossi: Caddyfile, 13 docker-compose*.yml, infra/, infrastructure/gitlab/,
     config/, skills-lock.json, ewctl.ps1 (broken wrapper)
   - 35 file, -3408 righe totali

2. **Pipeline semplificato** (azure-pipelines.yml: 779 → ~230 righe, -78%):
   - RIMOSSI: 7 wiki lint jobs, EnforcerCheck, Orchestrator, VersionsReport,
     WikiLint, KBConsistency, ActivityLog, GatesReport, DocAlignmentCheck
   - PRESERVATI (GEDI review): NodeBuild, DBDriftCheck, PreDeployChecklist,
     EventsSchemaValidate — gate portal-specific ancora funzionanti
   - DISABILITATI: Deploy stages (pending multi-repo setup)
   - BranchPolicyGuard + SourceSyncGuard: riscritti in bash (no scripts/pwsh)

3. **Deploy multi-repo** (easyway-infra PR #266):
   - `scripts/deploy.sh`: Standard Interface (Electrical Socket Pattern)
   - Testudo check: valida prerequisiti prima di qualsiasi azione
   - Bridge: `docker compose --project-directory <portal> -f <infra>/docker-compose.yml`
   - Validato su server: stessa semantica del deploy attuale

4. **infrastructure/gitlab**: migrato da monorepo a easyway-infra (3 file terraform)

5. **easyway-infra clonato sul server**: ~/easyway-infra disponibile

### Q&A / Decisioni

**Q: Perché non rimuovere tutti i quality gate?**
A: GEDI "Measure Twice, Cut Once" — DBDriftCheck e PreDeployChecklist sono
portal-specific e funzionano. Rimuovere solo gate broken, non quelli funzionanti.

**Q: Come funziona il deploy multi-repo?**
A: GEDI "Electrical Socket Pattern" — deploy.sh è il plug, i repo sono i cavi.
Chi deploiya chiama solo `~/easyway-infra/scripts/deploy.sh [env]`.

**Q: Iron Dome hooks?**
A: Pre-commit base (security checklist) funziona. Secrets scan (ewctl.secrets-scan.psm1)
bloccato: richiede merge initial-import → main su easyway-agents.

### PR Create

| PR | Repo | Source → Target | Stato |
|----|------|-----------------|-------|
| #265 | EasyWayDataPortal | feat/phase3b-infra-cleanup → develop | Active |
| #266 | easyway-infra | feat/gitlab-terraform-extraction → main | Active |

### Backlog Residuo (Session 59+)

- Rename repo EasyWayDataPortal → easyway-portal (dopo merge PR #265 su main)
- Merge easyway-agents initial-import → main (672 file, 236 commit)
- Restore Iron Dome secrets scan (dipende da merge agents)
- Re-enable deploy stages in pipeline (dopo rename + deploy.sh validato)
- GitHubMirror: aggiornare URL dopo rename
- Fix sql-edge depends_on (bug preesistente in docker-compose.yml)

### Lesson

- GEDI va invocato PRIMA di scrivere codice, non dopo
- Il bridge `--project-directory` è la soluzione più pulita per multi-repo compose
- I quality gate portal-specific (DBDrift, Checklist) hanno valore indipendente dal monorepo
