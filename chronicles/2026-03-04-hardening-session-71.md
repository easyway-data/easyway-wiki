---
title: "Session 71 — The Hardening"
date: 2026-03-04
category: infrastructure
session: S71
tags: [branch-protection, pipeline, ci-cd, ado, github, governance]
---

# Session 71 — The Hardening

Dopo la grande migrazione della Fabbrica e la potatura dei branch, era il momento di mettere i lucchetti. La sessione 71 e la sessione della governance: ogni repo ha ora le sue guardie.

## Le Pipeline Prendono Forma

Il primo segnale e stato il fix della pipeline #109 — un PAT GitHub mancante nella variable group. Risolto quello, il vero lavoro e cominciato: tre pipeline dedicate, una per repo satellite. Wiki ha il suo lint e il suo mirror. Agents ha i suoi test. Infra ha la sua validazione. Ognuna indipendente, ognuna con il suo GitHub mirror stage.

PR #316, #317, #318 — tre pipeline, tre run verdi, un unico Work Item (#59) a tenerle insieme.

## I Lucchetti

Branch protection su ADO: 5 policy a livello di progetto, poi policy specifiche per wiki, agents e infra. Require PR, minimum reviewer, no force push, Work Item linking. Le stesse regole che proteggono il portal ora proteggono l'intera Fabbrica.

Su GitHub, wiki e agents — i repo pubblici — hanno ottenuto la stessa protezione. Non si pusha su main senza PR e review.

## La Pulizia del Backlog

Sei voci chiuse nella wiki backlog: pipeline split, ADO branch protection, GitHub branch protection, npm audit vulns, .cursorrules refs, CI deploy gates. PR #319 ha portato ordine.

## Il Bilancio

| Task | PR | WI | Stato |
|------|----|----|-------|
| Pipeline fix (GitHub PAT) | — | — | Re-run #109 verde |
| S70 closeout | #315 | #58 | Merged |
| ADO branch protection | — | — | 5 policy progetto + repo-specifiche |
| Pipeline split per-repo | #316, #317, #318 | #59 | Merged, 3 run verdi |
| GitHub branch protection | — | — | wiki + agents protetti |
| Backlog update + refs audit | #319 | #58 | Merged |

---

*"Prima costruisci la fabbrica, poi metti i lucchetti."*
