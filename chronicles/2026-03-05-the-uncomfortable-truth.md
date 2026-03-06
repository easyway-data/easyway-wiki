---
title: "The Uncomfortable Truth"
date: 2026-03-05
category: vision
session: S75
tags: [manifesto, product-map, competitive-analysis, philosophy, easyway-ado, domain/mcp]
---

# S75 — The Uncomfortable Truth

## The session where we stopped building and started seeing

Session 75 was supposed to be about consolidating ADO scripts into easyway-ado. It became the session where we looked at what we'd built across 75 sessions and realized it was ten products, not one.

## What happened

**easyway-ado bootstrap.** We cloned the empty ADO repo, bootstrapped it with TypeScript project scaffold, platform config, connection registry, and a full English README with PAT routing map, environment setup, and example queries. PR #324, merged.

**The MCP question.** During planning, the question came: "should easyway-ado have an MCP server?" The answer was obvious — yes, 20 tools mapped from existing scripts, exposable to any AI agent via Model Context Protocol. stdio for Claude Code, HTTP for n8n. Planned for Phase 3.

**The maturity question.** "When will it be ready for the world?" This forced us to write a universal MVP Maturity Checklist — 4 levels (works, reliable, useful, presentable), 3 circles (private, source-available, open source). Applicable to everything, not just easyway-ado.

**The product map.** Mapping what we'd built: 10 extractable products from one project. Three HALE-BOPP engines, GEDI, Iron Dome, ewctl, the Maturity Checklist itself, an Agentic Playbook, easyway-ado, and the agent platform.

**The competitive analysis.** Honest look at the market. Most categories are crowded (Atlas, Kestra, Gitleaks, CrewAI). But GEDI is blue ocean — nobody builds an advisory ethical framework for AI coding agents. And the Agentic Playbook backed by 75 real sessions has no equivalent.

**The uncomfortable truth.** The industry sells complexity. We built everything on a free ARM server from Oracle Cloud. 4 cores, 24GB, zero cost. The frameworks we build are grandma's good advice in reusable packaging. "Non mandiamo shuttle sulla luna."

**The manifesto.** Written and merged. The story of how a data portal became a laboratory for agentic development. "We started building a data portal. We ended up somewhere else."

## Artifacts

| PR | Repo | Content |
|----|------|---------|
| #324 | easyway-ado | Bootstrap: README, package.json, tsconfig, config, docs/examples |
| #325 | wiki | Abandoned (superseded by #326) |
| #326 | wiki | Vision section: manifesto + product map + maturity checklist |
| #327 | wiki | Philosophy + competitive analysis update |

## The quote

> "Non mandiamo shuttle sulla luna. Facciamo cose semplici mettendo una falsa maschera di complessita. Vendiamo framework che erano i buoni consigli della nonna."
