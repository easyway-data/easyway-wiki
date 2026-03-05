# MVP Maturity Checklist

A framework for evaluating whether an MVP is ready for progressively wider audiences.
Applicable to any project — easyway-ado, HALE-BOPP engines, future tools.

## The Three Circles Model

Every project starts private and earns its way outward:

```
  Circle 3 (Private)        Circle 2 (Source-Available)      Circle 1 (Open Source)
  "works for us"       -->  "look how we do it"         -->  "use it yourself"
```

**Promotion rule:** a project moves to the next circle only when ALL questions at the corresponding maturity level are answered YES.

---

## Level 1 — Works for someone who isn't us

The minimum bar for leaving Circle 3. Can a stranger pick it up?

| # | Question | How to verify |
|---|----------|---------------|
| 1.1 | **Can a new user get it running in < 10 minutes?** | Someone follows only the README, no live help |
| 1.2 | **Does it work on a different environment than ours?** | Test on a second org/server/DB (even a trial/free tier) |
| 1.3 | **Zero hardcoded values in core code?** | Grep: no org-specific strings in library/SDK code |
| 1.4 | **Does the main entry point respond to a health check?** | Start the tool, call its simplest operation, get a clear OK/FAIL |

**Passes Level 1 →** safe to share with trusted collaborators.

---

## Level 2 — It's reliable

Not just "it works", but "it doesn't break in your hands".

| # | Question | How to verify |
|---|----------|---------------|
| 2.1 | **Do automated tests pass?** | `npm test` / `pytest` / `pester` — unit + smoke |
| 2.2 | **Does it handle bad credentials with a clear error?** | Test with expired/invalid token — human-readable message, no stack trace |
| 2.3 | **Does it fail gracefully under failure?** | Max retry limit, timeout, no infinite loops, no brute-force |
| 2.4 | **Does it work on the target platforms?** | CI cross-platform OR manual test (Windows + Linux at minimum) |

**Passes Level 2 →** safe to open as Circle 2 (source-available).

---

## Level 3 — It's useful

The tool delivers real value, not just theoretical potential.

| # | Question | How to verify |
|---|----------|---------------|
| 3.1 | **Is working WITH the tool measurably better than WITHOUT it?** | Side-by-side comparison: same task, with and without the tool |
| 3.2 | **Do the examples/docs cover 80% of common use cases?** | A typical user finds what they need without reading source code |
| 3.3 | **Can it complete a real workflow end-to-end autonomously?** | Run a full scenario (e.g., briefing + query + action) without manual intervention |

**Passes Level 3 →** ready for public beta / early adopters.

---

## Level 4 — It's presentable

Polish for public release. The difference between a tool and a product.

| # | Question | How to verify |
|---|----------|---------------|
| 4.1 | **Does it have a CHANGELOG with semantic versioning?** | `CHANGELOG.md` with release notes per version |
| 4.2 | **Does it have a clear license?** | Apache 2.0, MIT, or equivalent — no ambiguity |
| 4.3 | **Can it be installed with one command?** | `npm install`, `pip install`, or installable from git URL |
| 4.4 | **Does it have at least 3 documented use cases with real evidence?** | Not "it could do X" but "we did X in session Y, here's what happened" |

**Passes Level 4 →** ready for Circle 1 (open source, package registry, tell the world).

---

## Applying the Checklist

### Current project status

| Project | Level 1 | Level 2 | Level 3 | Level 4 | Circle |
|---------|:-------:|:-------:|:-------:|:-------:|:------:|
| **hale-bopp-db** | 3/4 | 3/4 | 2/3 | 1/4 | 1 (early) |
| **hale-bopp-etl** | 3/4 | 3/4 | 2/3 | 1/4 | 1 (early) |
| **hale-bopp-argos** | 3/4 | 3/4 | 1/3 | 1/4 | 1 (early) |
| **easyway-ado** | 1/4 | 0/4 | 0/3 | 0/4 | 3 |

> Update this table as projects evolve. Each session that adds tests, docs, or cross-env validation ticks boxes.

### How to use during development

1. **At sprint planning**: pick which maturity questions to tackle this sprint
2. **At session closeout**: update the status table above
3. **Before promoting circle**: verify ALL questions at the target level are YES
4. **For new projects**: copy this checklist into the project repo as a living document

### Anti-patterns

- **Don't skip levels.** A project at Level 4 polish but failing Level 2 reliability is a liability, not a product.
- **Don't self-assess alone.** Level 1 ("works for someone else") requires someone else. Ask a colleague, use a fresh machine, or test on a trial environment.
- **Don't rush to open source.** Circle 3 → Circle 1 in one jump skips the hardening that makes a tool trustworthy. Earn each circle.

---

## Origin

This framework was born from the EasyWay platform development experience (75+ sessions). Every question comes from a real failure or lesson learned — not theory.

> "Non fuffa, evidenze concrete." — Session 75
