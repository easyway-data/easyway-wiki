---
title: "Agentic Architecture Enterprise Standard"
tags: [gnosis, architettura, agenti, enterprise, governance, L1-L5, rbac, kpi]
status: active
created: 2026-03-07
layer: "Gnosis L1 - Chi sono gli agenti"
source: "#inbox-easyway/easyway/AGENTIC_ARCHITECTURE_ENTERPRISE_PRD.md"
---

# Agentic Architecture Enterprise Standard

> **Layer Gnosis**: L1 - Chi sono gli agenti
> **Relazioni**: vedi [gnosis-framework.md](gnosis-framework.md) per il quadro completo

---

## 1. Executive Summary

Standard Architetturale per sistemi agentici di livello Enterprise.

1. **Definizione**: Cosa distingue un Agente da un'Automazione.
2. **Tassonomia**: Classificazione universale per Ruolo (Brain/Arm) e Livello (L1-L5).
3. **Governance**: Regole ferree di ingaggio, promozione e controllo.
4. **KPI & Risk**: Framework di misurazione e mitigazione rischi.

**Obiettivo**: Sistema agentico enterprise-grade, stratificato, sicuro, auditabile e misurabile.

---

## 2. Definizione di Agente Enterprise

Un **Agente** e un sistema software che:
1. Riceve un input (task, evento, obiettivo)
2. Prende decisioni (logica deterministica o probabilistica)
3. Produce un'azione o un output

| Caratteristica | Automazione (Script) | Agente (AI) |
|:---|:---|:---|
| **Input** | Strutturato e Rigido | Ambiguo (Linguaggio Naturale) |
| **Logica** | Deterministica (If/Then) | Probabilistica (Ragionamento + Contesto) |
| **Adattabilita** | Zero (Fail on Error) | Alta (Retry, Self-Correction, Alternative) |

### Reality Check: La Natura dell'LLM

> Un LLM **NON e un database di verita**. E un motore probabilistico addestrato per **prevedere il prossimo token**.
> Il suo obiettivo primario e **completare il pattern** a qualsiasi costo -- rischiando di **inventare** ("Hallucination").
> Per questo, un Agente **non puo mai essere lasciato "libero"**. Deve essere ingabbiato da Prompt di Sistema, RAG e Guardrails deterministici.
> **L'AI non "sa", l'AI "genera".**

---

## 3. Le Due Dimensioni Fondamentali

- **Ruolo** = "Cosa Fa" (Brain vs Arm)
- **Livello** = "Quanto E Evoluto" (L1-L5)

Sono dimensioni indipendenti.

### Ruolo: Brain (Strategico)
- **Responsabilita**: Decisioni, Pianificazione, Governance, Gestione conflitti.
- **Output**: Policy, Direttive, Approvazioni, Piani.
- **Esempi**: `agent_governance`, `agent_scrummaster`, `agent_cartographer`.

### Ruolo: Arm (Esecutivo)
- **Responsabilita**: Scrivere codice, Deployare infrastruttura, Eseguire controlli, Applicare fix.
- **Output**: Codice, Terraform, Report, Azioni operative.
- **Esempi**: `agent_backend`, `agent_infra`, `agent_audit`.

---

## 4. Livelli Evolutivi (L1-L5)

### L1 -- Scripted Agent (Il "Robot")
- **Logica**: Deterministica (100%). Nessuna intelligenza/memoria.
- **Uso Ideale**: Enforcement, Validazioni, Controlli critici.
- *"Non penso, eseguo regole."*

### L2 -- LLM-Augmented Agent (Lo "Stagista")
- **Logica**: Probabilistica (LLM + RAG). Ragiona una volta (Single Shot).
- **Limite**: Non si auto-corregge. La qualita dipende dal primo tentativo.
- *"Ho un'intuizione basata sui dati, ma potrei sbagliare."*

### L3 -- Self-Improving Agent (L'"Esperto")
- **Logic**: Generate, Evaluate, Refine.
- **Caratteristiche**: Auto-valutazione, Quality scoring, Working memory.
- *"Ti consegno il codice solo ora che sono sicuro funzioni."*

### L4 -- Fully Autonomous Agent (Il "Manager")
- **Caratteristiche**: Memoria persistente, Budget awareness, Stop-loss, Trigger automatici (Watchdog).
- *"Gestisco il budget e ottimizzo le risorse proattivamente."*

### L5 -- Collective Intelligence (Lo "Sciame")
- **Caratteristiche**: Coordinazione dinamica, Ruoli emergenti, Collaborazione distribuita.
- **Stato**: Sperimentale.
- *"Agiamo come un organismo unico per sopravvivere."*

### Matrice Ruolo x Livello

| | L1 (Script) | L2 (LLM) | L3 (Reflective) | L4 (Autonomous) |
|:---|:---|:---|:---|:---|
| **Brain** | Rigido | Manager | Architetto Evoluto | Visionario |
| **Arm** | Solido | Potenziato | Autonomo Controllato | Esperto |

