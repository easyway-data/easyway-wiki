---
title: "Governance & Orchestration - Execution Policy"
category: governance
domain: orchestrator
features:
  - recipes
  - manifests
  - execution-policy
entities: []
tags: [process/governance, orchestrator, recipes, manifests, execution-policy]
id: ew-archive-imported-docs-2026-01-30-agent-rules-governance
summary: TODO - aggiungere un sommario breve.
status: draft
owner: team-platform
llm:
  include: true
  pii: none
  chunk_hint: 250-400
  redaction: [email, phone]
type: guide
---

# 🎯 GOVERNANCE & ORCHESTRATION - Execution Policy

## 📋 Overview

Il sistema Axet usa un modello di **governance decentralizzata** dove:
1. **recipes.jsonl** definisce i comandi safe (`autoExecute: true, safe: true`)
2. **orchestrator.js** legge i flag e genera un plan con `safeToAutoRun`
3. **manifest.json** degli agent definisce le `execution_policy`
4. **AI Assistant** legge le rules ed esegue autonomamente i comandi safe

---

## 🔧 Componenti

### 1. KB Recipes (recipes.jsonl)
**Path**: `Rules.Vault/agents/kb/recipes.jsonl`

**Formato**:
```json
{
  "id": "kb-ado-pbi-get",
  "trigger": "visualizza pbi",
  "intent": "ado:pbi.get",
  "procedure": "axctl --intent pbi <ID>",
  "autoExecute": true,  // 🚀 AI può eseguire automaticamente
  "safe": true          // ✅ Operazione read-only
}
```

**Actions con autoExecute=true**:
- ✅ `kb-ado-pbi-get` - Vista PBI 360°
- ✅ `kb-ado-pbi-children` - Lista figli
- ✅ `kb-ado-pipeline-get` - Pipeline linkate
- ✅ `kb-ado-pipeline-list` - Lista pipeline
- ✅ `kb-ado-pipeline-history` - Storico pipeline

**Actions senza autoExecute** (require approval):
- ❌ `kb-ado-testcase-001` - Crea test case
- ❌ `kb-ado-userstory-create` - Crea PBI

---

### 2. Orchestrator (orchestrator.js)
**Path**: `Rules.Vault/agents/core/orchestrator.js`

**Cosa Fa**:
1. Legge tutte le recipes da KB
2. Trova recipe per intent richiesto
3. Verifica `autoExecute` e `safe` flags
4. Genera plan con:
   ```json
   {
     "plan": {
       "intent": "ado:pbi.get",
       "recipeId": "kb-ado-pbi-get",
       "autoExecute": true,
       "safeToAutoRun": true,
       ...
     }
   }
   ```

**Logica Chiave**:
```javascript
const autoExecute = recipe?.autoExecute === true && recipe?.safe === true;
const safeToAutoRun = recipe?.safe === true;
```

---

### 3. Agent Manifest (manifest.json)
**Path**: `Rules.Vault/agents/agent_ado_userstory/manifest.json`

**Nuova Sezione** `execution_policy`:
```json
{
  "execution_policy": {
    "autonomous_execution": true,
    "safe_actions": [
      "ado:check",
      "ado:pbi.get",
      "ado:pbi.children",
      "ado:pipeline.get-runs",
      "ado:pipeline.list",
      "ado:pipeline.history",
      "ado:userstory.export"
    ],
    "require_approval": [
      "ado:userstory.create",
      "ado:testcase.create"
    ],
    "notes": "Read-only actions are always safe for auto-execution."
  }
}
```

**Cosa Definisce**:
- `autonomous_execution: true` → Agent supporta auto-execution
- `safe_actions` → Lista azioni sempre safe
- `require_approval` → Lista azioni che richiedono conferma

---

## 🚀 Workflow Completo

### Scenario: User dice "Mostra PBI 184797"

**1. AI legge EXECUTION_RULES.md**:
```
Regola: Visualizzazione PBI = SAFE, esegui direttamente
```

**2. AI consulta recipes.jsonl**:
```json
{
  "id": "kb-ado-pbi-get",
  "autoExecute": true,
  "safe": true,
  "procedure": "axctl --intent pbi <ID>"
}
```

**3. Orchestrator genera plan** (se invocato):
```json
{
  "autoExecute": true,
  "safeToAutoRun": true
}
```

**4. AI esegue direttamente**:
```powershell
# SafeToAutoRun: true
axctl --intent pbi 184797
```

**5. AI mostra risultato**:
```
✅ Ecco il PBI #184797:
[output]
```

---

## 📊 Decision Matrix

| Action | recipes.jsonl | manifest.json | AI Behavior |
|--------|---------------|---------------|-------------|
| `pbi.get` | `autoExecute:true, safe:true` | In `safe_actions` | ESEGUE direttamente |
| `pbi.children` | `autoExecute:true, safe:true` | In `safe_actions` | ESEGUE direttamente |
| `export` | NO flag (yet) | In `safe_actions` | ESEGUE con `-Print` |
| `testcase.create` | NO `autoExecute` | In `require_approval` | CHIEDE conferma |

---

## 🎯 Safety Rules

### SEMPRE Safe (auto-run):
```
✅ Read-only queries (pbi.get, children, pipeline)
✅ Export ADO (no write operations)
✅ Visualizzazioni (show-last-export.ps1)
✅ Get-*, Import-Csv, etc.
```

### SEMPRE Ask (require approval):
```
❌ Creazione work items
❌ Modifica work items
❌ Eliminazione file
❌ Deployment/Build triggers
```

---

## 🔄 Governance Flow

```
User Request
    ↓
AI reads EXECUTION_RULES.md
    ↓
AI checks recipes.jsonl for autoExecute flag
    ↓
AI checks manifest.json execution_policy
    ↓
Decision:
├─ All safe → EXECUTE (SafeToAutoRun=true)
└─ Requires approval → ASK user
```

---

## 📝 File Summary

| File | Ruolo | Aggiornato |
|------|-------|------------|
| **recipes.jsonl** | KB recipes con `autoExecute` flags | ✅ |
| **orchestrator.js** | Genera plan con `safeToAutoRun` | ✅ |
| **agent_ado_userstory/manifest.json** | `execution_policy` definita | ✅ |
| **EXECUTION_RULES.md** | Rules per AI | ✅ |
| **TASK_RULES.md** | Workflow operativi | ✅ |

---

**Versione**: 1.0 (Autonomous Governance)
**Data**: 2026-01-12



