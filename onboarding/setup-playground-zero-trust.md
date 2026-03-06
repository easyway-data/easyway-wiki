---
id: ew-setup-playground-zero-trust
title: Setup ambiente di test/Sandbox e Zero Trust
tags: [domain/onboarding, domain/devx, domain/security, sandbox, domain/agents, zero-trust, audience/dev, language/it]
summary: Guida passo‑passo all’onboarding sicuro di EasyWay DataPortal — tutto in sandbox locale, senza esporre segreti o toccare risorse reali per principio Zero Trust.
status: draft
owner: team-platform
updated: '2026-01-06'
llm:
  include: true
  pii: none
  chunk_hint: 250-400
  redaction: [email, phone]
entities: []
type: guide
---

[Home](./start-here.md)

# 🚦 Setup ambiente di test / sandbox agentica + Zero Trust

## Contesto (repo)
- Obiettivi e principi: `agents/goals.json`
- Orchestrazione/gates (entrypoint): `scripts/ewctl.ps1`
- Ricette operative (KB): `agents/kb/recipes.jsonl`
- Osservabilita: `agents/logs/events.jsonl`
- Standard contesto: `Wiki/EasyWayData.wiki/onboarding/documentazione-contesto-standard.md`

**Questa guida ti permette di esplorare EasyWay DataPortal, agent, pipeline e automazioni in sicurezza massima, senza rischio di “toccare” dati o risorse reali.**

---

## 1. Cos’è il Playground Zero Trust

- Un ambiente *locale* o *dev isolato* pensato per:
  - Provare agent, workflow, pipeline e script senza accedere a sistemi esterni, database cloud, o segreti veri.
  - Imparare e testare tutto “come in produzione”, ma con dati/credenziali finti e nessun effetto collaterale.

---

## 2. Setup rapido sandbox

**A) Clona la repository e posizionati nella directory di lavoro**
```bash
git clone <URL_REPO_EASYWAY>
cd EasyWayDataPortal
```sql

**B) Crea un file di env “mock”**
```bash
cp .env.example .env
# oppure crea manualmente un file .env con variabili di esempio:
echo 'DB_CONN_STRING="Server=localhost;Database=FAKE;User Id=mock;Password=mock;"' > .env
echo 'AZURE_STORAGE_KEY="fake-key"' >> .env
echo 'AGENT_MODE=mock' >> .env
```sql

**C) Installa le dipendenze**
```bash
npm install --prefix portal-api/easyway-portal-api/
```sql

**D) Avvia i servizi in modalità sandbox (mock)**
```bash
cd portal-api/easyway-portal-api/
npm run dev:mock
# oppure setta DB_MODE=mock e avvia:
# DB_MODE=mock npm run dev
```sql

**E) Prova una pipeline agentica/CLI di test**
```bash
pwsh ewctl.ps1 check
# Esegue controlli diagnostici in sola lettura
```sql

---

## 3. Policy minime e cosa succede per ogni script

| Script/Agent             | Cosa fa in modalità default (mock/safe)           | Gate di sicurezza      | Come fare rollback/reset        |
|--------------------------|---------------------------------------------------|------------------------|---------------------------------|
| ewctl check            | Esegue controlli diagnostici (DB, Docs, Governance) | Sola lettura (Safe)    | N/A                             |
| ewctl plan             | Propone un piano d'azione (Fix)                     | Sola lettura (Safe)    | N/A                             |
| ewctl fix              | Applica le correzioni (es. DB Drift, Wiki Normalize)| Modifica file/DB       | Reset working dir / Restore DB  |
| agent_datalake           | Simula chiamate, non tocca Datalake reale         | Richiede AGENT_MODE=mock | Nessuna azione su Azure locale  |
| Tutti gli script         | Se DB_MODE/mock/env non è attiva, bloccano run    | Check var. env         | Modifica .env o reset working dir |

**SEMPRE evidenziato nei log “MOCK mode”, nessuna azione su cloud/DB veri.**

---

## 4. Come tornare “safe” / azzerare dati sandbox

- Cancella i file “output”/log in working directory, oppure resetta repo con:
```bash
git clean -fd
git reset --hard
```sql
- Ripristina .env_example su .env
- Per sicurezza, verifica SEMPRE che le variabili d’ambiente non contengano secret reali prima di avviare
- Loggati come utente standard, mai admin/root

---

## 5. Policy Zero Trust: mai esporre segreti veri!

> **Attenzione:**  
> Tutte le ricette onboarding, agent, CLI e workflow vanno testate PRIMA solo in modalità mock/sandbox.  
> Solo dopo review e approvals usare token/segreti veri, e MAI committare/gestire questi nel repo.

---

**Per dubbi, suggerisci update direttamente su questa guida via PR o issue!**



