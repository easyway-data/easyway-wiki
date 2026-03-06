---
id: adr-python-ado-api
title: 'ADR: Python come linguaggio default per chiamate batch ADO API'
summary: Decisione architetturale — usare Python invece di bash/curl per interazioni batch con Azure DevOps REST API.
status: accepted
owner: team-platform
created: '2026-03-05'
updated: '2026-03-05'
tags: [easyway-ado, easyway-agents, domain/architecture, artifact/adr, domain/tooling, layer/sdk, audience/dev, privacy/internal, language/it]
entities: []
llm:
  include: true
  pii: none
  chunk_hint: 300
type: architecture
---

# ADR: Python come linguaggio default per chiamate batch ADO API

## Stato

**Accepted** — 2026-03-05 (Session 81)

## Contesto

Le interazioni con Azure DevOps REST API avvengono in due modalita:

1. **Singola chiamata** (es. creare 1 PR, aggiornare 1 WI) — OK con curl
2. **Batch/loop** (es. chiudere 12 PBI, creare 9 WI con parent link, WIQL + fetch dettagli) — problematico con curl

Il pattern bash/curl per batch ha mostrato ripetutamente questi problemi:

- **Quoting fragile**: JSON con apici singoli dentro heredoc bash causa `unexpected EOF` (Session 81)
- **Rate limiting**: loop `for id in $IDS; do curl ...` satura l'API con chiamate singole, risposte vuote
- **Parsing JSON**: `python3 -c` inline e illeggibile e non gestisce errori
- **Manutenibilita**: un singolo comando bash con 5 pipe e incomprensibile

## Decisione

**Usare Python (urllib.request / json) per tutte le interazioni batch con ADO API.**

Bash/curl resta valido per:
- Chiamate singole semplici (1 GET, 1 PATCH)
- Health check rapidi
- Script gia esistenti in `ado-auth.sh` (PAT routing)

## Motivazioni

| Criterio | bash/curl | Python |
|----------|-----------|--------|
| Quoting JSON | Fragile (escape hell) | Nativo (`json.dumps`) |
| Error handling | Exit code + grep | Try/except + status code |
| Batch requests | Loop singolo, rate limit | Array + batch endpoint |
| Leggibilita | Bassa su operazioni complesse | Alta |
| Dipendenze | Zero (built-in) | Zero (stdlib: urllib, json) |
| Velocita sviluppo | Lenta per batch | Rapida |
| Debugging | `echo` + `set -x` | `print` + stack trace |

## Esempio concreto

### Prima (bash — fallito S81)

```bash
for id in 42 43 44; do
  curl -s -X PATCH -H "Authorization: Basic $B64" \
    -H "Content-Type: application/json-patch+json" \
    -d '[{"op":"replace","path":"/fields/System.State","value":"Done"}]' \
    "https://dev.azure.com/.../workitems/$id?api-version=7.1"
done
```

Problemi: quoting JSON dentro bash, nessun error handling, nessun output strutturato.

### Dopo (Python — funzionante S81)

```python
import urllib.request, json

b64 = '...'  # da ado-auth.sh
base = 'https://dev.azure.com/.../workitems'

for parent_id, title, desc in pbis:
    body = [{'op':'add','path':'/fields/System.Title','value':title}, ...]
    req = urllib.request.Request(url, json.dumps(body).encode(), method='POST')
    req.add_header('Authorization', f'Basic {b64}')
    req.add_header('Content-Type', 'application/json-patch+json')
    resp = urllib.request.urlopen(req)
    result = json.loads(resp.read())
    print(f'OK #{result["id"]}: {title}')
```

## Conseguenze

- Gli script batch ADO usano Python con `urllib.request` (stdlib, zero dipendenze)
- Il PAT routing resta in bash (`ado-auth.sh`) — Python lo chiama via subprocess o legge `.env.local`
- easyway-ado (TypeScript) resta il SDK strutturato per integrazioni persistenti
- Python e per operazioni batch one-shot o scripting agente

## Alternativa scartata

- **jq + curl**: migliora il parsing ma non risolve quoting e rate limiting
- **PowerShell Invoke-RestMethod**: funziona ma aggiunge dipendenza PS su Linux
- **Node.js/fetch**: possibile ma overkill per scripting — gia coperto da easyway-ado per casi strutturati
