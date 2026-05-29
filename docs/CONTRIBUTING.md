# Guide de Contribution

---

## Principes

- **5 Règles Non-Négociables:** Lire `.claude/rules/annuaire-contacts-rules.md` avant toute modification
- **Orchestrator ≠ Logique Métier:** Seule l'orchestration et la documentation vont ici
- **Clients Générés:** Jamais d'URLs hardcodées
- **Git Commits:** Depuis les repos techniques (`annuaire-fastapi/`, `mcp-fast-mcp/`)
- **Docker Compose:** Source de vérité pour la configuration

---

## Setup Local

### 1. Cloner + Initialiser

```bash
git clone <repo> annuaire-contacts
cd annuaire-contacts

# Vérifier les 3 repos
ls -la annuaire-fastapi/.git
ls -la mcp-fast-mcp/.git
ls -la orchestrator/.git
```

### 2. Démarrer les services

**Services autonomes → démarrer indépendamment**

**Terminal 1: Backend**
```bash
cd annuaire-fastapi
docker compose up --build
```

**Terminal 2: MCP (une fois backend prêt)**
```bash
cd ../mcp-fast-mcp
docker compose up --build
```

**Accès:**
- Backend API: http://localhost:8000/docs (Swagger)
- MCP Health: http://localhost:8001/health
- Base de données: `annuaire-fastapi/data/contacts.db`

→ **Détails:** Voir `docs/SERVICES.md`

---

## Workflow de Développement

### Ajouter une Nouvelle Fonctionnalité

#### 1. Planifier (Orchestrator)

```bash
cd orchestrator
# Documenter la feature dans docs/ avant de coder
# Ajouter des exemples curl si endpoint API
```

#### 2. Implémenter Backend

```bash
cd annuaire-fastapi

# Créer une branche
git checkout -b feature/new-endpoint

# Modifier le code (models.py, main.py, database.py)
# Tester localement
python -m pytest

# Commit depuis le repo backend
git add .
git commit -m "feat: add new endpoint /contacts/filter"
git push origin feature/new-endpoint
```

#### 3. Implémenter MCP (si pertinent)

```bash
cd ../mcp-fast-mcp

# Même workflow
git checkout -b feature/new-mcp-endpoint
# Modifier main.py, models.py, backend_client.py
git add .
git commit -m "feat: add /filter endpoint wrapper"
git push
```

#### 4. Mettre à jour Orchestration

```bash
cd ../orchestrator

# Documenter la feature
# Ajouter des examples curl dans QUICKSTART.md
# Ajouter du détail dans docs/API.md ou docs/MCP.md

git add .
git commit -m "docs: add /filter endpoint documentation"
```

#### 5. Tester l'Ensemble

```bash
# Depuis racine
docker compose down
docker compose up --build

# Tester les endpoints
curl http://localhost:8000/contacts/filter
curl -X POST http://localhost:8001/filter -d '{"...": "..."}'
```

---

## Conventions de Code

### Backend (Python / FastAPI)

**Models:**
```python
# models.py - Toujours définir les deux couches

# Layer 1: Database (SQLAlchemy ORM)
class ContactDB(Base):
    __tablename__ = "contacts"
    id = Column(String, primary_key=True)
    nom = Column(String, nullable=False, index=True)

# Layer 2: API Schema (Pydantic)
class ContactCreate(BaseModel):
    nom: str
    email: EmailStr

class ContactResponse(BaseModel):
    id: str
    nom: str
    model_config = {"from_attributes": True}
```

**Endpoints:**
```python
# main.py - Toujours valider + documenter

@app.post("/contacts", response_model=ContactResponse, status_code=201)
async def create_contact(contact: ContactCreate, db: Session = Depends(get_db)):
    """Créer un contact avec validation d'unicité email"""
    # Validation métier
    existing = db.query(ContactDB).filter(ContactDB.email == contact.email).first()
    if existing:
        raise HTTPException(status_code=400, detail="Email déjà utilisé")
    
    # Persistence
    db_contact = ContactDB(**contact.model_dump())
    db.add(db_contact)
    db.commit()
    db.refresh(db_contact)
    return db_contact
```

### MCP (Python / FastAPI)

**Client:**
```python
# backend_client.py - Toujours async + error handling

async def search_contacts(query: str, limit: int = 10):
    async with await get_backend_client() as client:
        try:
            response = await client.get(f"/contacts/search/{query}")
            response.raise_for_status()
            return response.json()[:limit]
        except httpx.HTTPError as e:
            logger.error(f"Backend error: {e}")
            return []
```

**Endpoints:**
```python
# main.py - Wrapper clean autour du backend

@app.post("/search", response_model=SearchResponse)
async def mcp_search(request: SearchRequest):
    contacts = await search_contacts(request.query, limit=request.limit)
    return SearchResponse(
        query=request.query,
        results=[ContactInfo(**c) for c in contacts],
        count=len(contacts)
    )
```

---

## Testing

### Backend (pytest)

**Structure:**
```bash
annuaire-fastapi/
├── tests/
│   ├── test_models.py
│   ├── test_endpoints.py
│   └── conftest.py
```

