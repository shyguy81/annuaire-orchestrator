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
- Database: `./data/contacts.db`

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
```

### Variables d'environnement

| Variable | Default | Description |
|----------|---------|-------------|
| `DATABASE_URL` | `sqlite:///./data/contacts.db` | String de connexion DB |
| `LOG_LEVEL` | `INFO` | Niveau de log |

---

## Serveur MCP

**Localisation:** `mcp-fast-mcp/docker-compose.yml`

### Démarrer seul

```bash
cd mcp-fast-mcp
docker compose up --build
```

**Accès:**
- MCP Server: http://localhost:8001
- Health: http://localhost:8001/health

**Important:** MCP appelle le backend en local (http://localhost:8000). Si backend n'est pas actif → erreurs.

### Endpoints disponibles

```bash
# Chercher
curl -X POST http://localhost:8001/search \
  -H "Content-Type: application/json" \
  -d '{"query":"test","limit":10}'

# Résumé
curl -X POST http://localhost:8001/summary \
  -H "Content-Type: application/json" \
  -d '{"contact_id":"<ID>"}'

# Suggestions
curl -X POST http://localhost:8001/suggestions \
  -H "Content-Type: application/json" \
  -d '{"partial_name":"te","limit":5}'
```

### Variables d'environnement

| Variable | Default | Description |
|----------|---------|-------------|
| `BACKEND_URL` | `http://localhost:8000` | URL backend (pour appels internes) |
| `LOG_LEVEL` | `INFO` | Niveau de log |

---

## Lancer les 2 Services Ensemble

### Option 1: Terminal Séparé (Simple)

```bash
# Terminal 1: Backend
cd backend-fastapi
docker compose up --build

# Terminal 2: MCP
cd mcp-fast-mcp
docker compose up --build
```

Puis tester:
```bash
# Backend OK?
curl http://localhost:8000/health

# MCP OK?
curl http://localhost:8001/health

# MCP appelle backend?
curl -X POST http://localhost:8001/search -d '{"query":"test"}'
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

### Option 3: Docker Network (Recommandé Production)

Créer un réseau partagé pour que les services communiquent:

**backend-fastapi/docker-compose.yml:**
```yaml
services:
  backend:
    networks:
      - annuaire-net

networks:
  annuaire-net:
    driver: bridge
```

**mcp-fast-mcp/docker-compose.yml:**
```yaml
services:
  mcp:
    environment:
      - BACKEND_URL=http://backend:8000  # Nom du service backend
    networks:
      - annuaire-net
    depends_on:
      - backend  # Attendre que backend soit prêt

  backend:
    image: annuaire-contacts-backend:latest
    networks:
      - annuaire-net

networks:
  annuaire-net:
    driver: bridge
```

Puis:
```bash
# Depuis racine
cd mcp-fast-mcp && docker compose up --build
```

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
# Terminal 1: Backend
cd backend-fastapi
docker compose up --build
# Attendre: "Uvicorn running on http://0.0.0.0:8000"

# Terminal 2: MCP (une fois backend prêt)
cd ../mcp-fast-mcp
docker compose up --build
# Attendre: "Uvicorn running on http://0.0.0.0:8001"

# Terminal 3: Tests
curl http://localhost:8000/health          # ✅ OK?
curl http://localhost:8001/health          # ✅ OK?
curl -X POST http://localhost:8001/search \
  -H "Content-Type: application/json" \
  -d '{"query":"test"}'                    # ✅ OK?
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
