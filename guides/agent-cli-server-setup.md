# Agent CLI sul Server OCI — Guida Operativa

**Version:** 1.0.0
**Created:** 2026-03-06
**Session:** S90
**Owner:** EasyWay Platform Team

---

## Cosa c'e e dove

| Componente | Path / Comando | Note |
|---|---|---|
| **Server** | `ubuntu@80.225.86.168` | Ubuntu 24.04 ARM64, OCI |
| **SSH** | `ssh -i "/c/old/Virtual-machine/ssh-key-2026-01-25.key" ubuntu@80.225.86.168` | Da locale Windows |
| **nvm** | `~/.nvm/` | Node Version Manager — gestisce Node multi-versione |
| **Node 22 LTS** | `~/.nvm/versions/node/v22.22.1/` | Default via nvm |
| **Node 18 (system)** | `/usr/bin/node` | Ubuntu apt, usato come fallback |
| **Claude Code CLI** | `~/.nvm/versions/node/v22.22.1/bin/claude` | v2.1.70 |
| **Gemini CLI** | `~/.nvm/versions/node/v22.22.1/bin/gemini` | v0.32.1 (Google) |
| **Secrets** | `/opt/easyway/.env.secrets` | API keys — mai committare |
| **Agent manifests** | `~/easyway-agents/agents/agent_*/manifest.json` | Config LLM per agente |
| **LLM Integration Pattern** | `~/easyway-agents/agents/LLM_INTEGRATION_PATTERN.md` | Architettura L2+ |

### Prerequisiti

```bash
# nvm deve essere caricato in ogni sessione SSH
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh"

# Verifica
node --version   # v22.22.1
claude --version # 2.1.70
gemini --version # 0.32.1
```

**Importante**: nvm si carica automaticamente via `~/.bashrc` per login shell interattive.
Per comandi SSH non-interattivi, includere sempre il blocco `export NVM_DIR` prima del comando.

---

## Metodo di approccio

### Ordine di preferenza per LLM routing

1. **OpenRouter** (raccomandato) — una sola API key, tutti i modelli
2. **API diretta** (DeepSeek, Anthropic, Google) — per casi specifici o costi ottimizzati
3. **Ollama locale** — per test offline o dati sensibili

### Principio G16 — Presa Elettrica

> Non importa se l'energia viene dal nucleare, dal solare o dal carbone (Claude, Gemini, DeepSeek),
> basta che la presa sia standard (OpenAI-compatible API format).

Architettura:

```
Agent Manifest (llm_config.model)
    |
    v
OPENROUTER_API_KEY (una sola key in .env.secrets)
api_base: https://openrouter.ai/api/v1
    |
    +-- anthropic/claude-sonnet-4-6
    +-- google/gemini-2.0-flash
    +-- deepseek/deepseek-chat
    +-- meta-llama/llama-3.1-70b
    +-- ... (200+ modelli)
```

### Compatibilita protocolli (IMPORTANTE)

Le CLI agent e OpenRouter parlano **protocolli diversi**:

| Componente | Protocollo | Endpoint |
|---|---|---|
| **Claude Code CLI** | Anthropic Messages API | `api.anthropic.com/v1/messages` |
| **Gemini CLI** | Google Gemini API | `generativelanguage.googleapis.com` |
| **OpenRouter** | OpenAI-compatible | `openrouter.ai/api/v1/chat/completions` |
| **Agent scripts** (LLM_INTEGRATION_PATTERN) | OpenAI-compatible | Configurabile via manifest |

Conseguenza:
- **Claude Code CLI** richiede `ANTHROPIC_API_KEY` diretta (formato Anthropic)
- **Gemini CLI** richiede `GEMINI_API_KEY` diretta (formato Google)
- **OpenRouter** funziona come gateway per gli **agent script** che usano `Invoke-LLM` / curl con formato OpenAI-compatible
- I **manifest agent** (`llm_config.api_base`) puntano a OpenRouter per il routing multi-modello

```
CLI Agent (Claude/Gemini)           Agent Scripts (Invoke-LLM)
    |                                      |
    v                                      v
API Key diretta              OPENROUTER_API_KEY (una sola)
(Anthropic/Google)           api_base: openrouter.ai/api/v1
    |                                      |
    v                                      v
Protocollo nativo            OpenAI-compatible format
(Messages/Gemini API)        (chat/completions)
```

### Agent scripts con OpenRouter (raccomandato per automazione)

```bash
# Test diretto OpenRouter via curl (formato OpenAI-compatible)
source /opt/easyway/.env.secrets
curl -s https://openrouter.ai/api/v1/chat/completions \
  -H "Authorization: Bearer $OPENROUTER_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"model":"deepseek/deepseek-chat","messages":[{"role":"user","content":"Ciao"}],"max_tokens":50}'
```