**Exemple test:**
```python
# tests/test_endpoints.py
import pytest
from fastapi.testclient import TestClient
from main import app

client = TestClient(app)

def test_create_contact():
    response = client.post("/contacts", json={
        "nom": "Test",
        "email": "test@example.com"
    })
    assert response.status_code == 201
    assert response.json()["nom"] == "Test"

def test_duplicate_email():
    client.post("/contacts", json={
        "nom": "Test",
        "email": "test@example.com"
    })
    response = client.post("/contacts", json={
        "nom": "Test2",
        "email": "test@example.com"  # Doublon
    })
    assert response.status_code == 400
```

**Lancer les tests:**
```bash
cd annuaire-fastapi
pytest -v
```

---

## Git Workflow

### Branches

| Pattern | Usage | Exemple |
|---------|-------|---------|
| `main` | Stable, prêt production | (jamais éditer directement) |
| `feature/*` | Nouvelle feature | `feature/add-export-csv` |
| `fix/*` | Bug fix | `fix/email-validation` |
| `docs/*` | Documentation | `docs/api-guide` |

### Commits

**Format:**
```
<type>(<scope>): <subject>

<body>

Co-Authored-By: Claude Haiku 4.5 <noreply@anthropic.com>
```

**Types:**
- `feat` — Nouvelle fonctionnalité
- `fix` — Bug fix
- `docs` — Documentation uniquement
- `refactor` — Refactoring sans changement comportement
- `test` — Tests uniquement
- `chore` — Deps, config, tooling

**Exemples:**
```bash
# Feature
git commit -m "feat(contacts): add filter by organization"

# Fix
git commit -m "fix(email): validate before uniqueness check"

# Docs
git commit -m "docs(api): add /filter endpoint examples"
```

### Pull Requests

1. **Create PR:**
   ```bash
   git push origin feature/my-feature
   # Créer PR sur GitHub
   ```

2. **PR Title Format:**
   ```
   [Backend] Add filter endpoint for organization
   [MCP] Expose filter via /filter wrapper
   [Docs] Update API documentation
   ```

3. **Description:**
   ```markdown
   ## Changes
   - Add ContactFilter model
   - Implement GET /contacts/filter?org=Acme
   - Add MCP wrapper POST /filter

   ## Testing
   - [ ] Manual test with curl
   - [ ] Unit tests pass
   - [ ] Docker compose up works
   
   ## Related
   Closes #123
   ```

4. **Review & Merge:**
   - Checker les 5 règles
   - Vérifier les tests passent
   - Merge sur `main`

---

## Documentation

### Quand Ajouter Doc

| Changement | Doc | Fichier |
|-----------|-----|---------|
| Nouvel endpoint API | Exemple + paramètres | `docs/API.md` |
| Nouvel endpoint MCP | Modèle + use case | `docs/MCP.md` |
| Nouvelle variable env | Variable + description | `DEPLOYMENT.md` + `.env.example` |
| Nouvel endpoint test | Curl example | `QUICKSTART.md` |
| Changement architecture | Diagram + explication | `ARCHITECTURE.md` |

### Template Doc Endpoint

```markdown
### 7. [NomDuFeature]

\`\`\`
METHOD /path/{param}
\`\`\`

**Purpose:** Courte description

**Body:**
\`\`\`json
{...}
\`\`\`

**Réponse (200):**
\`\`\`json
{...}
\`\`\`

**Erreurs:**
- \`400 Bad Request\` — Raison
- \`404 Not Found\` — Raison

**Exemple curl:**
\`\`\`bash
curl ...
\`\`\`
```

---

## Checklist avant Commit

- [ ] Code passe les tests (`pytest`)
- [ ] Pas de logique métier dans orchestrator
- [ ] URLs pas hardcodées (clients générés)
- [ ] Commit depuis le bon repo (backend ou mcp)
- [ ] Message commit clair et concis
- [ ] Documentation mise à jour (API.md, MCP.md, etc.)
- [ ] `docker compose up --build` fonctionne
- [ ] Endpoints testés avec curl

---

## Troubleshooting Dev

### Import errors dans IDE

**Problem:** PyCharm/VSCode dit "Module not found"

**Solution:**
```bash
# Backend
cd annuaire-fastapi
pip install -e .

# MCP
cd ../mcp-fast-mcp
pip install -e .
```

### Tests échouent localement

```bash
# Vérifier la DB
rm ./data/contacts.db
docker compose down
docker compose up

# Relancer tests
cd annuaire-fastapi
pytest -v
```

### Docker build lent

```bash
# Limiter les layers
docker system prune -a
docker compose build --no-cache
```

---

## Roadmap Contributions Bienvenues

- [ ] Tests unitaires complets (coverage > 80%)
- [ ] PostgreSQL upgrade script
- [ ] JWT authentification
- [ ] Rate limiting / Throttling
- [ ] Export CSV/JSON
- [ ] Bulk import de contacts
- [ ] Soft delete (archive)
- [ ] Activity logs (audit trail)
- [ ] API versioning (v1, v2)

Créer une issue avant de commencer!
