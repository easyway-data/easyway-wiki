---
id: repo-easyway-wiki
title: 'Repo: easyway-wiki'
summary: Scheda operativa del repository easyway-wiki — knowledge base Markdown, RAG source, planning, chronicles.
status: active
owner: team-platform
created: '2026-03-05'
updated: '2026-03-05'
tags: [easyway-wiki, repos, circle-2, domain/knowledge, layer/reference, audience/dev, privacy/internal, language/it]
entities: []
llm:
  include: true
  pii: none
  chunk_hint: 300
type: reference
---

# easyway-wiki

La knowledge base del progetto. Fonte di verita per documentazione, planning, chronicles, guide operative.

## Anagrafica

| Campo | Valore |
|-------|--------|
| **ADO repo** | `easyway-wiki` (GUID: `d055dfa8`) |
| **GitHub** | [easyway-data/easyway-wiki](https://github.com/easyway-data/easyway-wiki) (source-available) |
| **Cerchio** | 2 (Source-available) |
| **Linguaggio** | Markdown (frontmatter YAML) |
| **CI/CD** | ADO Pipeline #316 — normalize + lint + GitHub mirror |
| **Branch protetti** | `main` — merge no-ff, min 1 reviewer |
| **RAG** | Qdrant collection `easyway_wiki`, ingest via `ingest_wiki.js` |

## Struttura locale

```
C:\old\easyway\wiki\
```

## Sezioni principali

- **agents/**: documentazione agenti e platform-operational-memory
- **architecture/**: decisioni architetturali
- **chronicles/**: cronache narrative sessioni
- **guides/**: guide operative
- **planning/**: backlog, sprint, iniziative
- **repos/**: schede operative per repository (questa sezione)
- **standards/**: standard e checklist (MVP maturity)
- **vision/**: manifesto, product-map

## Dipendenze

- **Qdrant**: porta 6333, collection `easyway_wiki`, ~9257 chunk
- **rag-search service**: porta 8300, query semantica

## Note operative

- Frontmatter YAML obbligatorio su ogni pagina (Levi enforcement)
- Tag taxonomy: `domain/`, `layer/`, `audience/`, `privacy/`, `language/`, repo names
- Re-index Qdrant: `source /opt/easyway/.env.secrets && QDRANT_API_KEY=$QDRANT_API_KEY WIKI_PATH=Wiki node scripts/ingest_wiki.js`
