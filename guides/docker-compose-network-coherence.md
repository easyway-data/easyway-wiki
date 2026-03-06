# Docker Compose Multi-File Network Coherence

> **GEDI Case #25 — La Rete Spezzata (The Split Network)**
> Session 65 — 2026-03-04

## TL;DR

Quando si usano **compose overlay multipli** (`-f base.yml -f apps.yml -f prod.yml`) con un **project name** (`-p easyway-prod`), le reti Docker possono finire **spezzate**: alcuni servizi su una rete, altri su un'altra. I container non si vedono tra loro.

**Regola d'oro**: usare sempre `name: <network-name>` nella definizione rete del compose base. Mai `external: true` negli overlay se la rete è definita nel base.

---

## Cosa È Successo

### Il Contesto

EasyWay usa 3 compose file sovrapposti per il deploy:

```
docker-compose.yml        (base: runner, qdrant, azurite, minio, frontend, n8n)
docker-compose.apps.yml   (overlay: api, restart policies, env tuning)
docker-compose.prod.yml   (overlay: caddy, postgres, traefik backup, healthchecks)
```

Deploy via `deploy.sh`:

```bash
docker compose \
    --project-directory "$PORTAL_DIR" \
    -f docker-compose.yml \
    -f docker-compose.apps.yml \
    -f docker-compose.prod.yml \
    -p "easyway-prod" \
    up -d --build
```

### L'Anti-Pattern

```yaml
# docker-compose.yml (BASE)
networks:
  easyway-net:
    driver: bridge       # ← Compose crea "easyway-prod_easyway-net" (prefisso progetto)

# docker-compose.apps.yml (OVERLAY)
networks:
  easyway-net:
    external: true       # ← Cerca "easyway-net" pre-esistente (SENZA prefisso)

# docker-compose.prod.yml (OVERLAY)
networks:
  easyway-net:
    external: true       # ← Idem: cerca "easyway-net" senza prefisso
```

### Il Risultato: La Rete Spezzata

| Rete | Servizi | Subnet |
|------|---------|--------|
| `easyway-prod_easyway-net` | portal, runner, orchestrator, cortex, storage, storage-s3 | 172.27.0.0/16 |
| `easyway-net` | caddy, meta-db, api | 172.18.0.0/16 |

**Caddy** (reverse proxy) era su `easyway-net` ma i servizi che doveva raggiungere (frontend, n8n, qdrant) erano su `easyway-prod_easyway-net`. DNS resolution falliva:

```
$ docker exec easyway-gateway-caddy nslookup frontend
** server can't find frontend: NXDOMAIN
```

Risultato: **HTTP 500** su porta 80.

### Perché È Insidioso

- Caddy risultava `healthy` (healthcheck su admin API :2019, non sulle route)
- I servizi singolarmente funzionavano (porta diretta :8080, :3000, :5678 OK)
- Solo il reverse proxy falliva — l'errore si manifesta nel punto di integrazione, non nei singoli componenti
- `docker ps` mostrava tutto "Up" — nessun segnale visibile di split

---

## La Root Cause Tecnica

### Come Docker Compose gestisce le reti con `-p` (project name)

1. **Rete definita normalmente** (`driver: bridge`): Compose la crea con prefisso progetto
   - Input: `easyway-net` + `-p easyway-prod`
   - Output: `easyway-prod_easyway-net`

2. **Rete con `external: true`**: Compose NON la crea, la cerca per nome esatto
   - Input: `easyway-net` + `external: true`
   - Output: cerca `easyway-net` (senza prefisso)

3. **Rete con `name: <nome>`**: Compose la crea con nome esatto, ignorando il prefisso
   - Input: `name: easyway-net` + `-p easyway-prod`
   - Output: `easyway-net` (nome esatto)

### Il Merge delle Definizioni

Quando compose mergia 3 file, la definizione rete viene unita. Ma `external: true` in un overlay **NON** sovrascrive `driver: bridge` nel base — i servizi del base usano la rete creata dal base (con prefisso), i servizi dell'overlay cercano la rete external (senza prefisso).

---

## La Soluzione

### Il Pattern Corretto

```yaml
# docker-compose.yml (BASE)
networks:
  easyway-net:
    name: easyway-net      # ← Nome esatto, nessun prefisso progetto
    driver: bridge

# docker-compose.apps.yml (OVERLAY)
networks:
  easyway-net:
    name: easyway-net      # ← Stesso nome, compose riconosce è la stessa rete

# docker-compose.prod.yml (OVERLAY)
networks:
  easyway-net:
    name: easyway-net      # ← Stesso nome, tutti sulla stessa rete
```

