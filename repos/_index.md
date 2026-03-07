---
id: ew-repos-index
title: Repository Registry
summary: Scheda operativa per ogni repository del progetto. Ogni repo name e un tag riutilizzabile nella wiki.
status: active
owner: team-platform
created: '2026-03-05'
updated: '2026-03-06'
tags: [process/repos, registry, domain/platform, layer/reference, audience/dev, privacy/internal, language/it]
entities: []
llm:
  include: true
  pii: none
  chunk_hint: 300
type: reference
---

# Repository Registry

> Ogni repository ha una scheda operativa in questa sezione.
> Il nome del repo (es. `easyway-portal`, `hale-bopp-db`) e un **tag di prima classe** usabile nel frontmatter di qualsiasi pagina wiki per cross-referencing.

## Come usare i tag repo

Nel frontmatter di una pagina wiki, guide, chronicle o planning:

```yaml
tags: [easyway-agents, hale-bopp-db, domain/platform]
```

Questo crea un legame navigabile tra la pagina e i repo coinvolti.

## Mappa dei repository

| Repo | Cerchio | Linguaggio | Stato | Scheda |
|------|:-------:|------------|:-----:|--------|
| easyway-portal | 3 (Private) | Node.js / Express | Production | [scheda](easyway-portal.md) |
| easyway-agents | 2 (Source-available) | PowerShell / Python / bash | Production | [scheda](easyway-agents.md) |
| easyway-infra | 3 (Private) | Docker / bash | Production | [scheda](easyway-infra.md) |
| easyway-wiki | 2 (Source-available) | Markdown | Production | [scheda](easyway-wiki.md) |
| easyway-ado | 2 (Source-available) | TypeScript | Phase 2 | [scheda](easyway-ado.md) |
| easyway-n8n | 3 (Private) | JSON (n8n) | Production | [scheda](easyway-n8n.md) |
| hale-bopp-db | 1 (Open source) | Python | Running | [scheda](hale-bopp-db.md) |
| hale-bopp-etl | 1 (Open source) | Python | Running | [scheda](hale-bopp-etl.md) |
| hale-bopp-argos | 1 (Open source) | Python | Running | [scheda](hale-bopp-argos.md) |
| marginalia | 1 (Open source) | Python | Alpha | [scheda](marginalia.md) |

## Relazione con product-map

- **product-map.md** (`vision/`) = vista strategica (cosa estrarre, maturity, mercato)
- **repos/** = vista operativa (URL, CI, deploy, connessioni, tech stack)

Le due viste sono complementari. La product-map guida le decisioni di business, le schede repo guidano lo sviluppo quotidiano.
