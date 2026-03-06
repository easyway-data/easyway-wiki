---
id: scelta-dominio-dns
title: Scelta Dominio e Registrar (Privacy First)
summary: Decision log sulla scelta del dominio e registrar con approccio privacy-first.
status: stable
owner: team-infra
tags: [domain/infrastructure, privacy, domain/dns, domain/compliance, decision-log]
llm:
  include: true
  pii: none
  chunk_hint: 300-500
  redaction: [email, phone]
created: '2026-02-01'
updated: '2026-02-02'
entities: []
type: guide
---

[[start-here.md|Home]] > [[infrastructure/index.md|Infrastructure]] > Scelta Dominio

# Scelta Dominio e Registrar (Privacy/Ethics First)

## Obiettivo
Definire una decisione coerente e ripetibile per acquistare e gestire un dominio, con priorità assoluta alla **privacy** e all’**etica** (livello 10/10), considerando anche lo scenario **server on-premise**.

---

## Contesto iniziale: “Voglio prendermi un dominio”
All’inizio l’obiettivo era capire **cosa fare** per acquistare un dominio e cosa fosse consigliabile in generale.

### Step generali (acquisto dominio)
1. Scegliere il nome (breve, memorizzabile, senza trattini)
2. Scegliere l’estensione (.com / .it / ecc.)
3. Registrare il dominio presso un registrar
4. Configurare DNS
5. Se serve: email, web, API, ecc.

---

## Momento chiave #1: “Namecheap vs Aruba?”
Qui la scelta si sposta su due opzioni tipiche:
- Registrar internazionale “developer friendly”
- Registrar italiano “ecosistema completo”

### Criteri emersi
- Facilità gestione DNS
- Trasparenza prezzi (soprattutto rinnovi)
- Supporto
- Servizi extra (hosting/email) *non necessari se l’infrastruttura è altrove*

---

## Momento chiave #2: “Ma se ho un server on-premise?”
Questo cambia l’architettura, NON la necessità del dominio.

### Architettura di base (on-prem)
Utente → Dominio → DNS → IP pubblico → Firewall/Router → Reverse Proxy → Server on-prem

### Requisiti tecnici minimi per on-prem “serio”
- IP pubblico statico (preferibile)
- Apertura/gestione porte 80/443 (o reverse proxy dedicato)
- HTTPS con certificato (Let’s Encrypt o equivalente)
- Hardening: firewall, rate limit, log, patching

### Nota
Il registrar **non ospita** il tuo servizio. Il registrar (e il DNS) determinano *solo* come il nome punta all’infrastruttura.

---

## Momento chiave #3: “Stai facendo pubblicità? Sono tutte in Florida”
Qui nasce il criterio dominante: **non voglio US-centric se posso evitarlo**.

### Impatto reale
- Registrar e DNS vedono **metadata** (dominio, contatti, query DNS)
- Non vedono direttamente i dati applicativi, ma:
  - il **DNS** può vedere richieste e pattern (metadata)
  - la giurisdizione può imporre regole diverse su accesso ai dati / ordini legali

### Conclusione del momento
Se la priorità è privacy/compliance “10/10”, la scelta più coerente è **EU-centric**, minimizzando esposizione e dipendenze da enti/leggi extra-UE.

---

## Momento chiave #4: “Mi dai i costi di questi?”
I costi variano molto per:
- promo primo anno vs rinnovo
- estensione (.com, .it, .io…)
- servizi aggiuntivi (privacy, email, ecc.)

### Regola decisionale (cost)
**Non si decide sul prezzo del primo anno.**
Si decide su:
- prezzo di rinnovo
- presenza di “upsell” sulla privacy
- trasparenza del listino
- governance dei dati

---

## Momento chiave #5: “Mi piace il discorso privacy: gli altri non lo garantiscono?”
Qui abbiamo chiarito un concetto fondamentale:

### GDPR vs WHOIS privacy
- **GDPR** (UE) tende a limitare la visibilità pubblica di dati personali nel WHOIS/RDAP
- **WHOIS Privacy / Proxy** è un livello ulteriore:
  - sostituisce i dati del registrante con dati proxy
  - riduce spam, scraping, esposizione diretta

### Interpretazione “ethics-first”
Il punto non è “chi è legale”.
Il punto è “chi rende la privacy **default** e non **upsell**”.

---

## Momento chiave #6: “Chi rispetta di default da un punto di vista etico?”
Qui nasce la preferenza netta: vogliamo **privacy by default** senza upsell.

