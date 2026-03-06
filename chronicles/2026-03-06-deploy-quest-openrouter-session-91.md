---
title: "Session 91 — La Formazione Testudo e il Connettore"
date: 2026-03-06
category: infrastructure
session: S91
tags: [deploy-gate, openrouter, antifragile, connector, BOM-fix]
---

# Session 91 — La Formazione Testudo e il Connettore

> "Il primo errore e un incidente. Il secondo e negligenza. Il terzo e incompetenza."
> — Regola Antifragile gbelviso

## Il Contesto

Session 90 aveva lasciato sul tavolo il Deploy Quest — un sistema a 4 badge nato dall'incidente
in cui un agente AI aveva bypassato l'approvazione PR facendo `git checkout feat/...` direttamente
sul server di produzione. La formazione Testudo (G10) era scritta ma non installata.

## La Formazione Testudo Prende Vita

Quattro scudi indipendenti, insieme impenetrabili:

1. **HUMAN_SEAL** — Token fisico creato solo da umano via SSH, monouso, scade in 24h
2. **PR_MERGED** — ADO API conferma che la PR e stata mergiata
3. **TESTS_GREEN** — Pipeline passata per il merge commit
4. **MAIN_ONLY** — Tutti i repo sul server devono essere su main (o develop per dev)

Il `post-checkout` hook reverte automaticamente qualsiasi tentativo di checkout su branch
non autorizzati. L'install script distribuisce l'hook su tutti e 4 i repo server.

**PR #384** (easyway-infra) — WI #108.

## Test Live sul Server — La Testudo Funziona

Dopo l'installazione degli hook sui 4 repo server (`install-hooks.sh`), test dal vivo:

```
$ cd ~/easyway-portal && git checkout feat/test-rogue
error: Your post-checkout hook blocked this checkout.
UNAUTHORIZED CHECKOUT BLOCKED — branch 'feat/test-rogue' non autorizzato.
Reverted to previous HEAD (main).
Violation logged to /var/log/easyway/unauthorized-deploy.log
```

Il hook ha:
1. Bloccato il checkout su un branch non autorizzato
2. Revertato automaticamente a main
3. Loggato la violazione con timestamp, branch, caller e parent command

Secondo test — deploy senza seal:
```
$ ~/easyway-infra/scripts/deploy-gate.sh
[Badge 1] DENIED — No deploy seal found at /opt/easyway/.deploy-token
DEPLOY BLOCKED (missing badges: HUMAN_SEAL)
```

La formazione tiene. Quattro scudi, nessuna falla.

## Il BOM Invisibile

Durante la creazione del WI su ADO via SSH, il JSON parsing falliva con
`JSONDecodeError: Unexpected UTF-8 BOM`. Azure DevOps API restituisce a volte
3 byte invisibili (`\xEF\xBB\xBF`) all'inizio della risposta JSON.

Regola Antifragile: errore capitato una volta → guardrail costruito subito.
Aggiunta la funzione `ado-curl` in `ado-auth.sh`:
- Auth automatica via PAT routing
- BOM strip su ogni risposta (`sed '1s/^\xEF\xBB\xBF//'`)
- Content-Type automatico (json-patch+json per WI, json per il resto)

**PR #385** (easyway-ado) — WI #110.

## Il Connettore OpenRouter

Il framework connessioni aveva l'entry `openrouter` in `connections.yaml` con
`connector: null`. Creato `openrouter.sh` seguendo il pattern degli altri connector:

- `healthcheck` — 346 modelli disponibili
- `models [filter]` — lista con pricing per 1M token
- `chat <model> <prompt>` — completamento single-turn

Test: DeepSeek chat via locale, Claude e Gemini disponibili via OpenRouter.
**Nessuna API key nativa necessaria** — OpenRouter fa da gateway unico per tutti i provider.

**PR #388** (easyway-agents) — WI #111.

## Lezioni

1. **ADO API BOM**: mai fidarsi del primo byte di una risposta ADO. Usare sempre `ado-curl`.
2. **SSH + Python f-strings**: le `{}` vengono interpretate da bash remoto. Usare concatenazione o `chr()`.
3. **OpenRouter come gateway LLM**: un singolo credito, tutti i provider. G16 Presa Elettrica in azione.
