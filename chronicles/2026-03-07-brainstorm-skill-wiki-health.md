---
title: "Il Brainstorm e il Battito della Wiki"
date: 2026-03-07
category: milestone
session: S99
tags: [skill, brainstorm, wiki-health, marginalia, antifragile, SDLC]
---

# Il Brainstorm e il Battito della Wiki

> "Prima di costruire, guarda dove metti i piedi." — Proverbio

## Il contesto

La sessione 98 aveva lasciato una promessa: una skill chiamata `process.workspace-brainstorm`, registrata nel registry come "planned", che avrebbe dovuto diventare il gate obbligatorio prima di ogni sviluppo Full-Track. E tre PR da approvare — il frutto di una giornata intensa di Gnosis Framework, Wiki Health Monitor e semafori.

La sessione 99 raccoglie entrambe le eredita.

## La skill prende vita

Il workspace-brainstorm nasce come uno skill deterministico — niente LLM, niente magia. Scansiona la wiki cercando documentazione per lo scope richiesto, rileva le dipendenze cross-repo leggendo le keyword della factory, interroga ADO per i work item esistenti. Poi calcola gaps, rischi, dipendenze, suggerisce un roster di agenti, e emette un verdetto: GO, CAUTION, o STOP.

Quattro scenari testati: un DryRun per vedere cosa farebbe, una descrizione naturale ("Epic: Gnosis Framework" — trova 11 documenti, verdict GO), un path wiki diretto, e un PBI inesistente che giustamente risponde CAUTION.

Il momento piu interessante e stato l'antifragilita. PowerShell ha un difetto noto: gli array vuoti sono "falsy", e `ConvertTo-Json` li trasforma in null. Invece di patchare caso per caso, nasce `ConvertTo-SafeArray` — un helper unico che garantisce che qualsiasi input diventi un array JSON-safe. Mai null, mai sorprese. Un guardrail costruito alla prima occorrenza, come insegna la regola dei 3 errori.

GEDI ha illuminato la strada: verbo approvato PowerShell (`ConvertTo-` invece di `Ensure-`), nessun path hardcodato per i secrets (solo env var dal framework connections), e la consapevolezza che il brainstorm non deve bloccare ma informare — almeno nella v1.

## Il battito della wiki

Con le PR mergiate, era il momento di misurare il battito. Il secondo snapshot del Wiki Health Monitor — snap-S99 — confrontato con il baseline S98.

Verdetto: NEUTRAL. La wiki non e degradata. 608 documenti (+1), coverage stabile al 100%, recall al 73%. La query "Gnosis framework" migliora di +0.048 — i nuovi documenti vengono trovati. Nessuna regressione.

E un buon segno. Il sistema nervoso della piattaforma regge.

## Lezioni

1. **Determinismo prima, LLM dopo**: il brainstorm v1 non usa intelligenza artificiale. Scansiona file, conta sezioni, cerca keyword. E gia utile cosi. Il LLM arrivera in v2 come enrichment, non come fondamento.

2. **Antifragilita come cultura**: `ConvertTo-SafeArray` non e solo un fix tecnico. E il principio che ogni errore genera un guardrail permanente — costruito subito, non alla terza volta.

3. **Documentare i tool come li usi**: marginalia aveva la sintassi giusta nel codice, ma nessuno l'aveva scritta dove conta — nel .cursorrules e nel tooling.md. Ora c'e, con alias e attenzione ai gotcha.

---

*Session 99 — dove una skill diventa reale e la wiki conferma di stare bene.*
