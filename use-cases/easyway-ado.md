# easyway-ado — Use Cases

> Il framework ADO opinionato che sa l'ordine delle cose.

## Cosa lo rende diverso

MCP = connessioni (tubi). **easyway-ado = framework** — sa l'ordine di esecuzione: prima WI poi PR (Palumbo), prima policy check poi merge, prima briefing poi decisione. Le regole EasyWay sono nel DNA del codice.

---

## UC-1: Palumbo Enforcement — PR senza WI rifiutata

**Scenario**: Un agente AI tenta di creare una PR senza specificare un Work Item.

**Input**:
```bash
easyway-ado pr create easyway-portal feature/quick-fix
```

**Output**:
```
Error: Palumbo mutu non po' essere servuto — specifica un Work Item con --wi <id>.
Ogni PR deve essere collegata a un Work Item.
```

**Perche conta**: In un flusso agentico, l'AI potrebbe "dimenticare" il WI per velocizzare. Il framework lo impedisce strutturalmente — non e un lint, e un rifiuto hard nel codice.

---

## UC-2: PR Create con ArtifactLink automatico

**Scenario**: Un agente crea una PR collegata a un PBI. Il framework crea la PR E linka automaticamente l'ArtifactLink sul WI.

**Input**:
```bash
easyway-ado pr create easyway-portal feature/new-api --wi 87 --title "PBI-87: New API endpoint"
```

**Output**:
```
Created PR #330 [easyway-portal]
Title: PBI-87: New API endpoint
Branch: feature/new-api -> develop
Linked WI: #87
URL: https://dev.azure.com/.../pullrequest/330
```

**Cosa succede dietro**: due chiamate API — POST PR + PATCH WI con ArtifactLink. L'agente non deve sapere come funziona, il framework gestisce la sequenza.

---

## UC-3: Policy Check prima del merge

**Scenario**: Prima di chiedere approvazione umana, l'agente verifica che le policy passino.

**Input**:
```bash
easyway-ado pr policy 330
```

**Output**:
```
PR #330 — Policy Status

Policies:
  APPROVED | Minimum number of reviewers (blocking)
  APPROVED | Comment requirements (blocking)
  RUNNING  | Build validation (blocking)

Reviewers:
  approved | Giuseppe Belviso
```

**Perche conta**: L'agente non chiede approvazione umana se la build sta ancora girando. Risparmia tempo all'umano.

---

## UC-4: Notifica compact per chat (Telegram/Teams/Discord)

**Scenario**: Dopo aver creato una PR, il MCP tool restituisce sia il JSON completo che il testo notification pronto per il forward su canale chat.

**MCP tool response** (campo notification):
```
🔀 PR #330 — easyway-portal
PBI-87: New API endpoint
WI: #87 | feature/new-api → develop
👉 https://dev.azure.com/.../pullrequest/330
```

**Flusso completo**:
```
Agente → ado_pr_create → JSON + notification
       → bot Telegram → umano legge in 5 secondi → approva
```

**Perche conta**: L'umano approva dal telefono in treno. Non deve aprire ADO, leggere la PR, cercare il WI. Tutto e nel messaggio.

---

## UC-5: Briefing di sessione

**Scenario**: Un agente inizia una sessione e chiede il contesto corrente.

**Input**:
```bash
easyway-ado briefing
```

**Output**: PR attive, PBI aperti, sprint corrente, lista repo. Tutto in un colpo.

**Perche conta**: Contesto zero-to-hero in 3 secondi. L'agente sa cosa sta succedendo senza 5 chiamate API separate.

---

## UC-6: WI Create con parent linking

**Scenario**: L'agente crea un PBI sotto un'Epic esistente.

**Input**:
```bash
easyway-ado wi create "Product Backlog Item" "Implementare notifiche push" --parent 61 --tags "notifications"
```

**Output**:
```
Created #100 [Product Backlog Item] New
Title: Implementare notifiche push
URL: https://dev.azure.com/.../_workitems/edit/100
```

**Cosa succede dietro**: JSON Patch con `System.LinkTypes.Hierarchy-Reverse` verso Epic #61.

---

## Pattern architetturale: La Presa Elettrica

```
                    ┌─────────────────┐
                    │  Dati strutturati │  ← wiGet(), prCreate(), briefing()
                    │  (return objects) │
                    └────────┬────────┘
                             │
              ┌──────────────┼──────────────┐
              │              │              │
        ┌─────┴─────┐ ┌─────┴─────┐ ┌─────┴─────┐
        │ Terminale  │ │   JSON    │ │   Chat    │
        │ (CLI)      │ │  (MCP)    │ │ (notify)  │
        │ console.log│ │ tool resp │ │ markdown  │
        └───────────┘ └───────────┘ └───────────┘
```

Stessa energia, tre spine. GEDI Case #32 — Electrical Socket Pattern.

---

## Comandi disponibili (v0.2.0)

| Comando | Tipo | Descrizione |
|---------|------|-------------|
| `briefing` | read | Overview completa progetto |
| `wi get <id>` | read | Dettaglio WI con relazioni |
| `wi list` | read | Lista WI attivi |
| `wi query <wiql>` | read | Query WIQL libera |
| `wi create <type> <title>` | write | Crea WI con parent linking |
| `wi update <id>` | write | Aggiorna stato/titolo/tags |
| `pr list` | read | PR per stato |
| `pr get <id>` | read | Dettaglio PR con WI linkati |
| `pr create <repo> <branch>` | write | Crea PR (Palumbo!) |
| `pr policy <id>` | read | Policy evaluations + votes |

**MCP tools**: stessi 10 comandi esposti come tool MCP con zod validation.