Questo e il formato usato dagli agent script in `LLM_INTEGRATION_PATTERN.md`.
Il modello si sceglie nel manifest agent: `deepseek/deepseek-chat`, `anthropic/claude-sonnet-4-6`, `google/gemini-2.0-flash`, ecc.

### Claude Code CLI (richiede ANTHROPIC_API_KEY)

```bash
source /opt/easyway/.env.secrets
export ANTHROPIC_API_KEY=<api-key-anthropic>  # Da console.anthropic.com
cd ~/easyway-wiki && claude -p "leggi .cursorrules e dimmi le 3 regole piu importanti" \
  --allowedTools Read
```

### Gemini CLI (richiede GEMINI_API_KEY)

```bash
source /opt/easyway/.env.secrets
export GEMINI_API_KEY=<api-key-google>  # Da aistudio.google.com
cd ~/easyway-wiki && gemini -p "analizza la struttura della wiki"
```

---

## Ricette comuni

### 1. Lanciare un task agent dal locale

```bash
# Pattern base: SSH -> source secrets -> nvm -> cd repo -> CLI
ssh -i "/c/old/Virtual-machine/ssh-key-2026-01-25.key" ubuntu@80.225.86.168 \
  'export NVM_DIR="$HOME/.nvm" && [ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh" && \
   source /opt/easyway/.env.secrets && \
   cd ~/easyway-wiki && \
   claude -p "scan per link rotti e reporta" --allowedTools Read,Glob,Grep'
```

### 2. Code review automatica su una PR

```bash
ssh ubuntu@80.225.86.168 \
  'export NVM_DIR="$HOME/.nvm" && . "$NVM_DIR/nvm.sh" && \
   source /opt/easyway/.env.secrets && \
   cd ~/easyway-portal && \
   git diff origin/main...origin/feature-branch | \
   claude -p "Fai code review di questo diff. Segnala problemi di sicurezza, performance, e stile." \
   --allowedTools Read'
```

### 3. Aggiornamento wiki autonomo

```bash
ssh ubuntu@80.225.86.168 \
  'export NVM_DIR="$HOME/.nvm" && . "$NVM_DIR/nvm.sh" && \
   source /opt/easyway/.env.secrets && \
   cd ~/easyway-wiki && \
   claude -p "Verifica che tutti i file in guides/ abbiano frontmatter valido (title, date). Correggi quelli mancanti." \
   --allowedTools Read,Edit,Glob,Grep,Bash'
```

### 4. Aggiornare le CLI

```bash
ssh ubuntu@80.225.86.168 \
  'export NVM_DIR="$HOME/.nvm" && . "$NVM_DIR/nvm.sh" && \
   npm update -g @anthropic-ai/claude-code @google/gemini-cli && \
   claude --version && gemini --version'
```

### 5. Switchare versione Node

```bash
# Listare versioni installate
nvm ls

# Installare una nuova versione
nvm install 24

# Usare una versione specifica
nvm use 22

# Tornare a Node system (18)
nvm use system
```

---

## Troubleshooting

### Errore: `gemini: SyntaxError: Invalid regular expression flags`

**Causa**: Gemini CLI richiede Node >= 20. Il flag regex `v` (Unicode sets) non e supportato in Node 18.

```
file:///usr/local/lib/node_modules/@google/gemini-cli/node_modules/string-width/index.js:17
const zeroWidthClusterRegex = /^(?:\p{Default_Ignorable_Code_Point}|...)+$/v;
                              ^
SyntaxError: Invalid regular expression flags
```

**Soluzione**: Usare Node 22 via nvm (gia configurato come default).

```bash
export NVM_DIR="$HOME/.nvm" && . "$NVM_DIR/nvm.sh"
# Ora gemini funziona perche usa Node 22
```

### Errore: `claude: command not found` via SSH

**Causa**: SSH non-interattivo non carica `~/.bashrc`, quindi nvm non e nel PATH.

**Soluzione**: Sempre prefissare con il blocco nvm:

```bash
# SBAGLIATO
ssh ubuntu@server 'claude -p "task"'

# CORRETTO
ssh ubuntu@server 'export NVM_DIR="$HOME/.nvm" && . "$NVM_DIR/nvm.sh" && claude -p "task"'
```

### Errore: `ANTHROPIC_API_KEY not set`

**Causa**: I secrets non sono stati caricati.

**Soluzione**: Aggiungere `source /opt/easyway/.env.secrets` prima del comando CLI.

### Errore: `npm WARN EBADENGINE` durante installazione

