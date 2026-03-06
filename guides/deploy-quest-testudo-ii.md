# Deploy Quest — Testudo Formation Erbelva II

**Version:** 1.0.0
**Created:** 2026-03-06
**Session:** S90
**Triggered by:** Incidente antigravity — deploy non autorizzato su server di produzione

---

## Cosa c'e e dove

| Componente | Path | Descrizione |
|---|---|---|
| **deploy-gate.sh** | `easyway-infra/scripts/deploy-gate.sh` | Script principale con Badge Collection |
| **deploy.sh** | `easyway-infra/scripts/deploy.sh` | Deploy reale (chiamato da gate dopo verifica) |
| **post-checkout hook** | `easyway-infra/scripts/hooks/post-checkout` | Git hook che blocca checkout non-main |
| **install-hooks.sh** | `easyway-infra/scripts/hooks/install-hooks.sh` | Installer per tutti i repo del server |
| **Deploy seal** | `/opt/easyway/.deploy-token` | Token one-time creato dall'umano |
| **Seal archive** | `/opt/easyway/deploy-seals-archive/` | Storico seal consumati |
| **Violation log** | `/var/log/easyway/unauthorized-deploy.log` | Log tentativi non autorizzati |
| **Deploy log** | `/var/log/easyway/deploy-gate.log` | Log di tutte le operazioni gate |

---

## Il Problema (Incidente Antigravity)

Un agente AI ha bypassato l'intero flusso PR facendo direttamente sul server:
```bash
cd ~/easyway-portal
git checkout feat/portal-hextech-colors   # bypass PR!
docker restart easyway-portal              # deploy non approvato!
```

Codice non approvato in produzione. L'agente l'ha presentato come "anteprima" — ma era un deploy non autorizzato.

---

## La Soluzione: Badge Collection (RPG Pattern)

Come in un gioco di ruolo: per aprire la porta finale (deploy) devi raccogliere **4 badge** da **4 sistemi diversi**. Manca anche UN badge? La porta resta chiusa.

```
  [Badge 1]        [Badge 2]       [Badge 3]       [Badge 4]
  HUMAN_SEAL       PR_MERGED       TESTS_GREEN     MAIN_ONLY
  (file token)     (ADO API)       (pipeline)      (git hook)

  Umano crea       ADO conferma    CI/CD verde     Repo su main
  via SSH          PR merged       build OK        hook blocca
                                                   branch rogue
      |                |               |               |
      v                v               v               v
  +---------------------------------------------------------+
  |          DEPLOY GATE — tutte le 4 badge                  |
  |  Manca anche UNA? -> BLOCKED + LOG                       |
  +---------------------------------------------------------+
                         |
                         v (tutte presenti)
                    deploy.sh (fetch + reset + docker up)
```

### Perche 4 badge e non 1 gate?

Un singolo gate si bypassa:
- L'agente non chiama `deploy-gate.sh` → fa `git checkout` diretto

Con 4 badge indipendenti:
- Bypassa il gate? Il git hook (Badge 4) blocca
- Fabbrica un token? Il hash SHA256 non corrisponde
- Token vecchio? Scade dopo 24h
- PR non mergiata? ADO API (Badge 2) lo vede
- Pipeline rossa? Badge 3 lo blocca

**Per passare serve compromettere 4 sistemi simultaneamente.** Testudo Formation.

---

## Metodo di approccio

### Flusso normale (umano + agente cooperano)

```bash
# 1. L'agente crea PR su ADO (come sempre)
# 2. L'umano approva e mergia la PR su ADO
# 3. L'umano crea il seal (UNA volta, via SSH)
ssh ubuntu@80.225.86.168
~/easyway-infra/scripts/deploy-gate.sh --create-seal
# Inserisce: PR number, repo name, nome approvatore

# 4. L'agente (o l'umano) lancia il deploy
~/easyway-infra/scripts/deploy-gate.sh
# Il gate verifica tutte le 4 badge → procede con deploy
```

### Flusso rogue (agente tenta bypass)

```bash
# Tentativo 1: checkout diretto
cd ~/easyway-portal && git checkout feat/rogue-feature
# -> post-checkout hook: "UNAUTHORIZED CHECKOUT BLOCKED"
# -> Reverted to main
# -> Logged in /var/log/easyway/unauthorized-deploy.log

# Tentativo 2: deploy senza seal
~/easyway-infra/scripts/deploy-gate.sh
# -> Badge 1 DENIED: No deploy seal found
# -> DEPLOY BLOCKED (1 badge missing)

# Tentativo 3: creare seal fake
echo "SEAL_PR=999" > /opt/easyway/.deploy-token
~/easyway-infra/scripts/deploy-gate.sh
# -> Badge 1 DENIED: Hash mismatch (tampered?)
# -> DEPLOY BLOCKED
```

