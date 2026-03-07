---
title: "levi-md — Guida Operativa Quickstart"
status: active
tags: [domain/levi, domain/obsidian, artifact/guide, tech/python, audience/dev]
created: "2026-03-07"
llm:
  include: true
  pii: none
---

# levi-md — Quickstart

## Cosa c'e e dove

| Elemento | Path | Note |
|----------|------|------|
| Pacchetto | `C:\old\easyway\levi-md\` | Repo locale |
| CLI entry | `levi_md/cli.py` | 9 comandi |
| Scanner core | `levi_md/scanner.py` | frontmatter, link, graph |
| Tag engine | `levi_md/tags.py` | taxonomy configurabile |
| Obsidian checks | `levi_md/obsidian.py` | .gitignore, hierarchy |
| Fix pipeline | `levi_md/fixer.py` | 4 Giri multi-pass |
| Discovery | `levi_md/discovery.py` | tag affinity, orphan homes |
| Index builder | `levi_md/index_builder.py` | MOC, tag indices, orphan |
| AI brain | `levi_md/brain.py` | OpenRouter/Ollama/OpenAI |

## Install

```bash
cd C:\old\easyway\levi-md
pip install -e .
```

Dopo l'install, il comando `levi` e disponibile ovunque.

## Ricette comuni

### Scan veloce del wiki

```bash
levi scan C:\old\easyway\wiki
levi scan C:\old\easyway\wiki --json > report.json
```

### Check Obsidian

```bash
levi check C:\old\easyway\wiki
```

### Fix completo (dry-run prima, poi apply)

```bash
# Prima vedi cosa farebbe
levi fix C:\old\easyway\wiki

# Poi applica
levi fix C:\old\easyway\wiki --apply
```

### Solo fix link (Giro 3)

```bash
levi fix C:\old\easyway\wiki --giri 0,3 --apply
```

### Scopri connessioni nascoste

```bash
levi discover C:\old\easyway\wiki
levi discover C:\old\easyway\wiki --json > connections.json
```

### Genera indici

```bash
levi index C:\old\easyway\wiki
# Output in wiki/_levi/ (vault-index.md, tags-*.md, orphans.md)
```

### CSS per tag colorati in Obsidian

```bash
levi css C:\old\easyway\wiki
# Output in .obsidian/snippets/levi-tag-colors.css
# Attivare in Obsidian: Settings > Appearance > CSS Snippets
```

### AI brain (richiede OPENROUTER_API_KEY)

```bash
export OPENROUTER_API_KEY=sk-or-v1-...

# Review qualita vault
levi ai review C:\old\easyway\wiki

# Suggerisci tag per un file
levi ai tag C:\old\easyway\wiki --file guides/levi-md-quickstart.md

# Trova connessioni per un file
levi ai connect C:\old\easyway\wiki --file chronicles/2026-03-06-levi-hierarchies-game.md
```

### Taxonomy custom (per studenti)

Crea `my-taxonomy.yml`:

```yaml
namespaces:
  course: [math, physics, history, literature, philosophy]
  type: [lecture-notes, essay, bibliography, summary, review]
  status: [draft, revision, final, published]
merges:
  notes: lecture-notes
  bib: bibliography
```

```bash
levi fix-tags ~/tesi/ --taxonomy my-taxonomy.yml --apply
```

## Troubleshooting

| Errore | Causa | Soluzione |
|--------|-------|-----------|
| `No API key configured` | Manca env var per AI brain | `export OPENROUTER_API_KEY=...` |
| `Not a directory` | Path vault errato | Verificare path passato a levi |
| Encoding errors su Windows | cp1252 default | levi forza UTF-8 automaticamente |
| Tag non migrati | Tag non nel taxonomy default | Creare taxonomy YAML custom |

## Per agenti esterni

Da qualsiasi directory:

```bash
python -m levi_md.cli scan /path/to/vault
```

O dopo `pip install -e C:\old\easyway\levi-md`:

```bash
levi scan /path/to/vault
```
