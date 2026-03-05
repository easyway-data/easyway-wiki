---
id: repo-easyway-ado
title: 'Repo: easyway-ado'
summary: Scheda operativa del repository easyway-ado — TypeScript SDK, CLI e MCP Server per Azure DevOps.
status: active
owner: team-platform
created: '2026-03-05'
updated: '2026-03-05'
tags: [easyway-ado, repos, circle-2, domain/tooling, layer/sdk, audience/dev, privacy/internal, language/it]
entities: []
llm:
  include: true
  pii: none
  chunk_hint: 300
type: reference
---

# easyway-ado

SDK TypeScript, CLI e MCP Server per Azure DevOps. PAT routing, work items, PR, briefing, governance — consumabile da AI agents via MCP.

## Anagrafica

| Campo | Valore |
|-------|--------|
| **ADO repo** | `easyway-ado` (GUID: `91b146f8`) |
| **GitHub** | Futuro: `easyway-data/easyway-ado` (quando Circle 1) |
| **Cerchio** | 2 (Source-available) |
| **Linguaggio** | TypeScript |
| **CI/CD** | Da configurare |
| **Branch protetti** | `main` |

## Struttura locale

```
C:\old\easyway\ado\
```

## Componenti chiave

- **CLI**: 10 comandi (`briefing`, `wi get/list/query/create/update`, `pr list/get/create/policy`)
- **MCP Server**: `mcp/index.ts` — 10 tool stub + zod validation
- **PAT routing**: `ado-client.ts` — router automatico per scope (pr/wi/build/github/general)
- **Guardrails**: Palumbo enforcement (WI obbligatorio), feat-to-main guard, duplicate PR guard
- **Notification formatter**: output strutturato per chat channels

## Dipendenze

- **ado-auth.sh**: `agents/scripts/ado-auth.sh` per PAT routing bash
- **C:\old\.env.local**: 4 PAT con scope segregati

## Roadmap

- Phase 0-2: completate (S75-S76)
- Phase 3: MCP server full (resources, prompts, remaining tools)
- Phase 4: guardrails configurabili `.guardrails.yml`
- Phase 5: cleanup + GitHub mirror + npm publish

## Scelte tecniche

- **Python per batch API ADO**: vedi [ADR: Python per ADO API](../architecture/adr-python-ado-api.md)