**Causa**: Alcune dipendenze di Gemini CLI richiedono Node >= 20. Con Node 22 via nvm sono solo warning residui di pacchetti interni — non bloccanti.

**Soluzione**: Ignorabili se `gemini --version` funziona. Se persistono errori runtime, reinstallare sotto Node 22:

```bash
nvm use 22
npm install -g @google/gemini-cli
```

### Errore: Claude Code CLI bloccata con OpenRouter come ANTHROPIC_BASE_URL

**Causa**: Claude Code CLI usa il protocollo Anthropic Messages API (`/v1/messages`), non OpenAI-compatible (`/v1/chat/completions`). Impostando `ANTHROPIC_BASE_URL=https://openrouter.ai/api/v1` la CLI manda richieste nel formato Anthropic, OpenRouter risponde nel formato OpenAI, e la CLI si blocca in attesa indefinita.

**Soluzione**: Non usare OpenRouter come backend per Claude Code CLI. Le opzioni sono:
1. **Per CLI agent**: usare API key native (ANTHROPIC_API_KEY per Claude, GEMINI_API_KEY per Gemini)
2. **Per automazione multi-modello**: usare OpenRouter via curl/agent scripts con formato OpenAI-compatible

```bash
# SBAGLIATO — blocca la CLI
export ANTHROPIC_BASE_URL=https://openrouter.ai/api/v1
export ANTHROPIC_API_KEY=$OPENROUTER_API_KEY
claude -p "task"  # Si blocca!

# CORRETTO — OpenRouter via curl per automazione
source /opt/easyway/.env.secrets
curl -s https://openrouter.ai/api/v1/chat/completions \
  -H "Authorization: Bearer $OPENROUTER_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"model":"deepseek/deepseek-chat","messages":[{"role":"user","content":"task"}]}'
```

### Errore: NodeSource `setup_22.x` timeout

**Causa**: Il server OCI potrebbe avere latenza verso i repo NodeSource o blocchi di rete.

**Soluzione**: Usare il binary precompilato via nvm (metodo gia adottato) — nvm scarica direttamente da nodejs.org che e piu affidabile.

---

## Q&A

### Perche nvm e non binary overlay o apt upgrade?

**Consultazione GEDI** (principi applicati: G3 Reversibilita, G5 Trasparenza, G12 Convention):

| Approccio | Rischio | Problema |
|---|---|---|
| Binary overlay (`cp` in `/usr/local/bin/`) | Medio | Split-brain: apt pensa Node 18, reale e 22. Operatore futuro confuso |
| apt upgrade via NodeSource | Basso | Un solo Node globale, niente multi-versione |
| **nvm** (scelto) | Basso | Multi-versione, reversibile, convenzionale, trasparente |

nvm permette di avere Node 18 (compatibilita sistema) e Node 22 (per Gemini CLI) in parallelo, con switch pulito.

### Perche OpenRouter e non API key dirette?

1. **Una sola key** invece di 4+ (Anthropic, Google, DeepSeek, OpenAI)
2. **Model routing via manifest** — ogni agente sceglie il suo modello
3. **Principio G16 Presa Elettrica** — interfaccia standard, provider intercambiabile
4. **Costi visibili** — dashboard unica su openrouter.ai
5. **Fallback automatico** — se un provider e down, OpenRouter puo routare su alternativa

### I container Docker sono impattati dall'upgrade Node?

**No.** I container hanno il loro Node interno (definito nel Dockerfile). L'upgrade nvm impatta solo:
- Tool npm globali sul server host (`claude`, `gemini`)
- Script che girano direttamente sull'host (non in container)

I Dockerfile dei container specificano la propria versione Node (es. `FROM node:18-alpine`). Queste sono indipendenti dal Node dell'host.

### Dove documentare le versioni Node dei container?

Le versioni Node usate nei container sono gia definite nei rispettivi `Dockerfile`. Per un inventario centralizzato:

```bash
# Comando per estrarre versioni Node da tutti i Dockerfile
grep -r "FROM node" ~/easyway-*/Dockerfile ~/easyway-portal/Dockerfile 2>/dev/null
```

Per un registro strutturato futuro, vedi `easyway-wiki/infrastructure/factory.yml` che e gia la source of truth per l'infrastruttura.

### Posso usare la Claude Code CLI con OpenRouter?

**No direttamente.** Claude Code CLI usa il protocollo Anthropic Messages API, non OpenAI-compatible.
Impostare `ANTHROPIC_BASE_URL=https://openrouter.ai/api/v1` causa un blocco (la CLI aspetta risposte nel formato Anthropic che OpenRouter non fornisce).