### Definizione operativa di “etico” in questo contesto
- Privacy non venduta come extra
- Minimizzazione dati esposti
- Trasparenza su costi e rinnovi
- Gestione chiara e non aggressiva
- Contesto giuridico coerente con GDPR (se possibile)

---

## Momento chiave #7: “Sono controllate da altre società?”
Domanda cruciale: proprietà e controllo possono cambiare policy nel tempo.

### Perché conta
- Un’azienda acquisita potrebbe cambiare strategia
- Un gruppo grande potrebbe spingere upsell, lock-in, monetizzazione
- Quindi serve valutare **comportamento + governance**

---

# Decisione finale (Privacy/Ethics 10/10)

## Scelta preferita: Gandi (EU-centric, privacy-first)
**Motivo etico principale**
- Approccio storico “privacy oriented”
- In generale, maggiore coerenza con principi GDPR e minimizzazione dati
- Minor dipendenza da giurisdizioni extra-UE

**Motivo tecnico**
- Buona gestione DNS
- Standard puliti (DNSSEC, gestione record)
- Buon fit con infrastruttura on-prem o ibrida

**Trade-off accettato**
- Non sempre il più economico
- La proprietà societaria può evolvere nel tempo (si gestisce con controlli periodici)

### Regola di coerenza
Se l’obiettivo è **privacy 10/10**, si accetta un costo leggermente maggiore e si privilegia:
- EU jurisdiction
- privacy “default”
- trasparenza

---

## Alternativa pragmatica: Namecheap (privacy forte, ma USA)
**Pro**
- Strumenti e pannello DNS molto semplici
- Whois privacy spesso molto efficace
- Buon rapporto qualità/prezzo

**Contro (decisivo per privacy 10/10)**
- Giurisdizione USA
- Anche se privacy tecnica è buona, la scelta non è coerente con “EU-only ethics”
- Quindi **si scarta** per coerenza di principio (non per qualità tecnica)

---

## Alternativa “ecosistema italiano”: Aruba (Italia, ma privacy spesso come extra)
**Pro**
- EU/Italia, contesto GDPR
- Ecosistema “tutto in uno”
- Supporto e servizi locali

**Contro (decisivo per privacy 10/10)**
- Privacy come servizio aggiuntivo (impostazione “commerciale”)
- Meno “privacy-by-default” come filosofia di base
- Quindi **si scarta** come scelta primaria per privacy 10/10
  (resta valida se si vuole un ecosistema unico e si accetta l’upsell)

---

# Coerenza on-prem: come si applica la scelta

## Obiettivo tecnico su on-prem
- Dominio → DNS → IP pubblico → Firewall → Reverse proxy → Server
- HTTPS obbligatorio
- Logging e hardening

## Regola privacy
Se la priorità è 10/10:
- preferire registrar EU-centric
- preferire DNS EU-centric (o self-managed)
- minimizzare servizi “edge” extra-UE che vedono traffico o metadata DNS

---

# Checklist decisionale (da riusare ogni volta)

## 1) Registrar
- [ ] EU-centric (se privacy 10/10)
- [ ] Privacy by default (non upsell)
- [ ] Trasparenza rinnovi
- [ ] DNSSEC supportato

## 2) DNS
- [ ] Gestione in EU o self-managed
- [ ] DNSSEC attivo
- [ ] Minimizzazione log/retention (se configurabile)

## 3) On-prem readiness
- [ ] IP statico o soluzione robusta se dinamico
- [ ] Firewall configurato (solo porte necessarie)
- [ ] Reverse proxy e HTTPS
- [ ] Patch management e monitoring

---

# Verdetto finale
Per privacy ed etica **10/10**, la scelta coerente è:

**Registrar EU-centric + privacy-by-default (scelta primaria: Gandi)**  
e un DNS coerente con la stessa filosofia (EU o self-managed).

Scartiamo le altre opzioni non perché “sbagliate”, ma perché:
- USA-centric (Namecheap) = incoerenza di principio
- privacy come upsell (Aruba) = incoerenza di impostazione

---

# Nota di manutenzione (importante)
Ogni 6-12 mesi:
- verificare eventuali cambi policy del registrar
- verificare costi di rinnovo aggiornati
- verificare cambiamenti su privacy WHOIS/RDAP dei registry (.com/.it/.eu)

Questa pagina è il riferimento “coerenza decisionale” e va aggiornata se cambiano fatti oggettivi.

*(Aprire solo quando il mail server è definito)*

---

---

