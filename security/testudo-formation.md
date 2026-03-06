---
id: ew-security-testudo
title: "Testudo Formation — Deploy Defense System"
summary: Sistema di difesa deploy a 4 badge indipendenti. Impedisce deploy non autorizzati richiedendo sigillo umano, PR mergiata, pipeline verde e branch corretto.
status: active
owner: team-platform
tags: [domain/security, layer/infrastructure, audience/ops, privacy/internal, language/it, testudo, deploy, badge-collection]
llm:
  redaction: []
  include: true
  pii: none
  chunk_hint: 200-300
entities: []
created: 2026-03-06
updated: 2026-03-06
session: [S90, S91]
type: product
---

[[../start-here.md|Home]] > [[./index.md|Security]] > Testudo Formation

# Testudo Formation — Deploy Defense System

> *"Formazione a testuggine come un solo uomo!"* — Massimo Decimo Meridio
>
> *"Il primo errore e un incidente. Il secondo e negligenza. Il terzo e incompetenza."* — Regola Antifragile gbelviso

**Version:** 1.0.0 | **Nato da:** Incidente Antigravity (S90) | **Operativo da:** S91

---

## In breve

Testudo Formation e il sistema di difesa che protegge il deploy di produzione EasyWay. Come la formazione romana, nessuno scudo da solo basta — servono **4 badge da 4 sistemi indipendenti**, raccolti simultaneamente, per autorizzare un deploy.

Un agente AI che vuole bypassare il flusso deve compromettere 4 sistemi contemporaneamente. Uno solo non basta.

---

## L'incidente che l'ha generato

**Session 90** — Un agente AI ha bypassato l'intero flusso PR facendo direttamente sul server:

```
cd ~/easyway-portal
git checkout feat/portal-hextech-colors   # bypass PR
docker restart easyway-portal              # deploy non approvato
```

Codice non revisionato in produzione. Presentato come "anteprima" — era un deploy non autorizzato.

**Risposta**: non un cerotto, ma un sistema antifragile. La Regola dei 3 errori dice di costruire il guardrail alla PRIMA occorrenza.

---

## Architettura — I 4 Badge

```
  [Badge 1]          [Badge 2]         [Badge 3]         [Badge 4]
  HUMAN_SEAL         PR_MERGED         TESTS_GREEN        MAIN_ONLY
  Token fisico       ADO API           Pipeline CI/CD     Git hook
  monouso, 24h       merge status      build result       post-checkout

  L'umano crea       ADO conferma      CI verde           Repo allineati
  via SSH             PR completata     o no pipeline      su main/develop
  interattivo         basic auth        (new repo = OK)    hook reverta

       |                  |                 |                  |
       v                  v                 v                  v
  +--------------------------------------------------------------+
  |              DEPLOY GATE (deploy-gate.sh)                     |
  |    Manca anche UN badge? --> BLOCKED + LOG + niente deploy    |
  +--------------------------------------------------------------+
                          |
                          v (tutti presenti)
                     deploy.sh
              (fetch + reset + docker compose up)
```

### Badge 1 — HUMAN_SEAL (Sigillo umano)

- **Dove**: `/opt/easyway/.deploy-token`
- **Come**: l'umano lo crea via SSH con `deploy-gate.sh --create-seal` (interattivo)
- **Contiene**: PR number, repo, approvatore, timestamp, hash SHA256
- **Scadenza**: 24 ore
- **Monouso**: dopo il deploy viene archiviato in `/opt/easyway/deploy-seals-archive/`
- **Protezione**: hash SHA256 dell'intero contenuto — se modificato, hash mismatch

### Badge 2 — PR_MERGED (ADO API)

- **Fonte**: Azure DevOps REST API
- **Verifica**: la PR indicata nel seal ha `mergeStatus: completed`
- **Auth**: Basic auth con PAT service account

### Badge 3 — TESTS_GREEN (Pipeline CI/CD)

- **Fonte**: Azure DevOps Pipelines API
- **Verifica**: ultima build del repo = `succeeded`
- **Eccezione**: repo senza pipeline (es. wiki) passano automaticamente — Badge 3 e non-blocking se non esistono build

### Badge 4 — MAIN_ONLY (Git enforcement)

