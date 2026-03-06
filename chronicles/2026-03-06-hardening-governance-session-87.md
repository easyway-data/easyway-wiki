---
title: "Session 87 — Il Fortino e le Regole"
date: 2026-03-06
category: infrastructure
session: S87
tags: [hardening, docker-compose, governance, n8n, deploy, security]
---

# Session 87 — Il Fortino e le Regole

> La sessione in cui il server diventa fortezza e il polyrepo trova le sue regole di convivenza.

## Il contesto

La S86 aveva lasciato due PR pendenti e un server con n8n ancora da configurare. Ma la vera urgenza veniva dalla S85: un collega aveva bypassato il deploy workflow via SCP, aprendo una falla nel processo. Serviva chiudere il cerchio — hardening reale, non solo documentato.

## La mattina delle PR

Le prime ore sono state di consolidamento. La PR #358 (infra compose con volume n8n e Docker socket) era gia pronta — merge pulito. La PR #359 (wiki) invece aveva 3 conflitti: inventory, security framework, n8n card. Risolti uno a uno, verificando che nessun contenuto andasse perso. Merged.

## Il Fortino

Il cuore della sessione: PR #360 con gli hardening scripts. Un deploy user segregato con ForceCommand che permette SOLO l'esecuzione di `deploy-shell.sh` — niente shell interattiva, niente SCP, niente SFTP. RBAC configurato, sudoers blindato. Il bypass della S85 non puo piu ripetersi.

Sul server, n8n ha ricevuto il suo volume dedicato (`easyway-n8n`) e il Docker socket read-only per i workflow di monitoring. I due workflow infra — Health Report e Census Watchdog — sono stati importati e sono pronti per l'attivazione (richiedono SMTP per gli alert).

## Le Regole del Gioco

Con 9 repository che condividono un ecosistema Docker Compose, servivano regole chiare. Sono nate le 6 Docker Compose Governance Rules: naming rete, volume mount, profiles, build context, env files, override precedence. Non sono burocrazia — sono la differenza tra un polyrepo che funziona e uno che esplode al primo deploy incrociato.

## Il backlog si arricchisce

5 nuove iniziative sono entrate nel backlog:
- **Multi-agent conflict prevention on main** — il problema dei conflitti PR quando piu agenti lavorano in parallelo
- **Email service sovereign** — SMTP proprio, non dipendente da provider esterni
- **La Valigetta** — la piattaforma portatile, un tar.gz che ricostruisce tutto su qualsiasi server
- **Manifesto messaging review** — allineare tutto a "sovereign by design, cloud by choice"
- **Docker Compose Governance Rules** — le 6 regole appena documentate

## Il bilancio

Una sessione di consolidamento e governance. Niente feature nuove, ma infrastruttura piu solida e regole che prevengono i problemi futuri. Il fortino e chiuso, le regole scritte. La piattaforma cresce non solo in funzionalita, ma in maturita.