## Implementazione concreta: DNS + Reverse Proxy on-prem (EU-only, Privacy 10/10)

Questa sezione traduce la scelta etica in **configurazione tecnica reale**, evitando incoerenze tra principi e implementazione.

---

## Obiettivo architetturale

Garantire che:
- il **dominio** sia gestito in modo privacy-first
- i **DNS** non esportino metadata fuori UE
- il **traffico applicativo** non passi da proxy esterni non necessari
- l’infrastruttura on-prem sia esposta **in modo minimale e sicuro**

---

## Architettura di riferimento (EU-only)

### DNSSEC (OBBLIGATORIO per privacy 10/10)
- Abilitare DNSSEC sul dominio
- Verificare:
  - DS record pubblicato
  - stato “signed” e “secure”

DNSSEC protegge da:
- spoofing DNS
- man-in-the-middle sulla risoluzione nomi

---

## DNS – regole di coerenza etica

✔ DNS gestiti:
- in UE **oppure**
- self-hosted (Bind / PowerDNS)

❌ Da evitare:
- DNS con logging aggressivo non disattivabile
- DNS che monetizzano query
- DNS usati come “gateway applicativo” (proxy/CDN se non necessari)

---

## Reverse Proxy on-prem (punto critico)

Il reverse proxy è **l’unico punto esposto** verso Internet.

### Scelte coerenti
- Nginx
- Traefik
- Caddy

(Tutti validi, l’importante è la configurazione)

---

### HTTPS – obbligatorio

- Certificati: **Let’s Encrypt**
- Porta 80:
  - solo per challenge ACME
  - redirect immediato a 443
- Porta 443:
  - unica porta pubblica permanente

---

### Hardening minimo reverse proxy

#### TLS
- TLS 1.2 / 1.3
- disabilitare cipher deboli
- HSTS attivo

#### Header di sicurezza
- `Strict-Transport-Security`
- `X-Content-Type-Options`
- `X-Frame-Options`
- `Referrer-Policy`
- `Content-Security-Policy` (se applicabile)

---

### Rate limiting
Protezione base contro scansioni e brute force.

- limitare richieste per IP
- soglie diverse per:
  - API
  - login
  - static assets

---

### Logging (privacy-aware)
- Log **minimali**
- Nessun header superfluo
- Retention breve
- Mascherare IP se non necessari per auditing

---

## Firewall perimetrale

### Regole minime
- Ingress:
  - TCP 443 → reverse proxy
  - TCP 80 → solo ACME
- Egress:
  - solo ciò che serve (update, CRL, ACME)

### Regola d’oro
> **Nessun servizio applicativo esposto direttamente.**

---

## Checklist finale di coerenza (Privacy 10/10)

### Dominio / Registrar
- [ ] EU-centric
- [ ] Privacy WHOIS by default
- [ ] Nessun upsell sulla privacy

### DNS
- [ ] EU o self-managed
- [ ] DNSSEC attivo
- [ ] Record CAA configurati
- [ ] Nessun proxy CDN non necessario

### On-prem
- [ ] IP statico
- [ ] Firewall chiuso by default
- [ ] Reverse proxy unico punto esposto
- [ ] HTTPS obbligatorio
- [ ] Logging minimale

---

## Regola finale (da ricordare sempre)

> **La privacy non è un plugin.  
> È una catena di scelte coerenti.  
> La catena è forte quanto l’anello più debole.**

Questa sezione completa la decisione etica trasformandola in **implementazione tecnica verificabile**.


Sezione 1 — Threat Model minimale (Privacy / Metadata / Coerenza)
---

## Threat Model minimale (focus Privacy & Metadata)

Questa sezione definisce **cosa stiamo proteggendo**, **da chi**, e **perché le scelte fatte sono coerenti** con un livello di privacy 10/10.

---

## Asset da proteggere

### Asset primari
- Identità del proprietario del dominio
- Metadata DNS (query, pattern di accesso)
- Architettura interna on-prem
- Superficie di esposizione pubblica

### Asset secondari
- Reputazione del progetto
- Continuità operativa
- Coerenza compliance (GDPR, data minimization)

---

## Attori potenzialmente ostili (non paranoici, realistici)

| Attore | Rischio |
|------|--------|
| Scraper WHOIS | Raccolta dati personali |
| Broker di dati | Correlazione domini ↔ persone |
| Attaccanti opportunistici | Scan IP / brute force |
| Fornitori terzi | Logging eccessivo / uso metadata |
| Cambi policy provider | Erosione privacy nel tempo |

