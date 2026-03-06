---
title: "Session 72 — HALE-BOPP GitHub Launch"
date: 2026-03-04
category: milestone
session: S72
tags: [domain/hale-bopp, domain/github, open-source, domain/ci-cd]
---

# HALE-BOPP GitHub Launch

> Tre motori deterministici escono dal garage e prendono il volo.

## Il contesto

Per settimane, i tre moduli HALE-BOPP — il database schema manager (DB), l'ETL pipeline runner, e ARGOS il motore di data quality — erano rimasti come cartelle locali in `C:\old\HALE-BOPP\`. Niente git, niente CI, niente documentazione pubblica. Codice buono, ma invisibile.

La Fabbrica (S53-S62) aveva dimostrato che la polyrepo migration funziona. Era il momento di applicare lo stesso pattern al progetto open-source.

## Cosa e successo

In una singola sessione, i tre moduli sono passati da cartelle locali a repository GitHub completi:

- **Ristrutturazione filesystem**: le cartelle `DB-HALE-BOPP`, `ETL-HALE-BOPP`, `ARGOS-HALE-BOPP` sono diventate `db/`, `etl/`, `argos/` — brand-first, pulite
- **GitHub org `hale-bopp-data`**: tre repo pubblici sotto Apache 2.0
- **CI/CD**: GitHub Actions con pytest per tutti e tre. DB include un PostgreSQL 16 service container per i test di integrazione
- **47 test verdi**: 17 (DB) + 16 (ETL) + 14 (ARGOS), tutti passano al primo colpo
- **Branch protection**: main protetto con require PR, 1 reviewer, no force push

## Il risultato

```
PRIMA:                              DOPO:
C:\old\HALE-BOPP\                   C:\old\hale-bopp\
├── DB-HALE-BOPP/   (no git)        ├── db/    → github ✅ CI green
├── ETL-HALE-BOPP/  (no git)        ├── etl/   → github ✅ CI green
├── ARGOS-HALE-BOPP/ (no git)       ├── argos/ → github ✅ CI green
└── 15+ .md sparsi                   └── docs/  (strategy, non git)
```

## La lezione

Il pattern e chiaro: quando hai codice testato e stabile, il costo di non pubblicarlo e maggiore del costo di pubblicarlo. Tre ore di setup CI/GitHub valgono mesi di visibilita e contributi potenziali.

---

*47 test, 3 pipeline, 3 repo. I muscoli sono pronti.*