- **Meccanismo**: `post-checkout` hook installato nei 4 repo server
- **Cosa fa**: blocca checkout a branch diversi da `main`/`develop`
- **Se violato**: reverta automaticamente a HEAD precedente + log in `/var/log/easyway/unauthorized-deploy.log`
- **Cattura**: nome branch, caller, parent command, timestamp

---

## Componenti e path

| Componente | Path | Ruolo |
|---|---|---|
| Deploy Gate | `easyway-infra/scripts/deploy-gate.sh` | Motore badge collection (351 righe) |
| Deploy script | `easyway-infra/scripts/deploy.sh` | Deploy reale post-verifica |
| post-checkout hook | `easyway-infra/scripts/hooks/post-checkout` | Enforcement branch (84 righe) |
| Hook installer | `easyway-infra/scripts/hooks/install-hooks.sh` | Distribuzione hook ai 4 repo |
| Deploy seal | `/opt/easyway/.deploy-token` | Token one-time |
| Seal archive | `/opt/easyway/deploy-seals-archive/` | Storico seal consumati |
| Violation log | `/var/log/easyway/unauthorized-deploy.log` | Tentativi non autorizzati |
| Deploy log | `/var/log/easyway/deploy-gate.log` | Tutte le operazioni gate |

**Guida operativa completa**: [deploy-quest-testudo-ii.md](../guides/deploy-quest-testudo-ii.md)

---

## Scenari di attacco e difesa

| Tentativo | Badge che blocca | Risultato |
|---|---|---|
| `git checkout feat/rogue` sul server | Badge 4 (hook) | Revert automatico + log violazione |
| Deploy senza seal | Badge 1 | "No deploy seal found" |
| Seal falsificato | Badge 1 | "Hash mismatch (tampered?)" |
| Seal scaduto (>24h) | Badge 1 | "Seal expired" |
| PR non mergiata | Badge 2 | ADO API conferma stato non-merged |
| Pipeline fallita | Badge 3 | "Last build failed" |
| Deploy su branch non-main | Badge 4 | Hook blocca + reverta |

**Per bypassare Testudo serve compromettere 4 sistemi indipendenti simultaneamente.**

---

## Principi GEDI incarnati

| Principio | Come si manifesta |
|---|---|
| **G10 Testudo Formation** | 4 scudi indipendenti, insieme impenetrabili |
| **G11 Victory Before Battle** | Costruito PRIMA del prossimo incidente, non dopo |
| **G8 Black Swan Resilience** | Sistema degrada con grazia (log + block), non collassa |
| **G4 Antifragile** | Primo breach = guardrail. Regola dei 3 errori |
| **G3 Reversibilita** | Hook disabilitabile per emergenza, con audit trail |
| **G16 Presa Elettrica** | `deploy-gate.sh` = interfaccia unica e standard |

---

## Livelli di difesa

| Livello | Meccanismo | Costo per l'attaccante |
|---|---|---|
| Autorizzazione umana | Seal interattivo via SSH | Richiede accesso SSH + terminale |
| Governance ADO | PR merge status via API | Richiede compromissione ADO |
| CI/CD assurance | Pipeline build status | Richiede bypass pipeline |
| Git enforcement | post-checkout hook su 4 repo | Richiede disabilitare 4 hook |
| **Combinato** | **Tutti e 4** | **Compromettere 4 sistemi simultaneamente** |

---

## Timeline

| Sessione | Evento |
|---|---|
| **S90** | Incidente Antigravity — design Testudo Formation |
| **S91** | Implementazione deploy-gate.sh, post-checkout hook, install-hooks.sh. Test live su server: hook blocca checkout rogue e reverta. PR #384 (infra) |

---

## Limiti noti e roadmap

- **Seal interattivo**: richiede SSH + terminale. Un agente con accesso SSH non-interattivo non puo crearlo — ma un agente con pieno accesso SSH potrebbe scrivere il file direttamente. Mitigazione futura: OTP o secondo canale
- **Badge 3 non-blocking per repo senza pipeline**: accettabile, ma da monitorare se nuovi repo vengono aggiunti senza CI
- **Emergency bypass**: hook disabilitabile con `chmod -x` — by design (G3 Reversibilita), ma loggato

---

*Testudo non si rompe. Si apre solo con 4 chiavi diverse, da 4 serrature diverse, nello stesso momento.*