**Alternativa testata e funzionante**: usare OpenRouter via curl/agent scripts con formato OpenAI-compatible:

```bash
source /opt/easyway/.env.secrets
curl -s https://openrouter.ai/api/v1/chat/completions \
  -H "Authorization: Bearer $OPENROUTER_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"model":"deepseek/deepseek-chat","messages":[{"role":"user","content":"task"}]}'
```

Questo e il modo corretto per automazione multi-modello sul server.

### Come si collega alle config nel repo agents?

Ogni agente ha un `manifest.json` con sezione `llm_config`:

```json
{
  "llm_config": {
    "provider": "openrouter",
    "model": "anthropic/claude-sonnet-4-6",
    "api_base": "https://openrouter.ai/api/v1",
    "temperature": 0.2,
    "max_tokens": 4096
  }
}
```

Il pattern `LLM_INTEGRATION_PATTERN.md` nel repo agents documenta come il runner legge il manifest e instrada la chiamata al provider configurato.

---

## Cronologia installazione (S90 — 2026-03-06)

### Passi eseguiti

1. Verifica stato server: Ubuntu 24.04 ARM64, Node 18.19.1 (apt), 20GB RAM, 131GB disco
2. Installazione Claude Code CLI v2.1.70 via `sudo npm install -g @anthropic-ai/claude-code` — OK immediato
3. Installazione Gemini CLI v0.32.1 via `sudo npm install -g @google/gemini-cli` — warning EBADENGINE (Node >= 20)
4. Test: `claude --version` OK, `gemini --version` FAIL (SyntaxError regex v flag)
5. **Tentativo**: binary overlay Node 22 in `/usr/local/bin/` — funzionava ma...
6. **Consultazione GEDI**: split-brain apt/reale = rischio medio. Raccomandazione: nvm
7. **Rollback**: rimosso binary overlay, tornati a Node 18 system
8. **Installazione nvm** v0.40.1 + Node 22.22.1
9. Reinstallazione CLI sotto nvm Node 22: entrambe funzionanti
10. Decisione architetturale: OpenRouter come gateway LLM unico (principio G16)
11. `OPENROUTER_API_KEY` aggiunta a `/opt/easyway/.env.secrets` (copiata da `.env.local` locale)
12. Test OpenRouter via curl + DeepSeek: **OK funziona** (346 modelli disponibili)
13. Test Claude Code CLI con OpenRouter: **FALLITO** — protocollo incompatibile (Anthropic API != OpenAI-compatible)
14. Scoperta: le CLI agent (Claude/Gemini) richiedono API key native; OpenRouter serve per agent scripts

### Errori incontrati e risolti

| # | Errore | Causa | Soluzione | Principio GEDI |
|---|---|---|---|---|
| 1 | Gemini CLI SyntaxError regex | Node 18 non supporta flag `v` | Upgrade a Node 22 via nvm | — |
| 2 | Binary overlay split-brain | `/usr/local/bin/node` shadowa `/usr/bin/node`, apt confuso | Rollback + nvm | G3 Reversibilita, G5 Trasparenza |
| 3 | NodeSource `setup_22.x` timeout | Latenza rete OCI verso deb.nodesource.com | nvm scarica da nodejs.org direttamente | G14 Adapt & Overcome |
| 4 | `npm WARN EBADENGINE` su Gemini deps | Dipendenze interne vogliono Node >= 20 | Non bloccante sotto Node 22 | G15 Known Bug over Chaos |
| 5 | Claude CLI bloccata con OpenRouter | Protocollo Anthropic != OpenAI-compatible | Usare OpenRouter solo per agent scripts; CLI richiedono key native | G16 Presa Elettrica (il protocollo e l'interfaccia!) |

---

## Prossimi passi

- [x] Aggiungere `OPENROUTER_API_KEY` a `/opt/easyway/.env.secrets`
- [x] Test OpenRouter via curl (DeepSeek): OK
- [ ] Aggiungere `ANTHROPIC_API_KEY` a `.env.secrets` per Claude Code CLI (se si vuole usare la CLI)
- [ ] Aggiungere `GEMINI_API_KEY` a `.env.secrets` per Gemini CLI (se si vuole usare la CLI)
- [ ] Aggiornare `LLM_INTEGRATION_PATTERN.md` con sezione OpenRouter e compatibilita protocolli
- [ ] Aggiornare manifest agenti con `api_base: openrouter` dove appropriato
- [ ] Creare connector `openrouter.sh` nel framework connessioni (`connections.yaml` ha gia entry, manca script)
- [ ] Creare wrapper script `~/bin/agent-run.sh` che legge manifest e invoca LLM via OpenRouter