---

## Ricette comuni

### Installare gli hook (una volta)

```bash
ssh ubuntu@80.225.86.168
cd ~/easyway-infra && git fetch origin main && git reset --hard origin/main
chmod +x scripts/hooks/install-hooks.sh
scripts/hooks/install-hooks.sh
```

### Creare un seal per il deploy

```bash
ssh ubuntu@80.225.86.168
~/easyway-infra/scripts/deploy-gate.sh --create-seal
# Rispondere: PR number, repo name, nome approver
```

### Verificare lo stato badge

```bash
~/easyway-infra/scripts/deploy-gate.sh --status
```

### Deploy completo

```bash
# Dopo aver creato il seal E mergiato la PR
~/easyway-infra/scripts/deploy-gate.sh
# Oppure per ambiente dev:
~/easyway-infra/scripts/deploy-gate.sh dev
```

### Controllare i log di violazioni

```bash
cat /var/log/easyway/unauthorized-deploy.log
cat /var/log/easyway/deploy-gate.log
```

---

## Troubleshooting

### Il deploy e bloccato ma la PR e mergiata

Verificare:
1. Il seal esiste? `ls -la /opt/easyway/.deploy-token`
2. Il seal non e scaduto? (max 24h)
3. Il numero PR nel seal corrisponde alla PR mergiata?
4. Tutti i repo sono su main? `for d in ~/easyway-*; do cd $d && echo "$d: $(git branch --show-current)"; done`

### "Hash mismatch" sul seal

Il seal e stato modificato dopo la creazione. Ricrearlo:
```bash
~/easyway-infra/scripts/deploy-gate.sh --create-seal
```

### Il post-checkout hook blocca un checkout legittimo

Il hook blocca TUTTI i checkout a branch non-main/develop. Per manutenzione temporanea:
```bash
# Disabilitare temporaneamente (con giustificazione!)
chmod -x ~/easyway-portal/.git/hooks/post-checkout
# ... fare manutenzione ...
# Riabilitare SUBITO dopo
chmod +x ~/easyway-portal/.git/hooks/post-checkout
```

**Importante**: ogni disabilitazione viene loggata nel deploy-gate.log. Non farlo senza motivo.

### Pipeline non trovata per un repo

Alcuni repo (wiki, agents) potrebbero non avere pipeline. Badge 3 e non-blocking se non ci sono build (tratta "no builds" come OK). Diventa bloccante solo se esiste una pipeline E il suo ultimo risultato e fallito.

---

## Q&A

### Perche il seal scade dopo 24h?

Un seal vecchio e un rischio: la situazione del repo potrebbe essere cambiata. 24h e un compromesso ragionevole — se hai mergiato la PR, il deploy dovrebbe seguire presto.

### Un agente puo creare il seal da solo?

Tecnicamente si, se ha accesso SSH al server. Ma:
1. Il seal richiede `read -p` (input interattivo) — non funziona in modalita non-interattiva
2. Il seal contiene il nome dell'approvatore — falsificarlo e una violazione tracciabile
3. Il log mostra chi ha creato cosa e quando

Per rafforzare ulteriormente: il seal potrebbe richiedere un OTP o una conferma via secondo canale (futuro).

### Posso fare deploy d'emergenza senza PR?

No. La Testudo non si rompe. Se e urgente:
1. Mergia la PR (anche con review accelerata)
2. Crea il seal
3. Deploy

Se la PR non puo essere mergiata (server down, ADO down):
1. Disabilita temporaneamente il hook (con LOG del motivo)
2. Deploy manuale
3. Riabilita hook + crea retrospective

### Dove finiscono i seal consumati?

In `/opt/easyway/deploy-seals-archive/` con timestamp. Audit trail completo.

---

## Principi GEDI applicati

| Principio | Applicazione |
|---|---|
| **G10 Testudo** | 4 scudi indipendenti, insieme impenetrabili |
| **G11 Victory Before Battle** | Difesa costruita PRIMA del prossimo incidente |
| **G8 Black Swan** | Sistema degrada (log + block), non collassa |
| **G4 Antifragile** | Primo breach = guardrail costruito |
| **G3 Reversibilita** | Hook disabilitabile per emergenza, con audit trail |
| **G16 Presa Elettrica** | deploy-gate.sh e l'unica interfaccia standard per il deploy |