### Perché Funziona

- `name:` forza il nome esatto della rete Docker, indipendentemente dal project name
- Tutti i compose file convergono sulla stessa rete
- Non serve `external: true` perché compose crea la rete se non esiste
- Idempotente: funziona sia al primo deploy che ai successivi

---

## Framework: Compose Coherence Gate

### Principio

> **Se deploy.sh è il soldato Testudo, la validazione rete è lo scudo mancante.**

Il Testudo check esistente valida:
- [x] File compose esistono
- [x] Directory repo esistono
- [x] Caddyfile esiste
- [x] Docker disponibile
- [ ] ~~Coerenza rete tra compose files~~ ← **MANCAVA**

### Gate Proposto: Network Coherence Check

Aggiungere a deploy.sh, dopo il Testudo check:

```bash
# ═══════════════════════════════════════════════════════════════
# COMPOSE COHERENCE GATE — Validate before deploy
# ═══════════════════════════════════════════════════════════════
echo -e "${YELLOW}[deploy] Coherence check — validating compose config...${NC}"

# 1. Dry-run: docker compose config (merge + validate)
COMPOSE_CONFIG=$(docker compose \
    --project-directory "$PORTAL_DIR" \
    $COMPOSE_FILES \
    -p "easyway-${ENV}" \
    config 2>&1) || {
    echo -e "${RED}[FAIL] Compose config validation failed:${NC}"
    echo "$COMPOSE_CONFIG"
    exit 1
}

# 2. Check: tutte le reti hanno un name esplicito (no project prefix)
UNNAMED_NETS=$(echo "$COMPOSE_CONFIG" | grep -A1 "^networks:" | grep -v "name:" | grep -v "^networks:" | grep -v "^--$")
if [ -n "$UNNAMED_NETS" ]; then
    echo -e "${RED}[FAIL] Network without explicit 'name:' detected — risk of split${NC}"
    echo "$UNNAMED_NETS"
    exit 1
fi

# 3. Check: nessuna rete dichiarata 'external' (anti-pattern con overlay)
if echo "$COMPOSE_CONFIG" | grep -q "external: true"; then
    echo -e "${RED}[FAIL] 'external: true' network detected — use 'name:' instead${NC}"
    exit 1
fi

echo -e "${GREEN}[deploy] Coherence OK — compose config validated.${NC}"
```

### Checklist Manuale (per code review)

Quando si modifica un compose file, verificare:

1. **Ogni rete ha `name:`?** — Se no, rischio di prefisso progetto
2. **Nessun `external: true`?** — Se sì, rischio di split con overlay
3. **Stesso `name:` in tutti i file?** — Se no, reti diverse
4. **`docker compose config` passa?** — Validazione merge senza ambiguità
5. **Servizi sullo stesso network?** — Tutti i servizi che devono comunicare

---

## Q&A

### Q: Perché non usare `external: true` negli overlay?

**A**: Perché `external: true` cerca una rete pre-esistente per nome esatto (senza prefisso progetto). Se il base compose crea la rete con prefisso (comportamento default), overlay e base finiscono su reti diverse. `name:` è idempotente: crea se non esiste, riusa se esiste.

### Q: Quando è corretto usare `external: true`?

**A**: Solo quando la rete è gestita **fuori** da compose (es. creata manualmente, da un altro stack, o da infrastruttura). In un setup multi-overlay dove tutti i file sono dello stesso stack, mai.

### Q: Come ci si accorge di un network split?

**A**: Sintomi tipici:
- Reverse proxy ritorna 502/500 ma i servizi individuali funzionano
- `docker exec <container> nslookup <service>` ritorna NXDOMAIN
- `docker network inspect` mostra container su reti diverse
- Healthcheck passa ma le route falliscono

### Q: Il workaround `docker network connect` è sicuro?

**A**: Sì, come fix temporaneo. Connette un container a una rete aggiuntiva senza restart. Ma non sopravvive a un `docker compose up` successivo — va rifatto ogni volta. Il fix strutturale (`name:`) è l'unica soluzione duratura.

### Q: Perché `docker ps` non mostra il problema?

**A**: `docker ps` mostra lo stato del container (running/healthy), non la topologia di rete. Per vedere le reti: `docker inspect <container> --format '{{json .NetworkSettings.Networks}}'` oppure `docker network inspect <network>`.

