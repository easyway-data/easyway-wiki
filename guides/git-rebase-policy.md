---
title: "Git Rebase Policy — Quando e Come"
date: 2026-03-04
category: devops
tags: [domain/git, rebase, domain/policy, process/best-practices, domain/gedi]
gedi_case: 22
---

# Git Rebase Policy — Quando e Come

> *"Misuriamo due, tagliamo una."* — GEDI, Principio #1
>
> Il rebase riscrive la storia. Su un feature branch e pulizia.
> Su un branch protetto e distruzione. La differenza va codificata.

## Contesto

Questa policy nasce dal GEDI Case #22 (Session 63): durante il closeout, una PR wiki (#284) ha sviluppato conflitti perche main era avanzato dopo un merge. Il rebase ha risolto il problema, ma l'operazione non era documentata nel processo.

GEDI ha identificato un **gap nella Testudo**: le policy ADO proteggono main e develop (merge strategy, reviewer, WI linking), ma mancava una regola esplicita sul rebase.

## La Regola

### PERMESSO — Feature branch, pre-merge

```bash
# 1. Aggiorna il riferimento al target
git fetch origin <target-branch>

# 2. Rebase sul target aggiornato
git rebase origin/<target-branch>

# 3. Risolvi eventuali conflitti
# ... edit files, git add, git rebase --continue

# 4. Push con safety net
git push --force-with-lease
#          ^^^^^^^^^^^^^^^^ MAI --force
```

**Quando**:
- PR ha conflitti con il target branch
- Vuoi allineare il branch dopo merge sul target
- Pulizia commit prima di aprire PR (squash locale)

### VIETATO — Branch protetti

| Branch | Rebase | Force-push | Motivo |
|--------|--------|------------|--------|
| `main` | MAI | MAI | Storia pubblica, deployata |
| `develop` | MAI | MAI | Branch di integrazione condiviso |
| `baseline` | MAI | MAI | Snapshot immutabile |

**ADO enforcement**: policy `Require a merge strategy` (blocking) su main e develop impone merge no-fast-forward. Force-push bloccato da ADO su branch con policy attive.

### ATTENZIONE — Casi limite

| Scenario | Azione | Motivo |
|----------|--------|--------|
| Branch condiviso da piu autori | **Evita rebase** | Riscrive i commit degli altri |
| PR gia approvata | **Evita rebase** | Invalida la review, reviewer deve ri-approvare |
| Branch con CI in corso | **Aspetta** | Il rebase cambia SHA, la pipeline corre su commit fantasma |
| In dubbio | **Merge commit** | Safe default, nessun rischio di perdita |

## Safety Net: `--force-with-lease` vs `--force`

```bash
# CORRETTO — fallisce se qualcun altro ha pushato
git push --force-with-lease

# VIETATO — sovrascrive senza controllo
git push --force
```

`--force-with-lease` verifica che il remote ref sia dove te lo aspetti. Se qualcuno ha pushato nel frattempo, il push fallisce. Questo previene la sovrascrittura accidentale del lavoro altrui.

## Flusso Tipico: PR con Conflitti

```
1. ADO segnala mergeStatus: conflicts sulla PR
2. Locale:
   git fetch origin <target>
   git rebase origin/<target>
   # risolvi conflitti
   git add <files>
   git rebase --continue
   git push --force-with-lease
3. ADO ricalcola: mergeStatus: succeeded
4. PR pronta per review/merge
```

## Relazione con le Policy Esistenti

| Policy | Stato | Interazione con rebase |
|--------|-------|----------------------|
| Merge no-fast-forward (main, develop) | Attiva, blocking | Protegge da rebase accidentale su target |
| Minimum reviewers | Attiva, blocking | Dopo rebase, reviewer deve ri-approvare se commit cambiano |
| Work item linking | Attiva, blocking | Non impattata dal rebase (WI linkato alla PR, non al commit) |
| File size restriction | Attiva, blocking | Non impattata |

## Prevenzione Conflitti: PR Sequencing (Lesson S63b)

> **Prima di creare una PR, verificare se ne esiste una pendente sugli stessi file verso lo stesso target.**

| Situazione | Azione |
|---|---|
| Nessuna PR pendente sugli stessi file | Procedi normalmente |
| PR pendente sugli stessi file, non ancora mergiata | **Accorpa**: includi le modifiche nella nuova PR, abbandona la precedente con commento |
| PR pendente su file diversi | PR parallele OK, nessun rischio |

**Perche**: due PR che toccano lo stesso file verso lo stesso target generano conflitti merge garantiti. La seconda PR richiedera sempre un rebase dopo il merge della prima. Meglio accorpare subito.

**Come abbandonare**:
1. Crea la nuova PR con tutte le modifiche (vecchie + nuove)
2. Commenta sulla PR vecchia: "Accorpata in PR #NNN — stessi file modificati"
3. Abbandona la PR vecchia

## Principi GEDI Applicati

- **Measure Twice, Cut Once**: la distinzione feature/protetto va fatta PRIMA del rebase
- **Testudo Formation**: `--force-with-lease` e la policy ADO sono due scudi coordinati
- **Known Bug Over Chaos**: in dubbio, merge commit (brutto ma sicuro) > rebase (pulito ma rischioso)
- **Victory Before Battle**: documentare la policy PRIMA che qualcuno faccia un rebase sbagliato

---

**Autore**: GEDI Case #22, Session 63
**Approvato da**: Processo collaborativo umano + GEDI OODA
