---
id: ew-levi-md
title: "levi-md — Markdown Vault Quality Scanner"
summary: Standalone CLI tool for scanning, fixing, and enriching Markdown vaults. Obsidian-native. Zero dependencies. AI brain via OpenRouter.
status: alpha
owner: team-platform
created: "2026-03-07"
tags: [levi-md, domain/levi, domain/obsidian, tech/python, artifact/documentation, process/quality, audience/dev]
circle: 1
language: Python
repo_url_github: https://github.com/easyway-data/levi-md
repo_url_ado: https://dev.azure.com/EasyWayData/EasyWay-DataPortal/_git/levi-md
local_path: C:\old\easyway\levi-md
llm:
  include: true
  pii: none
  chunk_hint: 300
---

# levi-md

Markdown vault quality scanner, fixer, and AI brain for Obsidian and documentation teams.

Extracted from EasyWay's internal Levi agent (v2.2) as a standalone, zero-dependency Python package.

## Target audience

- Students (thesis vaults, lecture notes, research)
- Researchers (knowledge graphs, bibliographies)
- Documentation teams (wiki quality gates)
- Obsidian users (vault hygiene)

## 9 CLI Commands

| Command | What it does | Modifies files? |
|---------|-------------|:---:|
| `levi scan` | Quality scan: frontmatter, links, sections, wikilinks | No |
| `levi check` | Obsidian health: .gitignore, hierarchy, naming, canvas | No |
| `levi fix` | **4-Giri pipeline**: frontmatter + tags + links + obsidian | Yes (--apply) |
| `levi fix-tags` | Tag migration flat to namespaced with custom taxonomy | Yes (--apply) |
| `levi discover` | Hidden connections: tag affinity, orphan homes, bridges | No |
| `levi index` | Generate MOC, per-namespace tag indices, orphan index | Creates files |
| `levi css` | Obsidian CSS snippet for colored tags | Creates file |
| `levi graph` | Relationship graph as JSON | No |
| `levi ai` | LLM brain: review, tag, connect, frontmatter | No |

## The 4 Giri (Fix Pipeline)

```
Giro 0: Inventory   -> file index, frontmatter map, snapshot state
Giro 1: Frontmatter -> add missing --- title/tags --- blocks
Giro 2: Tags        -> migrate flat to namespaced (configurable taxonomy)
Giro 3: Links       -> fix stale links using file index ("did you mean?")
Giro 4: Obsidian    -> .gitignore, Untitled.canvas, wikilink repair
```

Each giro reads the state left by the previous one. No stale data.

## AI Brain

Connects to any OpenAI-compatible API (OpenRouter default, also Ollama, Claude, etc.):

```bash
export OPENROUTER_API_KEY=sk-or-v1-...
levi ai review ~/vault/
levi ai tag ~/vault/ --file notes/lecture-01.md
levi ai connect ~/vault/ --file thesis/chapter-3.md
levi ai frontmatter ~/vault/ --file raw-notes.md
```

Default model: `deepseek/deepseek-chat` via OpenRouter.

## Install

```bash
pip install levi-md           # from PyPI (when published)
pip install -e .              # from source
```

## Tech stack

- Python 3.9+ (zero external dependencies)
- Simple regex YAML parser (no PyYAML needed)
- urllib for LLM API calls (no requests needed)

## Origin

Extracted from `easyway-agents/scripts/levi-scan.py` (Levi v2.2, Session 97-98).
The internal Levi agent remains in easyway-agents for EasyWay-specific polyrepo scanning.
levi-md is the generic, portable version for any Markdown vault.

## Relation to EasyWay

- **Circle 1** (open source) — designed for external use
- Internal Levi agent uses levi-md core + EasyWay-specific extensions (polyrepo, repo cards, ADO integration)
- Tag taxonomy is configurable via YAML — EasyWay uses its own, users bring theirs
