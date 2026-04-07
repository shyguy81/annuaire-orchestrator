# Guide de Déploiement

Services autonomes: chacun peut être déployé indépendamment.

---

## Environnements

### Développement (Local)

**Terminal 1: Backend**
```bash
cd backend-fastapi
cp .env.example .env  # Configurer si besoin
docker compose up --build
```

**Terminal 2: MCP**
```bash
cd mcp-fast-mcp
cp .env.example .env  # Configurer si besoin
docker compose up --build
```

**Accès:**
- Backend: http://localhost:8000
- MCP: http://localhost:8001
- Swagger UI: http://localhost:8000/docs
- Database: `backend-fastapi/data/contacts.db`

→ **Détails:** Voir `docs/SERVICES.md`

---

### Staging / Production

À adapter selon votre infrastructure.

---

## Configuration

### Variables d'Environnement

**Backend** (`backend-fastapi/.env`):
```env
DATABASE_URL=sqlite:///./data/contacts.db
LOG_LEVEL=INFO
```

**MCP** (`mcp-fast-mcp/.env`):
```env
BACKEND_URL=http://localhost:8000  # Ou IP backend en prod
LOG_LEVEL=INFO
```

### Fichiers

- `backend-fastapi/.env.example` — Template backend
- `mcp-fast-mcp/.env.example` — Template MCP
- **NE PAS committer** les fichiers `.env` (secrets)

**Setup:**
```bash
# Backend
cd backend-fastapi
cp .env.example .env
# Éditer .env si besoin

# MCP
cd ../mcp-fast-mcp
cp .env.example .env
# Éditer .env si besoin
```

---

## Persistent Database (Production)

### Actuellement (SQLite)

Données sauvegardées à `/data/contacts.db`.

**Volume Docker:**
```yaml
services:
  backend:
    volumes:
      - ./data:/app/data  # Persiste données entre redémarrages
```

### Upgrade vers PostgreSQL

1. **Changer backend `requirements.txt`:**
   ```
   psycopg2-binary==2.9.9  # PostgreSQL driver
   ```

2. **Ajouter service PostgreSQL à `docker-compose.yml`:**
   ```yaml
   postgres:
     image: postgres:15
     environment:
       POSTGRES_PASSWORD: secret
       POSTGRES_DB: annuaire
     volumes:
       - postgres_data:/var/lib/postgresql/data
     ports:
       - "5432:5432"

   backend:
     environment:
       DATABASE_URL=postgresql://postgres:secret@postgres:5432/annuaire
     depends_on:
       - postgres
   ```

3. **Modifier `database.py`:**
   ```python
   DATABASE_URL = os.getenv(
     "DATABASE_URL", 
     "postgresql://user:pass@localhost/annuaire"
   )
   engine = create_engine(DATABASE_URL)  # Pas besoin check_same_thread pour PG
   ```

4. **Redémarrer:**
   ```bash
   docker compose up --build
   ```

---

## Scaling

### 1 Serveur (Actuel)

```
docker-compose up
```

- Backend + MCP + SQLite (1 container chacun)
- OK pour MVP (< 1000 contacts)

### Multi-conteneurs (Scaling Horizontal)

```yaml
version: '3.9'
services:
  backend:
    deploy:
      replicas: 3  # 3 instances
  
  mcp:
    deploy:
      replicas: 2  # 2 instances
  
  postgres:  # Base de données centralisée
    image: postgres:15
```

**Load Balancer requis:** Nginx / HAProxy

---

## Authentification (À Ajouter)

### JWT Tokens

**Backend `requirements.txt`:**
```
PyJWT==2.8.1
python-jose==3.3.0
```

**Backend `main.py`:**
```python
from fastapi import Depends, HTTPException
from jose import JWTError, jwt

SECRET_KEY = "your-secret-key"

@app.post("/token")
async def login(username: str, password: str):
    # Vérifier credentials
    token = jwt.encode({"sub": username}, SECRET_KEY)
    return {"access_token": token}

async def verify_token(token: str = Depends(HTTPBearer())):
    try:
        payload = jwt.decode(token, SECRET_KEY)
        return payload
    except JWTError:
        raise HTTPException(status_code=401)

@app.get("/contacts")
async def list_contacts(current_user = Depends(verify_token)):
    # Endpoints protégés
    ...
```

---

## CI/CD (GitHub Actions)

**`.github/workflows/deploy.yml`:**
```yaml
name: Deploy

on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      
      - name: Build Docker images
        run: docker compose build
      
      - name: Run tests
        run: docker compose run backend pytest
      
      - name: Push to registry
        run: |
          docker tag annuaire-contacts-backend myregistry/backend:latest
          docker push myregistry/backend:latest
```

---

## Health Checks

### Docker Compose

```yaml
services:
  backend:
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8000/health"]
      interval: 10s
      timeout: 5s
      retries: 3
```

**Commande manuelle:**
```bash
curl http://localhost:8000/health
curl http://localhost:8001/health
```

---

## Monitoring (Optionnel)

### Logs

```bash
# Backend
docker compose logs -f backend

# MCP
docker compose logs -f mcp

# Tout
docker compose logs -f
```

### Prometheus (Futur)

**Backend `requirements.txt`:**
```
prometheus-client==0.18.0
```

**Exposer métriques:**
```python
from prometheus_client import Counter, Histogram

request_count = Counter('requests_total', 'Total requests')
request_duration = Histogram('request_duration_seconds', 'Request duration')

@app.post("/contacts")
async def create_contact(...):
    request_count.inc()
    with request_duration.time():
        # Logique
    ...
```

**Accès:** `http://localhost:8000/metrics`

---

## Backup (Production)

### SQLite

```bash
# Backup manuel
cp data/contacts.db data/contacts.db.backup

# Restaurer
cp data/contacts.db.backup data/contacts.db
```

### PostgreSQL

```bash
# Backup
docker exec annuaire-postgres pg_dump -U postgres annuaire > backup.sql

# Restaurer
docker exec -i annuaire-postgres psql -U postgres annuaire < backup.sql
```

### Automatisé (Cron)

```bash
# /etc/cron.d/annuaire-backup
0 2 * * * cd /media/tjeanbaptiste/PUBLIC/annuaire-contacts && \
  docker compose exec -T postgres pg_dump -U postgres annuaire | \
  gzip > /backups/annuaire-$(date +\%Y\%m\%d).sql.gz
```

---

## Troubleshooting

| Problème | Cause | Solution |
|----------|-------|----------|
| `error while creating mount source path` | Permission dossier | `chmod 777 ./data` |
| `dependency cycle detected` | Backend dépend de MCP | Retirer `depends_on: - mcp` du backend |
| `Connection refused (backend)` | MCP démarre avant backend | Ajouter delay ou health check |
| `ImportError: email-validator` | Dépendance manquante | `pip install pydantic[email]` |
| `Permission denied (.db)` | Permissions SQLite | `chmod 666 ./data/contacts.db` |

---

## Roadmap Infra

- [ ] PostgreSQL setup
- [ ] JWT authentification
- [ ] Monitoring Prometheus
- [ ] CI/CD GitHub Actions
- [ ] Kubernetes deployment
- [ ] Rate limiting / Throttling
- [ ] API versioning (v1, v2)
