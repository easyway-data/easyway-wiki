---
id: ew-07-iam-naming-utenti-gruppi
title: 07 iam naming utenti gruppi
summary: Naming Convention ufficiale per Utenze Tecniche (portal.*) e Gruppi IAM (grp.*).
status: active
owner: team-docs
created: '2025-01-01'
updated: '2026-01-16'
tags: [domain/control-plane, layer/reference, audience/dev, audience/ops, privacy/internal, language/it, domain/iam, domain/rbac, process/naming]
llm:
  include: true
  pii: none
  chunk_hint: 250-400
  redaction: [email, phone]
entities: []
next: Applicare a terraform.
type: guide
---
[Home](./start-here.md) >  > 

# 📂 Wiki IAM - Naming Convention Utenze Tecniche e Gruppi  
EasyWay DataPortal - Storage & Datalake  

## 🎯 Scopo del Documento  
Definire le regole ufficiali di **naming convention** per:  
- Utenze di servizio tecniche  
- Gruppi di accesso IAM (RBAC)  

Questo garantisce ordine, chiarezza e coerenza su tutta la piattaforma EasyWay DataPortal.

---

## 🔑 Naming Convention - Utenze Tecniche di Servizio  

### 📋 Prefisso da usare: `portal.`  
Le utenze tecniche devono iniziare sempre con `portal.`.  
Queste utenze sono dedicate a:  
- Portale EasyWay  
- Power BI  
- ETL, App esterne  
- Accesso Storage Explorer (admin)  

### 📂 Esempi standard:
| Nome utenza tecnica           | Scopo                          | Ambito      |
|--------------------------------|--------------------------------|-------------|
| `portal.datalake.dev.read`     | Lettura dati Datalake           | App, Power BI |
| `portal.datalake.dev.write`    | Scrittura su Datalake           | ETL, App    |
| `portal.datalake.dev.admin`    | Admin su Datalake (Explorer)    | Amministrazione |
| `portal.storage.dev.read`      | Lettura su Blob Storage (assets)| Assets, immagini |
| `portal.storage.dev.write`     | Scrittura su Blob Storage       | Upload App  |
| `portal.storage.dev.admin`     | Admin su Blob Storage           | IAM, ACL    |

---

## 🔑 Naming Convention - Gruppi IAM (RBAC)  

### 📋 Prefisso da usare: `grp.`  
I gruppi IAM devono iniziare sempre con `grp.`.  
Questi gruppi controllano l'accesso tramite RBAC su Azure.

### 📂 Esempi standard:
| Nome gruppo IAM                | Permesso        | Ambito     |
|--------------------------------|-----------------|------------|
| `grp.portal.datalake.read`     | Lettura         | Datalake   |
| `grp.portal.datalake.write`    | Scrittura       | Datalake   |
| `grp.admin.datalake`           | Amministrazione | Datalake   |
| `grp.portal.storage.read`      | Lettura         | Blob Storage |
| `grp.admin.storage`            | Amministrazione | Blob Storage |

---

## 📊 Differenza tra Utenze e Gruppi
| Tipologia      | Prefisso | Scopo                                 |
|----------------|----------|----------------------------------------|
| **Utenza tecnica** | `portal.` | Account di servizio per App, Power BI, ETL |
| **Gruppo IAM**     | `grp.`    | Gestione permessi tramite RBAC IAM      |

---

## 🛑 Regola fondamentale
| Tipo        | Utenti diretti? | Consiglio |
|-------------|-----------------|-----------|
| Storage IAM | **No**           | Solo tramite `grp.` gruppi |
| Accesso tecnico | **Sì** (utenza tecnica `portal.`) | Per App / Power BI / ETL |

---

## 🚩 Esempio di organizzazione pulita

```sql
Users:
└── portal.datalake.dev.admin@tenant.onmicrosoft.com
└── portal.storage.dev.read@tenant.onmicrosoft.com

Groups:
└── grp.admin.datalake
└── grp.portal.datalake.read
```sql

---

## 📝 Perché questa convenzione?
| Motivazione   | Beneficio       |
|---------------|-----------------|
| Chiarezza     | Capisci subito cosa fa ogni utenza o gruppo |
| Scalabilità   | Aggiungere nuovi ruoli è semplice e ordinato |
| Audit         | IAM più pulito e leggibile nei log           |

---

## 📂 Best Practice - Assegnazione Permessi e IAM (IAM Assignment)
Quando assegni i permessi tramite IAM su Azure Storage o Datalake, usa sempre una **descrizione chiara** e coerente.

### 📄 Esempio di descrizione:
```sql
IAM Assignment:
Gruppo: grp.admin.datalake
Ruolo: Storage Account Contributor
Scopo: Accesso amministrazione Datalake (IAM + ACL)
```sql

### 📋 Esempi pratici di descrizione:
| Gruppo IAM             | Ruolo Assegnato               | Descrizione consigliata                                       |
|-------------------------|--------------------------------|----------------------------------------------------------------|
| `grp.admin.datalake`    | `Storage Account Contributor`   | Accesso amministrativo completo al Datalake tramite IAM + ACL  |
| `grp.portal.datalake.read` | `Storage Blob Data Reader`    | Accesso in sola lettura ai dati certificati Datalake per report |
| `grp.portal.datalake.write`| `Storage Blob Data Contributor` | Scrittura dati in staging/landing per ETL e App                |

---