### Q: Come testare la coerenza PRIMA del deploy?

**A**:
```bash
# 1. Validare il merge dei compose files
docker compose -f a.yml -f b.yml -f c.yml config

# 2. Verificare che tutte le reti siano nominate
docker compose -f a.yml -f b.yml -f c.yml config | grep -A2 "^networks:"

# 3. Simulare il deploy senza eseguirlo
docker compose -f a.yml -f b.yml -f c.yml -p myproject up --dry-run
```

---

## Principi GEDI Applicati

| # | Principio | Lezione |
|---|-----------|---------|
| 1 | Measure Twice, Cut Once | `docker compose config` è il "measure" — non deployare senza validare |
| 10 | Testudo Formation | Ogni gate deve coprire un vettore di failure — la rete era lo scudo mancante |
| 11 | Victory Before Battle | `name:` previene il problema by design — non serve fix post-deploy |
| 12 | Black Swan Resilience | Un split silenzioso è un cigno nero — serve detection esplicita |
| 15 | Known Bug Over Chaos | Meglio un `exit 1` esplicito che un deploy apparentemente OK ma spezzato |
| 16 | Electrical Socket Pattern | deploy.sh deve garantire coerenza — il chiamante non deve sapere di reti interne |

---

## Docker Compose Governance Rules (S87)

> Regole operative per evitare network split, container orfani, e volumi duplicati.

### Regola 1: Project Name Unico

| Chiave | Valore |
|---|---|
| **Project name** | `easyway-prod` |
| **Come** | `COMPOSE_PROJECT_NAME=easyway-prod` nel `.env` di `~/easyway-infra/` |
| **Perche** | Un solo owner di reti, volumi, container. Cambiare project name crea duplicati orfani |

### Regola 2: Network con `name:` Esplicito

```yaml
networks:
  easyway-net:
    name: easyway-net    # SEMPRE. Mai senza name, mai external: true
    driver: bridge
```

### Regola 3: MAI `docker run` per Servizi Compose

| Comando | Quando |
|---|---|
| `docker compose -f docker-compose.apps.yml up -d <service>` | Creare/aggiornare un servizio |
| `docker compose -f docker-compose.apps.yml restart <service>` | Restart senza cambi config |
| `docker run` | MAI per servizi definiti nel compose. Solo per container one-shot/debug |

**Eccezione S87**: container creati con project name diverso vanno prima migrati. Il docker run e accettabile come bridge temporaneo durante la migrazione.

### Regola 4: Un Solo Compose File per Ambiente

Oggi: `docker-compose.yml` (base) + `docker-compose.apps.yml` (prod) + `docker-compose.prod.yml` (overlay).
Target: consolidare in `docker-compose.apps.yml` unico per produzione. Meno file = meno rischio di split.

### Regola 5: Verifica Pre-Deploy

Prima di ogni `docker compose up`:

```bash
# 1. Validate config
docker compose -f docker-compose.apps.yml config --quiet

# 2. Check network names (must have explicit name:)
docker compose -f docker-compose.apps.yml config | grep -A2 "^networks:"

# 3. Check project name
echo $COMPOSE_PROJECT_NAME  # Must be easyway-prod
```

### Regola 6: Container Inventory Aggiornato

Dopo ogni cambio compose: aggiornare `infrastructure/container-inventory.md` con stato reale.

```bash
docker ps --format "table {{.Names}}\t{{.Image}}\t{{.Status}}\t{{.Ports}}"
```

### Anti-Pattern da Evitare

| Anti-Pattern | Conseguenza | Alternativa |
|---|---|---|
| `docker run` per servizio compose | Container fuori progetto, nome conflitto | `docker compose up -d <service>` |
| Cambiare `-p` project name | Volumi orfani, reti duplicate | Sempre `easyway-prod` |
| `external: true` in overlay | Network split (GEDI Case #25) | `name: easyway-net` |
| `docker compose up` senza `.env` | Variabili mancanti, fail o default sbagliati | Sempre `source /opt/easyway/.env.secrets` prima |
| Deploy senza `config --quiet` | Errori YAML silenti | Sempre validare prima |

---

## Riferimenti

- **PR #294** (easyway-infra): fix strutturale `name: easyway-net`
- **GEDI Case #25**: La Rete Spezzata
- **Docker Compose Networking docs**: `name` vs `external` vs default behavior
- **deploy.sh**: `easyway-infra/scripts/deploy.sh` — Testudo check + Coherence Gate
