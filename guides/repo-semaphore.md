---
title: "Repo Semaphore - Semaforo di Stato Repository"
tags: [governance, workflow, agenti, semaforo, concorrenza]
status: active
created: 2026-03-07
---

# Repo Semaphore

Ogni repository ha un file `.semaphore.json` nella root che indica lo stato corrente del repo. Il file e gitignored -- vive solo in locale.

---

## Stati

| Colore | Significato | Azione per chi arriva |
|--------|-------------|----------------------|
| **green** | Libero, nessuno ci sta lavorando | Puoi procedere, metti giallo |
| **yellow** | Lavoro in corso | Coordina con la sessione attiva prima di toccare |
| **red** | Deploy in corso | NON toccare. Aspetta il verde |

Se il file `.semaphore.json` non esiste, il repo e considerato **verde** (libero).

---

## Formato

```json
{
  "status": "yellow",
  "branch": "feat/nome-feature",
  "session": "S98",
  "agent": "claude-code",
  "since": "2026-03-07T10:00:00",
  "note": "Descrizione breve di cosa si sta facendo"
}
```

| Campo | Obbligatorio | Descrizione |
|-------|:---:|-------------|
| `status` | si | `green`, `yellow`, `red` |
| `branch` | si | Branch su cui si sta lavorando |
| `session` | si | ID sessione (es. S98) |
| `agent` | si | Chi sta lavorando (claude-code, cursor, umano) |
| `since` | si | Timestamp ISO di inizio |
| `note` | no | Cosa si sta facendo |

---

## Regole operative

### Inizio sessione
1. Controlla `.semaphore.json` di ogni repo che intendi toccare
2. Se **verde** (o assente): metti **giallo** con i tuoi dati
3. Se **giallo**: leggi chi sta lavorando, coordina prima di procedere
4. Se **rosso**: NON toccare, aspetta

### Fine sessione / Fine lavoro su un repo
1. Metti **verde** (o cancella il file)
2. Fallo PRIMA del closeout -- il semaforo si rilascia appena finisci, non dopo

### Deploy
1. Metti **rosso** PRIMA di iniziare il deploy
2. Torna **verde** solo DOPO aver verificato che il deploy e andato a buon fine

---

## Setup

Aggiungere `.semaphore.json` al `.gitignore` di ogni repo:

```
# Repo semaphore (agent lock)
.semaphore.json
```

---

## Cosa NON e

- NON e un lock bloccante -- il semaforo e informativo, non impedisce il lavoro
- NON sostituisce git (branch, merge, PR restano il meccanismo vero di concorrenza)
- NON va committato -- e stato locale, non condiviso via git

Il semaforo serve a **comunicare** tra sessioni sullo stesso terminale, dove piu agenti possono lavorare in parallelo sugli stessi repo.
