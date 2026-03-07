---
id: ew-marginalia
title: "marginalia — Markdown Vault Quality Scanner & Linker"
summary: Standalone CLI for scanning, fixing, evaluating, and linking Markdown vaults. Obsidian-native. TF-IDF linker. Eval snapshots. Zero dependencies.
status: alpha
owner: team-platform
created: "2026-03-07"
tags: [marginalia, domain/levi, domain/obsidian, tech/python, artifact/documentation, process/quality, audience/dev]
circle: 1
language: Python
repo_url_github: https://github.com/hale-bopp-data/marginalia
local_path: C:\old\easyway\marginalia
llm:
  include: true
  pii: none
  chunk_hint: 300
---

# marginalia

Markdown vault quality scanner, fixer, evaluator, and smart linker for Obsidian, academics, and documentation teams.

Evolution of levi-md. Renamed to marginalia (S97) for broader audience and clearer identity.

## Key capabilities

| Command | What it does | Modifies files? |
|---------|-------------|:---:|
| `marginalia scan` | Quality scan: frontmatter, links, sections, wikilinks | No |
| `marginalia check` | Obsidian health: .gitignore, hierarchy, naming, canvas | No |
| `marginalia fix` | 4-Giri pipeline: frontmatter + tags + links + obsidian | Yes (--apply) |
| `marginalia fix-tags` | Tag migration flat to namespaced with custom taxonomy | Yes (--apply) |
| `marginalia discover` | Hidden connections: tag affinity, orphan homes, bridges | No |
| `marginalia index` | Generate MOC, per-namespace tag indices, orphan index | Creates files |
| `marginalia css` | Obsidian CSS snippet for colored tags | Creates file |
| `marginalia graph` | Relationship graph as JSON | No |
| `marginalia ai` | LLM brain: review, tag, connect, frontmatter | No |
| `marginalia eval` | Snapshot-based health evaluation with queries and verdicts | No |
| `marginalia link` | TF-IDF smart linking suggestions between docs | No |

## New in marginalia (vs levi-md)

- **eval**: snapshot vault health, run queries (YAML), compare over time, produce HEALTHY/DEGRADED verdicts
- **linker**: TF-IDF similarity + tag overlap + directory proximity for smart link suggestions
- **config**: `.marginalia.yml` per-vault configuration
- **tests**: 48 tests (pytest), CI via GitHub Actions

## Tech stack

- Python 3.9+ (zero external dependencies for core)
- Simple regex YAML parser (no PyYAML needed)
- TF-IDF with pure Python (no numpy/sklearn)
- urllib for LLM API calls (no requests needed)

## Branch strategy

- **main**: current working branch (alpha, direct push OK)
- **develop**: da creare quando il progetto raggiunge beta — da quel punto, feature branches + PR obbligatorie

## Relation to EasyWay

- **Circle 1** (open source) — designed for external use
- Internal Levi agent uses marginalia core + EasyWay-specific extensions (polyrepo, repo cards, ADO integration)
- marginalia eval e il motore del Wiki Health Monitor (Gnosis L3, GEDI Case #42)
- Tag taxonomy configurable via YAML — EasyWay uses its own, users bring theirs

## Origin

Extracted from `easyway-agents/scripts/levi-scan.py` (Levi v2.2).
Renamed from levi-md to marginalia in Session 97.
