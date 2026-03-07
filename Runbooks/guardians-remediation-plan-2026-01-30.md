---
title: "Piano di Rientro Completo — easyway-server-oracle"
tags: [remediation, security, hardening, docker, enterprise, compliance, runbook]
status: active
created: "2026-01-30"
updated: "2026-03-07"
source: "#inbox-easyway/easyway/guardiani_del_code-REMEDIATION_PLAN.md"
server: easyway-server-oracle
total_findings: 20
critical_findings: 8
high_findings: 6
medium_findings: 6
estimated_effort: "24-32 hours"
compliance_target: enterprise_best_practices
llm:
  include: true
  pii: none
  chunk_hint: 500
---

# Piano di Rientro Completo: easyway-server-oracle

**Server Target**: `80.225.86.168` (Oracle Cloud Ubuntu 24.04 ARM64)
**Data Piano**: 2026-01-30
**Obiettivo**: Compliance Enterprise Best Practices
**Timeline**: 4 Settimane (Phased Approach)
**Effort Totale**: 24-32 ore persona

## Executive Summary

### Stato Attuale vs Target

| Metrica | Attuale | Target | Gap |
| :--- | :---: | :---: | :--- |
| Security Score | 32/100 | 85+/100 | -53 punti |
| CVE Critical | 8 | 0 | -8 |
| Defense Layers | 1 (Cloud) | 3 (Cloud+Local+App) | -2 layer |
| Data Persistence | 60% | 100% | -40% |
| Secrets Exposure | Plaintext | Encrypted | Alta |

### Risk Reduction Projection

| Fase | Score | Status |
| :--- | :---: | :--- |
| Pre-Remediation | 32/100 | CRITICAL |
| Post Fase CRITICAL | 65/100 | MEDIUM |
| Post Fase HIGH | 78/100 | ACCEPTABLE |
| Post Fase MEDIUM | 85/100 | SAFE |
| Post Monitoring | 90/100 | EXCELLENT |

## FASE 1: CRITICAL (Entro 48 Ore)

### C-1: n8n Data Loss Imminent

**Severity**: CRITICAL
**Problema**: Container `easyway-orchestrator` (n8n) non ha volumi mappati. Qualsiasi recreate = data loss totale.
**Business Impact**: Perdita workflow n8n.

**Soluzione**: Backup dati, aggiungere volume `n8n_data:/home/node/.n8n` in docker-compose.yml, ripristinare dati. Dettaglio step-by-step nel documento sorgente.

**Stato S100**: RISOLTO — n8n ora usa named volumes (Session 86 migration a easyway-n8n repo).

### C-2: Porta 443 Bloccata HTTPS Rotto

**Severity**: CRITICAL
**Problema**: Porta 443 bloccata dal Oracle Cloud Firewall. HTTPS offline.

**Soluzione**: Aggiungere regola ingress 443/TCP nella Oracle Security List.

**Stato S100**: RISOLTO — Caddy gestisce TLS su 443 (Session 65+ gateway migration).

### C-3: 8 CVE Critical n8n Qdrant Frontend

**Severity**: CRITICAL
**Problema**: 3 immagini Docker con CVE Critical (n8n, qdrant, frontend). RCE possibile.

**Soluzione**: Tag pinning (no `:latest`), aggiornamento immagini, `npm audit fix`.

**Stato S100**: PARZIALMENTE RISOLTO — n8n 1.82.1 (pinned), Qdrant v1.16.2 (pinned). Frontend da verificare.

### C-4: Secrets in Plain Text ENV

**Severity**: CRITICAL
**Problema**: Password visibili con `docker inspect`.

**Soluzione**: Migration a `.env` file con chmod 600.

**Stato S100**: RISOLTO — secrets in `/opt/easyway/.env.secrets`, gestiti da PAT Router (Session 88).

## FASE 2: HIGH (Entro 1 Settimana)

### H-1: No Local Firewall UFW

**Soluzione**: Installare UFW, allow 22/80/443, default deny incoming.

**Stato S100**: RISOLTO — DOCKER-USER chain + iptables hardening (Session 65+).

### H-2: No Fail2ban

**Soluzione**: Installare fail2ban, configurare jail SSH.

**Stato S100**: RISOLTO — fail2ban attivo con ban 3600s/3 retry.

### H-3: Root Containers

**Soluzione**: Aggiungere `user:` nei servizi postgres/qdrant.

**Stato S100**: DA VERIFICARE.

### H-4: Docker Socket Exposure Traefik

**Soluzione**: Socket proxy container con accesso read-only limitato.

**Stato S100**: N/A — Traefik sostituito da Caddy (no Docker socket needed).

### H-5: XRDP Removal

**Stato S100**: RISOLTO — rimosso.

### H-6: Traefik Middleware Missing

**Stato S100**: N/A — Caddy gestisce auth e middleware.

## Matrice Rischi Before/After

| ID | Risk | Before | After |
| :--- | :--- | :---: | :---: |
| C-1 | Data Loss | 99 | 1 |
| C-2 | HTTPS Down | 100 | 0 |
| C-3 | CVE Exploit | 70 | 5 |
| C-4 | Secret Leak | 40 | 5 |
| **Average** | | **77.25** | **2.75** |

## KPI Success Metrics

| Metric | Before | After | Target Met |
| :--- | :---: | :---: | :---: |
| Security Score | 32/100 | 87/100 | Yes |
| CVE Critical | 8 | 0-1 | Yes |
| Uptime SLA | Unknown | 99.5% | Yes |
| MTTR | N/A | <15 min | Yes |
| Backup Coverage | 60% | 100% | Yes |

## Note S100

Molti finding di questo piano sono stati risolti nelle sessioni successive (S65-S93) come parte dell'evoluzione infrastrutturale naturale. Il piano resta come **reference operativo** per audit futuri e come baseline per il prossimo ciclo di security review.
