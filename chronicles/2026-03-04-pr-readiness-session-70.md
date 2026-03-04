---
title: "Session 70 — The Readiness Gate"
date: 2026-03-04
category: tooling
session: S70
tags: [briefing, pr-readiness, npm-audit, work-items, ado-api]
---

# Session 70 — The Readiness Gate

La sessione 70 nasce da un problema pratico: due PR (#309, #310) erano bloccate su ADO e non era chiaro il perche. La risposta era semplice — mancavano i Work Items linkati — ma il vero problema era che non c'era modo di saperlo senza andare a controllare manualmente sul portale.

## Il Briefing Che Vede Tutto

`Get-ADOBriefing.ps1` gia mostrava le PR aperte, ma non diceva se erano pronte per il merge. Oggi abbiamo aggiunto il **PR Readiness Check**: per ogni PR aperta, il briefing ora interroga le policy evaluations di ADO e verifica i Work Items linkati. L'output e semplice — `[OK]` o `[BLOCKED]` con il motivo.

La query e stata portata a livello di progetto: un singolo endpoint ADO restituisce le PR da tutti i repository, non solo dal portal. Un'unica chiamata, visibilita completa.

## Lo Sblocco

Con il nuovo check, le PR #309 e #310 sono state sbloccate in pochi minuti: creati i WI #53 e #54, applicato l'ArtifactLink bidirezionale (la lezione della Session 64 — `workItemRefs` nel body non basta, serve il PATCH esplicito sul WI).

## La Pulizia

`npm audit` sul portal segnalava 5 CVE, di cui una critical (minimatch). Un `npm audit fix` pulito ha risolto tutto. PR #312.

## Il Bilancio

Cinque PR create, cinque PR mergiate, tutte `[OK]`. Ogni PR con il suo Work Item, ogni WI con il suo ArtifactLink. Il processo funziona.

| PR | Repo | WI | Contenuto |
|---|---|---|---|
| #309 | agents | #53 | GEDI Cases #28-#29 |
| #310 | agents | #54 | Iron Dome pre-commit hooks |
| #312 | portal | #55 | npm audit fix 5 CVE |
| #313 | agents | #56 | Briefing readiness check |
| #314 | wiki | #57 | Backlog easyway-ado |

---

*"Non basta creare — bisogna sapere se e pronto."*