**Regola Fondamentale**: I Brain critici (Governance) non devono rimanere L1.

---

## 5. Architettura Stratificata (Core Design)

Un sistema maturo NON sostituisce i livelli inferiori. Li **orchestra**.

```
User -> L5 Swarm -> L4 Auto Orchestrator -> L3 Review Validator -> L2 Analyze Generator -> L1 Execute Enforcer -> System
```

**Regola d'Oro**: L1 e sempre l'ultimo step prima dell'azione.
L'orchestrazione dall'alto fornisce intelligenza; l'esecuzione dal basso fornisce sicurezza.

---

## 6. Enterprise Governance Framework

### 6.1 RBAC-A (Role-Based Agent Control)
Ogni agente deve avere definiti:
- Ruolo (Brain/Arm)
- Livello (L1-L5)
- Permessi (Read/Write path)
- Ambiente autorizzato (Dev/Prod)
- Budget massimo

### 6.2 Deterministic Gatekeeping
Ogni azione irreversibile (Deployment, Delete) richiede check L1:
- Verifica ambiente, backup, whitelist, policy.

### 6.3 Budget & Stop-Loss
Ogni Agente Autonomo (L4+) deve avere:
- Budget massimo (Token/$)
- Timeout rigido
- Max recursion depth
- **Kill switch** globale

---

## 7. Risk & Compliance Matrix

| Categoria Rischio | Mitigazione Mandatoria |
|:---|:---|
| **Operational** (errori, crash) | L1 Mandatory: esecuzione finale deterministica |
| **Financial** (costi incontrollati) | Cost Cap + Rate Limit: stop-loss a livello di API Gateway |
| **Cognitive** (allucinazioni) | L3 Scoring + Human Escalation: se confidence < soglia, ferma |
| **Compliance** (GDPR, dati) | Data Masking + Audit Trail: log immutabili |

> L'architettura a livelli e i guardrails mitigano drasticamente i rischi, ma **NON eliminano la necessita di supervisione umana**.
> Il **Giudizio Umano** rimane l'ultima linea di difesa.

---

## 8. KPI Framework

### Technical
- **Accuracy Rate**: % task completati correttamente al primo colpo
- **Self-Correction Rate**: % errori corretti autonomamente (L3)
- **Mean Resolution Time**: Tempo medio per task
- **Unsafe Action Block Rate**: Azioni pericolose bloccate dai Gatekeeper L1

### Governance
- **Audit Coverage**: 100% (non negoziabile)
- **Explainability**: >= 95%
- **Rollback Time**: < 15 min

### Financial
- **Cost per Task**: Costo medio ($)
- **Stop-Loss Activation**: 100% successo nei test

### Safety
- **Hallucination Detection**: >= 70% rilevate da L3 Validator
- **Escalation Accuracy**: >= 90%

---

## 9. Promotion Criteria

### L2 -> L3 (The Reliability Jump)
- >= 25% riduzione errori rispetto a L2
- >= 90% stabilita dello score di auto-valutazione

### L3 -> L4 (The Autonomy Jump)
- Budget Compliance: 100% (mai sforato)
- Zero Incidenti Distruttivi: 7 giorni in Sandbox senza rompere nulla
- Miglioramento continuo senza intervento umano

---

## 10. Infrastructure Safeguards

1. **Immutable Logs**: Audit trail inalterabile
2. **Kill Switch Globale**: Un bottone rosso per fermare tutto
3. **Sandbox First Policy**: Nessun L4 in Prod senza certificazione Sandbox
4. **Replay Capability**: Riprodurre ogni esecuzione per debug
5. **Max Parallel Threads**: Limite di concorrenza per evitare DoS interni

---

## 11. Anti-Pattern: Il "Super Agent"

**Da EVITARE**: Architettura monolitica con SuperAgent che Analizza, Decide, Esegue e si Auto-valida.

Rischi: Single point of failure, nessuna separazione responsabilita, difficile da auditare, costi incontrollati.

---

## 12. Strategic Principles (The Manifesto)

Un sistema agentico maturo:
1. **Non centralizza potere** (Architettura distribuita)
2. **Non elimina L1** (Il determinismo e sicurezza)
3. **Non idolatra L4** (Autonomia senza controllo = pericolo)
4. **Non opera senza Audit**
5. **Non opera senza Budget**

> La maturita agentica non e avere l'agente piu intelligente.
> E avere Architettura, Stratificazione, Controllo, Metriche, Governance, Sicurezza.
> Un sistema agentico enterprise-ready e un **ecosistema**. Non un Super Agente.

---

## Relazioni con gli altri Layer Gnosis

| Layer | Documento | Connessione |
|-------|-----------|-------------|
| L1 | **Questo documento** | Chi sono gli agenti, tassonomia e governance |
| L2 | [idea-to-production.md](idea-to-production.md) | Come si costruisce (il ciclo SDLC) |
| L3 | [agent-context-truth-memory-ledger.md](agent-context-truth-memory-ledger.md) | Come il sistema ricorda e impara |
