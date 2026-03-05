# The Nonna Standard — Software per la Nonna

> Se la nonna non riesce a usarlo leggendo solo le istruzioni, non e pronto.

## Principio

Ogni pezzo di software, tool, script, API, workflow o configurazione della piattaforma EasyWay
DEVE essere utilizzabile da chiunque — umano o agente AI — senza conoscenze pregresse e senza
passaggi "ovvi" lasciati impliciti.

"La nonna" non e un insulto. E il nostro standard di qualita.

## Perche

Sessione 83: un agente AI (Gemini) ha perso 15 minuti cercando di creare un PBI su Azure DevOps
con curl, PowerShell quoting, base64 manuale — fallendo ripetutamente. Il tool `easyway-ado` con
CLI e MCP server esisteva gia, ma nessuno glielo aveva detto. Nessuna guida operativa spiegava
"per fare X, usa Y".

Il problema non era il tool. Il problema era l'assenza di istruzioni chiare.

## Le 5 regole della Nonna

### 1. Zero prerequisiti scontati

Ogni guida parte da zero. Non dare per scontato che chi legge sappia:
- Dove sono i file
- Quali variabili d'ambiente servono
- Come autenticarsi
- In quale directory eseguire i comandi

**Male**: "Lancia il briefing"
**Bene**: "Da `C:\old\easyway\ado\`, esegui `npx tsx bin/easyway-ado.ts briefing`. Il PAT viene letto automaticamente da `C:\old\.env.local`."

### 2. Metodo di approccio esplicito

Per ogni operazione, dichiarare l'ordine di preferenza dei tool disponibili.
Chi legge non deve scegliere — deve solo seguire la lista dall'alto.

**Esempio**:
```
Per interagire con ADO:
1. MCP Server (se disponibile nel tuo IDE)
2. CLI: npx tsx bin/easyway-ado.ts <comando>
3. curl (ultimo resort, quoting fragile)
```

### 3. Ricette copia-incolla

Ogni guida include ricette task-oriented: "Vuoi fare X? Copia questo."
Non spiegazioni astratte — comandi pronti con valori di esempio reali.

**Male**: "Puoi creare work item usando l'API REST di Azure DevOps con un PATCH request..."
**Bene**:
```bash
# Creare un PBI
npx tsx bin/easyway-ado.ts wi create "Product Backlog Item" "Descrizione del lavoro"
# Output: Created #103 - Descrizione del lavoro
```

### 4. Troubleshooting con causa e soluzione

Non basta dire "se non funziona, controlla i log". Serve una tabella:

| Problema | Causa | Soluzione |
|----------|-------|-----------|
| `401 Unauthorized` | PAT scaduto | Rigenera PAT in ADO > User Settings > Personal Access Tokens |
| MCP non risponde | Non buildato | `cd easyway/ado && npm install && npm run build` |

### 5. Funziona da qualsiasi punto di partenza

Un agente puo lavorare da `C:\old\easyway\portal\`, da `~/.gemini/scratch/`, o da SSH sul server.
Le istruzioni devono funzionare indipendentemente dalla directory corrente.
Usare path assoluti, dichiarare le variabili d'ambiente necessarie, fornire alternative.

## Quando applicare

**Trigger**: ogni volta che crei o modifichi una capability (tool, script, workflow, MCP, API, connettore).

**Azione**: crea o aggiorna una guida operativa in `wiki/guides/` seguendo le 5 regole.

**Verifica**: rileggi la guida fingendo di non sapere nulla del progetto. Se ti serve un'informazione
che non c'e scritta, la guida e incompleta.

## Checklist per ogni guida operativa

- [ ] Tabella "Cosa c'e e dove" con path, env vars, prerequisiti
- [ ] Metodo di approccio con ordine di preferenza
- [ ] Almeno 3 ricette copia-incolla per i task piu comuni
- [ ] Tabella troubleshooting con almeno i 3 errori piu frequenti
- [ ] Sezione "da directory esterne" per agenti che lavorano altrove
- [ ] Link a guide correlate

## Guide operative esistenti

| Guida | Capability |
|-------|-----------|
| `guides/agent-ado-operations.md` | Azure DevOps (MCP, CLI, curl) |
| `guides/connection-registry.md` | Connettori bash (GitHub, ADO, Qdrant, server) |
| `guides/polyrepo-git-workflow.md` | Git multi-repo (commit, PR, deploy) |
| `Runbooks/rag-operations.md` | RAG / Qdrant (ingest, search, reset) |

## Riferimenti

- Sessione 83: origine dello standard
- `.cursorrules` portal: regola "Guida Operativa" in sezione Integrita
- `MEMORY.md`: regola GUIDA OPERATIVA
