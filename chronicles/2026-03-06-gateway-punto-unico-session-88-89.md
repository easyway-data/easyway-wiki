---
title: "Session 88-89 — Il Gateway e il Punto Unico"
date: 2026-03-06
category: infrastructure
session: S88-S89
tags: [gateway, ssh, domain/pat, domain/security, ado-api, branch-guard, domain/hale-bopp]
---

# Session 88-89 — Il Gateway e il Punto Unico

> "Il punto di accesso deve essere sempre uno delle connessioni, non 100mila" — gbelviso

## Il contesto

Il polyrepo era maturo, le pipeline attive, i PAT distribuiti. Ma c'era un problema strutturale: ogni script locale parlava direttamente con le API ADO usando PAT sparsi in `.env.local`. Troppi punti di ingresso, troppi segreti in giro. Serviva un collo di bottiglia — un gateway.

## La Regola dei 3 Errori (S88, Parte 1)

La sessione nasce da un errore ricorrente: PR create verso il branch sbagliato. La prima volta (PR #362 verso main invece di develop) era comprensibile. La seconda no. Nasce il G12 Branch Guard — enforcement dinamico develop-first a 3 livelli:

1. **Pipeline**: BranchPolicyGuard stage che blocca PR verso main se esiste develop
2. **ewctl**: check pre-commit che avvisa del target corretto
3. **ado.sh pr-create**: `_build_json()` Python antifragile con G12 auto-detect da `factory-vcs.json`

GEDI Casebook #34 registra il principio: la regola antifragile di gbelviso. Un errore e tollerabile. Due e disattenzione. Tre e incompetenza.

## Il Gateway SSH (S88, Parte 2)

Il cuore architetturale: `_gateway.py` sul server OCI diventa il proxy per tutte le chiamate API ADO.

- **connections.yaml** riceve la sezione `gateway: mode: server`
- **_common.sh** guadagna `_use_gateway()` e `_gw_api()` — helper per routing SSH
- **ado.sh** diventa dual-path: se il gateway e attivo, ogni chiamata API passa via SSH al server; altrimenti fallback su curl locale
- I **4 PAT ADO** vengono migrati su `/opt/easyway/.env.secrets` e rimossi da `.env.local`

Risultato: un unico punto di accesso, un unico luogo dove vivono i segreti.

## Il Commit Day (S89)

La S89 chiude il cerchio operativo. 5 repo con modifiche pendenti su `feat/s88-branch-guard-framework`:

| Repo | Commit | PR |
|------|--------|----|
| agents | gateway proxy + dual-path | #372 |
| wiki | connection registry docs | #368 |
| portal | Hale-Bopp comet + cursorrules | #371 |
| infra | cursorrules gateway refs | #369 |
| ado | cursorrules gateway refs | #370 |

Work Item #106 creato (Regola del Palumbo rispettata) e linkato a tutte e 5 le PR.

La PR portal (#371) aveva un merge conflict su `pages-renderer.ts` — develop aveva aggiunto le particle nodes, il feature branch la cometa Hale-Bopp. Risolto tenendo entrambi: le particelle orbitano, la cometa passa.

## Lezioni

- **Gateway > PAT distribuiti**: un solo punto di accesso riduce la superficie d'attacco e semplifica la rotazione dei PAT
- **G12 auto-detect**: `factory-vcs.json` come source of truth per i target branch elimina l'errore umano
- **Merge > discard**: quando due branch aggiungono feature complementari, la risposta e sempre tenere entrambe
