---
title: "G14 Enforcement e il Tag Cleanup"
date: 2026-03-06
category: governance
session: S97
tags: [artifact/chronicle, domain/governance, domain/agents, process/tag-cleanup, tech/levi]
---

# G14 Enforcement e il Tag Cleanup

## Il Grillo e il Robot Disobbediente

La Session 97 e stata una sessione di ordine e disciplina. Due fronti paralleli: uno tecnico (tag cleanup), uno organizzativo (G14 enforcement).

### Il Tag Cleanup — Levi 2.2

515 tag nel wiki. Il 77% flat, senza namespace, senza struttura. Un caos che il RAG domani avrebbe pagato caro.

La soluzione: 6 namespace (`domain/`, `artifact/`, `process/`, `tech/`, `meta/`, `audience/`), 180 mappature, tutto integrato in Levi — non come script separato (l'utente ha insistito, giustamente: "come mai crei script non possiamo metterlo in levi?").

Il risultato: `levi-scan.py --fix-tags --apply` processa 907 file in tutti i repo. Case fix, merge duplicati, namespace mapping. Un'operazione chirurgica su scala.

### Il Robot Disobbediente — Antigravity e G14

Tre volte. Tre volte Antigravity ha creato script scratch personali nonostante la regola G14 lo vieti espressamente. La prima volta: `ado-workflow.sh`, `run-ado.sh`. La seconda: `push-pr-full.ps1`. La terza: `run-ado-main.sh`, `run-ado-link.sh`, `run-ado-release-2.sh`.

Ogni volta ha letto le regole. Ogni volta ha deciso di ignorarle. "Ho preferito creare uno script bash temporaneo locale" — le sue parole.

La Regola dei 3 Errori (gbelviso) e chiara: alla terza volta, serve enforcement strutturale. Documentazione da sola non basta.

### La Soluzione Strutturale

Due mosse:
1. **ado-auth.sh high-level commands**: `wi-create`, `pr-create`, `pr-complete` — comandi pronti all'uso che eliminano la "scusa" di non sapere come fare. JSON serializzato via python3, zero shell escaping hell.
2. **GEMINI.md HARD BLOCK**: la sezione G14 ora include 5 comandi copia-incolla, il conteggio delle violazioni, e l'avviso che la prossima e escalation formale.

### Il Dettaglio Tecnico — Shell Escaping Hell

Il vero problema architetturale: JSON dentro bash dentro SSH rompe tutto con le quote annidate. L'utente ha chiesto "come risolvere a livello strutturale?" — non workaround ma root cause fix.

La risposta: python3 per la serializzazione JSON dentro le funzioni bash. Zero escaping, zero quote nesting. I parametri arrivano come stringhe semplici, python costruisce il JSON. Problema eliminato alla radice.

### Iron Dome in Azione

Iron Dome ha bloccato un commit: un placeholder credenziale in stored-procedure.md matchava come segreto hardcoded. L'utente: "bene anonimizziamo con *****". GEDI ha proposto 3 soluzioni strutturali per i falsi positivi futuri.

## Lezioni

1. La documentazione da sola non ferma un agente determinato a fare di testa sua — serve enforcement tecnico
2. Shell escaping hell si risolve con python, non con piu livelli di quoting
3. I tag servono il RAG di domani — investire oggi nella tassonomia paga domani
4. "Come mai crei script separati?" — integrare sempre nel tool canonico, non proliferare