---

## Vettori di rischio principali

### 1. WHOIS / RDAP
- Espone dati del registrante se non anonimizzati
- Mitigazione:
  - Registrar con privacy **by default**
  - Dati proxy al posto di dati reali

### 2. DNS
- Le query DNS rivelano:
  - esistenza del servizio
  - frequenza e pattern di accesso
- Mitigazione:
  - DNS EU-only o self-managed
  - DNSSEC
  - Nessun DNS “analytics-based”

### 3. Edge / CDN / Proxy esterni
- Possono vedere:
  - IP client
  - URL
  - header
- Mitigazione:
  - Nessun proxy esterno non necessario
  - Reverse proxy on-prem come unico punto di ingresso

### 4. Superficie di esposizione on-prem
- Porte aperte = superficie di attacco
- Mitigazione:
  - firewall default-deny
  - reverse proxy unico
  - rate limiting
  - TLS obbligatorio

---

## Decisioni chiave e minacce mitigate

| Decisione | Minaccia mitigata |
|--------|------------------|
| Registrar EU + privacy default | Data scraping, spam |
| DNS EU/self-managed | Metadata leakage |
| DNSSEC | Spoofing DNS |
| Reverse proxy unico | Attacchi diretti ai servizi |
| No CDN esterno | Profilazione traffico |
| Logging minimale | Over-collection dati |

---

## Regola guida del threat model

> **Non preveniamo tutto.  
> Riduciamo ciò che non è necessario esporre.**

Questo threat model è intenzionalmente minimale per rimanere **manutenibile nel tempo**.

🧾 Sezione 2 — Audit leggero periodico (governance & coerenza)
---

## Audit leggero periodico (coerenza etica e tecnica)

Questa sezione serve a evitare il rischio più comune:
> *fare la scelta giusta oggi e dimenticarsene domani.*

L’audit è **leggero**, ripetibile e non burocratico.

---

## Frequenza consigliata
- **Ogni 6 o 12 mesi**
- Oppure:
  - cambio provider
  - cambio policy
  - espansione del progetto

---

## Checklist audit – Registrar

- [ ] Privacy WHOIS ancora attiva by default
- [ ] Nessun cambio policy su esposizione dati
- [ ] Nessun nuovo upsell “obbligatorio”
- [ ] Prezzo rinnovo coerente e trasparente
- [ ] Giurisdizione invariata

Se **una risposta è NO** → rivalutare il registrar.

---

## Checklist audit – DNS

- [ ] DNSSEC ancora attivo
- [ ] Record corretti (A / AAAA / CAA)
- [ ] Nessun servizio “analytics DNS” abilitato
- [ ] Provider DNS invariato
- [ ] Retention log compatibile con minimizzazione

---

## Checklist audit – On-prem exposure

- [ ] Solo 80/443 esposte
- [ ] Certificati validi
- [ ] TLS aggiornato
- [ ] Rate limiting attivo
- [ ] Nessun servizio bypassa il reverse proxy

---

## Checklist audit – Coerenza etica

Domande da porsi:
- Stiamo usando questo servizio perché **serve** o perché è “comodo”?
- Questo componente vede dati che non dovrebbe?
- C’è un’alternativa EU-only o self-hosted oggi più matura?

Se la risposta è sì → valutare migrazione.

---

## Decision Log (obbligatorio)

Ogni audit deve produrre **una riga di log**:

| Data | Ambito | Esito | Azione |
|---|---|---|---|
| 2026-06-30 | DNS | OK | Nessuna |
| 2026-06-30 | Registrar | Warning | Verifica policy rinnovo |

Questo evita:
- decisioni emotive
- incoerenze future
- perdita di memoria storica

---

## Regola finale di governance

> **L’etica non è una scelta una tantum.  
> È una scelta che va difesa nel tempo.**

🧠 Dove siamo arrivati (meta-riepilogo)

Ora il documento contiene:

✅ Ragionamento decisionale completo

✅ Scelta etica motivata

✅ Implementazione tecnica coerente

✅ Threat model

✅ Audit e governance

Questo non è più “un appunto”:
è una policy tecnica difendibile, anche tra 5 anni.

Prossimi step possibili (scegline uno)

📧 Email EU-only coerente col dominio

🧱 Separazione ambienti (dev / prod / lab)

🗂 Gestione segreti e certificati (on-prem)

📜 Allineamento esplicito a GDPR / ISO27001 / SOC2

🔐 Zero Trust minimale per accesso admin

