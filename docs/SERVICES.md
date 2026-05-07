# Services — Démarrage Autonome

Chaque service est autonome avec son propre `docker-compose.yml`.

---

## Backend FastAPI

**Localisation:** `backend-fastapi/docker-compose.yml`

### Démarrer seul

```bash
cd backend-fastapi
docker compose up --build
```

**Accès:**
- API: http://localhost:8000
- Swagger UI: http://localhost:8000/docs
- Health: http://localhost:8000/health
- Database: MariaDB (port 3307 local → 3306 interne, container: `annuaire-mariadb`)

### Endpoints disponibles

```bash
# Créer un contact
curl -X POST http://localhost:8000/contacts \
  -H "Content-Type: application/json" \
  -d '{"nom":"Test","email":"test@example.com"}'

# Lister
curl http://localhost:8000/contacts

# Chercher
curl "http://localhost:8000/contacts/search/test"

# Health check
curl http://localhost:8000/health
```

### Variables d'environnement

| Variable | Default | Description |
|----------|---------|-------------|
| `DATABASE_URL` | `mysql+pymysql://annuaire_user:annuaire_password@mariadb:3306/annuaire_contacts` | String de connexion MariaDB |
| `LOG_LEVEL` | `INFO` | Niveau de log |

**MariaDB Automatique:**
- Démarré automatiquement dans docker-compose
- Healthcheck: ping mariadb
- Volume: `mariadb_data` (persistence)

---

## Serveur MCP (avec Nginx Reverse Proxy)

**Localisation:** `mcp-fast-mcp/docker-compose.yml`

### Démarrer seul

```bash
cd mcp-fast-mcp
docker compose up --build
```

**Accès:**
- MCP HTTP: http://localhost (port 80)
- MCP HTTPS: https://localhost (port 443, certificat autosigné)
- Hostname: `annuaire-mcp.local.docker.dev` (ou ajuster `/etc/hosts`)
- Health: http://localhost/health

