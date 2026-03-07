---
id: valentino-framework
title: 🌹 The Valentino Framework
summary: Il Framework Frontend Sovrano e Antifragile di EasyWay.
status: active
owner: team-frontend
created: '2026-02-03'
updated: '2026-02-03'
tags: [domain/frontend, layer/core, privacy/internal, language/it]
llm:
  pii: none
  redaction: [email, phone]
  include: true
  chunk_hint: 500-800
entities: []
type: guide
---

# 🌹 The Valentino Framework

> *"Il Frontend è il nostro vestito. Deve essere tagliato su misura, cucito a mano, e durare per sempre."*

Il **Valentino Framework** è la risposta di EasyWay alla complessità accidentale del frontend moderno. Rifiuta le dipendenze esterne inutili in favore della **Sovranità Digitale**.

---

## 1. I Principi Fondanti

### A. Sovereign Architecture
Non dipendiamo da `npm`, React, Vercel o CDN esterne per il funzionamento base.
Tutto il necessario è nel repository. Se internet cade, noi lavoriamo.

### B. Haute Couture Engineering (Sartorialità)
Ogni componente è costruito "su misura" per il dominio EasyWay.
Non usiamo librerie generiche da 10MB per un bottone. Usiamo Web Components nativi standard.

### C. Agent-Native
Il codice è strutturato per essere letto e modificato dall'Intelligenza Artificiale.
- **Blueprints**: Modelli chiari.
- **Test E2E**: Feedback immediato per l'Agente.

### D. The Unified Portal (Voss & Sinek Doctrine)
Il concetto di "Sito Vetrina" è abolito. La parte pubblica è la vista non autenticata del Portale Operativo (Dogfooding Estremo).
- **Integrità (Sinek)**: Non "vendiamo" un'infrastruttura, la "mostriamo" scendendo in campo con la stessa tecnologia che offriamo al cliente. Le interfacce marketing manipolative tradiscono il nostro "Perché".
- **Empatia Tattica (Voss)**: Rifiutiamo fermamente l'"arroganza dell'ingegneria cruda". Il Portale deve essere una "Master Console" potente (Cyber/Agentica) ma estremamente elegante, accogliente e rassicurante visivamente (tramite Glassmorphism e animazioni) per non terrorizzare la mente primitiva dell'utente. Vogliamo che l'utente si senta ammesso a un'armeria d'elite, non di fronte a un prompt incomprensibile.

### E. The Guardian of Intent (GEDI Governance)
Mentre Valentino cuce l'aspetto visivo, **GEDI (La Coscienza)** valida l'intento.
Nessun design radicale viene rilasciato se GEDI determina che esso scade nell'arroganza tecnica (violando Voss) o nel marketing ingannevole (violando Sinek). Valentino costruisce la UI; GEDI si assicura che l'anima Sovereign Symbiosis rimanga incorrotta.

---

## 2. The Quality Shield (I Guardiani) 🛡️

Per garantire che questa "Sartorialità" non diventi fragilità, abbiamo istituito 4 Guardiani automatizzati che proteggono il codice ad ogni modifica.

### 👁️ Visual Guardian
*   **Ruolo**: Curatore Estetico.
*   **Strumento**: Playwright Visual Snapshots.
*   **Cosa fa**: Scatta foto pixel-perfect di ogni pagina. Se un margine si sposta di 1px, blocca il rilascio.
*   **Comando**: `npx playwright test visual.spec.ts`

### ♿ Inclusive Guardian
*   **Ruolo**: Garante dell'Accessibilità.
*   **Strumento**: Axe-Core (WCAG 2.1 AA).
*   **Cosa fa**: Scansiona il DOM cercando contrasti bassi, label mancanti o trappole da tastiera.
*   **Comando**: `npx playwright test accessibility.spec.ts`

### ⚡ Chaos Guardian
*   **Ruolo**: Stress Tester (Antifragilità).
*   **Strumento**: Gremlins.js.
*   **Cosa fa**: Scatena un'orda di "gremlins" che cliccano a caso, scrollano e toccano tutto per 10 secondi. Se la pagina crasha o la memoria esplode, il test fallisce.
*   **Comando**: `npx playwright test chaos.spec.ts`

### 🛡️ Code Guardian
*   **Ruolo**: Doganiere del Codice.
*   **Strumento**: Husky + Commitlint.
*   **Cosa fa**: Impedisce di committare messaggi sciatti (es. "fix") o codice che non passa il linter.
*   **Comando**: Automatico su `git commit`.

---

## 3. Sovereign Deployment

Il framework include un modello di deployment **User-Space**:
1.  **Build**: `npm run build` (Locale/CI).
2.  **Artifact**: Cartella `dist/` statica.
3.  **Run**: Python SimpleHTTPServer su porta >1024 (es. 8080).
4.  **Resilience**: Cron job `@reboot` per la persistenza.

---
*Vedi anche*: [[concept/history|Storia del Progetto]], [[domain/frontend|Dominio Frontend]]

