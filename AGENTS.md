# AGENTS.md — easyway-wiki

Istruzioni operative per agenti AI (Codex, Claude Code, Copilot Workspace, ecc.)
che lavorano in questo repository.

---

## Identità

**easyway-wiki** è la FONTE DI VERITA per tutta la documentazione della piattaforma EasyWay.
- Remote primario: Azure DevOps (`dev.azure.com/EasyWayData/EasyWay-DataPortal/_git/easyway-wiki`)
- GitHub: mirror read-only — MAI creare PR/push/branch su GitHub
- Branch strategy: `feat→main` o `docs/→main` (NO develop)
- Merge strategy: Merge (no fast-forward)

---

## Comandi rapidi

```bash
# Commit (con Iron Dome pre-commit scan)
ewctl commit

# Push feature branch
git push origin <branch>
```

---

## Struttura directory

```
guides/          # Guide operative (polyrepo-git-workflow, connection-registry, agent-ado-operations)
agents/          # Doc agenti + platform-operational-memory.md
chronicles/      # Cronache narrative sessione per sessione + _index.md
standards/       # Standard architetturali
security/        # Secrets governance
repos/           # Schede operative per-repo (una per repo)
planning/        # Initiatives backlog, epic taxonomy
vision/          # Manifesto, product map
Runbooks/        # Runbook operativi (RAG operations)
out/             # Output generati (non committare)
```

---

## Convenzioni per agenti

### Frontmatter obbligatorio
Ogni file `.md` DEVE avere frontmatter YAML:
```yaml
---
title: "Titolo pagina"
created: YYYY-MM-DD
status: active   # active | draft | archived
tags: [domain/xxx, process/yyy, language/it]
---
```

### Regole naming e struttura
- Tag `language/it` per contenuti in italiano
- Aggiornare `_index.md` della sezione quando si creano nuovi file
- MAI committare `.obsidian/` o file canvas
- Chronicles: `chronicles/YYYY-MM-DD-<slug>.md` + aggiornare `chronicles/_index.md`

### Session Closeout (obbligatorio)
Dopo ogni sessione di lavoro aggiornare:
1. `agents/platform-operational-memory.md` → sezione "Session N — COMPLETATA"
2. `chronicles/YYYY-MM-DD-<slug>.md` → cronaca narrativa
3. `planning/initiatives-backlog.md` → idee/task emersi

---

## File chiave

| File | Scopo |
|---|---|
| `agents/platform-operational-memory.md` | Memoria operativa della piattaforma (sync → .cursorrules) |
| `guides/connection-registry.md` | PAT, connettori, scadenze, routing |
| `guides/polyrepo-git-workflow.md` | Git governance G1-G14 |
| `guides/agent-ado-operations.md` | Operazioni ADO via agenti |
| `planning/initiatives-backlog.md` | Wishlist e backlog iniziative |
| `chronicles/_index.md` | Indice cronache |

---

## Regole assolute

- **MAI** modificare `guides/connection-registry.md` senza aggiornare anche MEMORY.md e `.cursorrules` dei repo coinvolti
- **MAI** creare file senza frontmatter
- **MAI** committare secrets o PAT
- **G13**: dopo ogni PR creata su ADO → `ado-auth.sh pr-autolink-wi <pr_id>`
- **G14**: nuove automazioni → skill in `easyway-agents/agents/skills/`, mai script locali

---

## Connessioni & PAT

- Guida completa: `guides/connection-registry.md`
- Gateway S88: tutte le API call passano via SSH server (`ubuntu@80.225.86.168`)
- PAT: SOLO su server `/opt/easyway/.env.secrets` — MAI in locale
- `.env.local` locale: solo `OPENROUTER_API_KEY` e `QDRANT_*`

---

## ADO ↔ GitHub — Regola `AB#`

Ogni commit e PR su GitHub deve referenziare il Work Item ADO:

```bash
git commit -m "feat: update chronicles format AB#1234"
# oppure nel body della PR: AB#1234
```

ADO mostra automaticamente il link alla PR/commit GitHub sul WI.
**MAI creare PR senza Work Item ADO** — vale anche su GitHub (Regola del Palumbo).