**Important:**
1. MCP appelle le backend en local (http://localhost:8000) → Backend doit être démarré
2. Nginx expose MCP en ports publics (80/443)
3. TLS: Certificats autosignés dans `docker/certs/` (créés via mkcert)

### Endpoints disponibles

```bash
# Chercher (via Nginx HTTP)
curl -X POST http://localhost/search \
  -H "Content-Type: application/json" \
  -d '{"query":"test","limit":10}'

# Chercher (via HTTPS, ignorer certif auto-signé)
curl -k -X POST https://localhost/search \
  -H "Content-Type: application/json" \
  -d '{"query":"test","limit":10}'

# Résumé
curl -X POST http://localhost/summary \
  -H "Content-Type: application/json" \
  -d '{"contact_id":"<ID>"}'

# Suggestions
curl -X POST http://localhost/suggestions \
  -H "Content-Type: application/json" \
  -d '{"partial_name":"te","limit":5}'

# Health check
curl http://localhost/health
```

### Variables d'environnement

| Variable | Default | Description |
|----------|---------|-------------|
| `BACKEND_URL` | `http://localhost:8000` | URL backend (pour appels internes) |
| `LOG_LEVEL` | `INFO` | Niveau de log |

### Configuration Nginx

**Fichiers:**
- `docker/nginx/nginx.conf` — Config globale Nginx
- `docker/nginx/conf.d/default.conf` — VHost pour annuaire-mcp
- `docker/certs/` — Certificats TLS (mkcert autosignés)

**Architecture Docker-Compose:**
- Service `annuaire-mcp` (port 8000 interne)
- Service `nginx` (ports 80/443 externes, reverse proxy)
- Network `mcp-network` (bridge, communication intra-compose)

---

## Lancer les 2 Services Ensemble

### Option 1: Terminal Séparé (Simple)

```bash
# Terminal 1: Backend (inclut MariaDB)
cd backend-fastapi
docker compose up --build
# Attendre: "Uvicorn running on http://0.0.0.0:8000"

# Terminal 2: MCP (inclut Nginx)
cd mcp-fast-mcp
docker compose up --build
# Attendre: "Uvicorn running on http://0.0.0.0:8000" (MCP interne) et nginx running
```

Puis tester:
```bash
# Backend OK?
curl http://localhost:8000/health

# MCP via Nginx OK?
curl http://localhost/health

# MCP appelle backend?
curl -X POST http://localhost/search -d '{"query":"test","limit":10}'

# HTTPS via Nginx (ignorer certificat auto-signé)?
curl -k https://localhost/health
```

---

### Option 2: Script d'Orchestration (Avancé)

**Créer `start-all.sh` à la racine:**

```bash
#!/bin/bash

# Démarrer backend en background
cd backend-fastapi
docker compose up -d --build
BACKEND_PID=$!
echo "Backend démarré (PID: $BACKEND_PID)"

# Attendre que backend soit prêt
sleep 5

# Démarrer MCP
cd ../mcp-fast-mcp
docker compose up --build
MCP_PID=$!
echo "MCP démarré (PID: $MCP_PID)"

# Graceful shutdown
trap "kill $BACKEND_PID $MCP_PID" EXIT
wait
```

**Lancer:**
```bash
chmod +x start-all.sh
./start-all.sh
```

**Arrêter:**
```bash
# Ctrl+C ou:
cd backend-fastapi && docker compose down
cd ../mcp-fast-mcp && docker compose down
```

---

### Option 3: Architecture Actuelle (Services Autonomes)

**État réel:** Les services sont déjà configurés pour autonomie maximale.

**Backend (docker-compose.yml):**
- ✅ Service `mariadb` (port 3307 mapping)
- ✅ Service `backend` (port 8000)
- ✅ Healthcheck intégré

**MCP (docker-compose.yml):**
- ✅ Service `annuaire-mcp` (port 8000 interne)
- ✅ Service `nginx` (ports 80/443 publics, reverse proxy)
- ✅ Network `mcp-network` (bridge)
- ✅ Dépendance: nginx → annuaire-mcp

**Lancer ensemble:**
```bash
# Terminal 1
cd backend-fastapi && docker compose up --build

# Terminal 2 (dans un autre terminal, après que backend soit prêt)
cd mcp-fast-mcp && docker compose up --build
```

**Note:** Les services ne partagent pas encore un réseau unique pour communication intra-compose. Backend est accessible via `localhost:8000` depuis hôte ou depuis MCP container via `host.docker.internal:8000` si nécessaire.

---

## Autonomie vs Orchestration

| Aspect | Autonome | Avec Network |
|--------|----------|--------------|
| **Démarrage** | Indépendant | Lancer MCP suffisant |
| **Communication** | localhost:8000 | Service name (backend) |
| **Simplicité dev** | ✅ Facile | Plus complexe |
| **Production** | ❌ Manuel | ✅ Orchestré |
| **Tests** | ✅ Chacun teste seul | Possible |

---

## Logs & Debugging

### Voir les logs

```bash
# Backend
cd backend-fastapi && docker compose logs -f

# MCP
cd mcp-fast-mcp && docker compose logs -f
```

### Vérifier la connexion Backend ↔ MCP

```bash
# Depuis container MCP
docker compose exec mcp curl http://localhost:8000/health
```

Si erreur `Connection refused` → Backend n'est pas démarré.

---

## Checklist Démarrage Complet

```bash
# Terminal 1: Backend (+ MariaDB)
cd backend-fastapi
docker compose up --build
# Attendre: 
#   - "mariadb_1 | ready for connections"
#   - "Uvicorn running on http://0.0.0.0:8000"

# Terminal 2: MCP (+ Nginx) — une fois backend prêt
cd ../mcp-fast-mcp
docker compose up --build
# Attendre: 
#   - "Uvicorn running on http://0.0.0.0:8000" (MCP interne)
#   - "nginx_1 | [notice] worker process started" ou "GET / HTTP/1.1" 200

# Terminal 3: Tests
curl http://localhost:8000/health          # Backend ✅ OK?
curl http://localhost/health                # MCP via Nginx ✅ OK?
curl -X POST http://localhost/search \
  -H "Content-Type: application/json" \
  -d '{"query":"test","limit":10}'         # MCP search ✅ OK?
```

---

## Arrêter

```bash
# Chaque terminal (Ctrl+C)
docker compose down

# Ou depuis chaque dossier
cd backend-fastapi && docker compose down
cd ../mcp-fast-mcp && docker compose down
```

---

## Avantages Autonomie

✅ Chaque service peut être démarré indépendamment  
✅ Dev backend n'a pas besoin de MCP  
✅ Tests isolés par service  
✅ Déploiement granulaire possible  
✅ Respecte les 5 règles (orchestrator n'orchestre rien)
