---
title: "Guardiani del Codice (The Untouchables) — PRD"
tags: [guardiani, security, governance, quality-gates, agents, audit]
status: active
created: "2026-03-07"
source: "#inbox-easyway/easyway/guardiani-del-codice.md"
llm:
  include: true
  pii: none
  chunk_hint: 400
---

# Guardiani del Codice (The Untouchables)

Questo documento definisce il concetto e la proposta operativa dei "Guardiani del Codice": agenti/auditor incorruttibili (stile Batman/Untouchables) che non fanno feature, ma proteggono il sistema con regole, prove e gate deterministici.

## Perche esistono

Il problema non sono gli agenti in se. Il problema e l'assenza di fondamenta:
- Loop di completamento ("finche non ho una risposta, continuo")
- Metriche su task done invece che su verita, qualita, riproducibilita
- Poco o zero audit trail
- Zero validazione scientifica
- Sicurezza trattata come post-it mentale
- Fiducia cieca: "se funziona una volta, va bene"

I Guardiani ribaltano questa logica: AI come sistema regolato, non magia.

## Principi (coerenti con goals.json)

- Human-in-the-loop: decisioni importanti sempre confermabili
- Trasparenza: piani, esiti e log strutturati ricostruibili
- Idempotenza: check e script ripetibili, sicuri
- Documentazione viva: KB + Wiki allineate ai cambi
- Policy e gates: qualita automatica e comprensibile
- Parametrizzazione: zero hardcode, segreti fuori dal repo
- Osservabilita: eventi e metriche (almeno eventi)

## Identita: anti-eroi, ma eroi

Sono potenti ma mai "trusted-by-default".
- Non "sanno la verita": producono esiti misurabili + evidenze.
- Hanno confini duri: allowed_paths e perimetri espliciti.
- Non si negoziano in runtime: regole versionate e deterministiche.

## Due fasi (audit prima, assunzione dopo)

### Fase 1 - External Audit (read-only, non bloccante)

Obiettivo: far vedere la realta a chi non li ha mai "visti".
- Scansionano repo/agents/wiki/pipeline e producono:
  - Report machine-readable (JSON)
  - Report umano (Markdown)
  - Lista issue (severita + evidenze + next step)
- Niente auto-fix di default (solo suggerimenti / patch proposte).

Deliverable tipici:
- `out/guardians-audit.json`
- `out/guardians-audit.md`
- `out/issues.json`
- (opzionale) attestato negativo "Found Wanting" se fallisce hard-fail.

### Fase 2 - Assunzione (integrati, con potere)

Obiettivo: trasformare i guardiani in gate non bypassabili.
- Entrano nel "sacred path" (es. `ewctl check` + CI).
- Solo i guardiani assunti diventano bloccanti.
- Eccezioni solo se tracciate (decision trace).

## Hard laws (non negoziabili)

Queste regole devono portare a FAIL oggettivo.
- No secrets in repo (token/password/chiavi)
- No scope break: modifiche fuori `allowed_paths`
- No skip validation: check riproducibili (lint/test/build) mancanti o rossi
- No bypass policy/approval (quando richiesto)

## Squadra minima (proposta)

1. **Guardiano Confini** (Scope/Paths) — Input: diff (git), manifest/agent. Output: pass/fail + lista violazioni.
2. **Guardiano Qualita** (Determinismo) — Input: diff + comandi standard di check. Output: pass/warn/fail + "how to reproduce".
3. **Guardiano Sicurezza** (Secrets/Injection) — Input: diff + KB/Wiki/recipes. Output: severita + match + evidenza.
4. **Guardiano Tracciabilita** (Audit/Decision Trace) — Input: PR/commit + intent dichiarato. Output: pass/fail + cosa manca.
5. **Guardiano Coerenza Docs/KB** (Living Docs) — Input: diff su DB/API/Wiki/CI. Output: pass/warn/fail + file richiesti mancanti.

## Certificati di affidabilita

I guardiani possono rilasciare un attestato basato su misure e prove.

### Tiers (doppia etichetta: canonico + storytelling)

| Tier | Label | Significato |
|------|-------|-------------|
| Bronze | Clean | Baseline ok, zero warning bloccanti |
| Silver | Sharp | Certificato con riserve (azioni consigliate) |
| Gold | Untouchable | Compliance piena |

`warn` non e un tier: e uno stato. `outcome`: `pass|warn|fail`. Tier valorizzato solo se `outcome != fail`.

### Formato minimo certificato (machine-verifiable)

Campi: `issuer`, `subject` (repo+branch+SHA), `timestamp`, `scope`, `controls[]`, `results[]`, `evidence[]`, `outcome`, `tier_canonical`, `tier_label`, `valid_until`.

## Rollout per test

1. **Shadow mode** (1-2 settimane) — Non bloccano, solo report. Metriche: false positive/negative, riproducibilita, tempo risparmiato.
2. **Advisory gate** — Blocca solo hard laws (secrets/scope/test rossi). Il resto e warn con next steps.
3. **Full gate** (solo dopo tuning) — I tier diventano requisito per merge/deploy.

## Integrazione (EasyWay)

Opzione preferita: integrarli come gates nel "kernel" `ewctl` (sacred path) e farli girare anche in CI.
- Locale: `ewctl check --json`
- CI: job governance gates + publish artifacts (report/certificato)
