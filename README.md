# Annuaire de Contacts — Orchestrateur

Système de gestion contacts avec API FastAPI + serveur MCP pour intégration IA.

**État:** MVP fonctionnel (v0.1.0)  
**Stack:** FastAPI + SQLAlchemy + FastAPI MCP wrapper  
**Deploy:** Docker Compose (dev) / À venir: production

---

## 🚀 Démarrage (5 min)

```bash
# Depuis la racine (annuaire-contacts/)
docker compose up --build

# Accès
- Backend: http://localhost:8000
- MCP: http://localhost:8001
- Swagger UI: http://localhost:8000/docs
```

**Tester:**
```bash
# Créer un contact
curl -X POST http://localhost:8000/contacts \
  -H "Content-Type: application/json" \
  -d '{"nom":"Alice","email":"alice@example.com"}'

# Chercher via MCP
curl -X POST http://localhost:8001/search \
  -H "Content-Type: application/json" \
  -d '{"query":"Alice"}'
```

**Plus d'exemples:** Voir [`QUICKSTART.md`](./QUICKSTART.md)

---

## 📖 Documentation

**Nouveau?** Commencer par [`docs/INDEX.md`](./docs/INDEX.md) pour naviguer.

| Document | Contenu |
|----------|---------|
| **[docs/INDEX.md](./docs/INDEX.md)** | 📍 Navigation guide (LIS-LE PREMIER!) |
| **[QUICKSTART.md](./QUICKSTART.md)** | Exemples curl des 5 endpoints |
| **[docs/API.md](./docs/API.md)** | Backend endpoints détaillés |
| **[docs/MCP.md](./docs/MCP.md)** | Serveur MCP pour IA/agents |
| **[docs/ARCHITECTURE.md](./docs/ARCHITECTURE.md)** | Design système |
| **[docs/DEPLOYMENT.md](./docs/DEPLOYMENT.md)** | Production, scaling, monitoring |
| **[docs/CONTRIBUTING.md](./docs/CONTRIBUTING.md)** | Workflow développement |
| **[.claude/rules/](./​.claude/rules/)** | 5 règles non-négociables |

---

## 🏗️ Architecture

```
annuaire-contacts/
├── orchestrator/        ← Ce dossier (config + doc)
│   └── docs/           → Toute la documentation
├── backend-fastapi/     ← API REST (CRUD contacts)
├── mcp-fast-mcp/        ← Serveur MCP (recherche IA)
└── docker-compose.yml   ← Orchestration
```

**Flux:**
```
Client (HTTP / IA)
  → Backend (8000) : CRUD sur SQLite
  → MCP (8001) : Wrapper pour agents IA
```

---

## 📦 Schéma Contact

```json
{
  "id": "uuid",
  "nom": "string (required)",
  "email": "string (required, unique)",
  "telephone": "string (optional)",
  "adresse": "string (optional)",
  "organisation": "string (optional)",
  "tags": ["string"] (optional)
}
```

---

## 📡 API Endpoints

### Backend (8000)

| Méthode | Endpoint | Description |
|---------|----------|-------------|
| `POST` | `/contacts` | Créer |
| `GET` | `/contacts` | Lister |
| `GET` | `/contacts/{id}` | Détails |
| `PUT` | `/contacts/{id}` | Mettre à jour |
| `DELETE` | `/contacts/{id}` | Supprimer |
| `GET` | `/contacts/search/{query}` | Chercher |

→ **Détails complets:** [`docs/API.md`](./docs/API.md)

### MCP (8001) — Pour IA/Agents

| Méthode | Endpoint | Description |
|---------|----------|-------------|
| `POST` | `/search` | Chercher contacts |
| `POST` | `/summary` | Résumé d'un contact |
| `POST` | `/suggestions` | Autocomplete noms |
| `GET` | `/contacts` | Lister (contexte IA) |

→ **Détails complets:** [`docs/MCP.md`](./docs/MCP.md)

---

## ⚙️ Configuration

### Variables d'environnement

Copier `.env.example` → `.env` et adapter:

```bash
cp .env.example .env
```

**Backend:**
```env
DATABASE_URL=sqlite:///./data/contacts.db
LOG_LEVEL=INFO
```

**MCP:**
```env
BACKEND_URL=http://backend:8000
LOG_LEVEL=INFO
```

---

## 🧪 Commandes Utiles

```bash
# Démarrer
docker compose up --build

# Logs temps réel
docker compose logs -f

# Tests (backend)
cd backend-fastapi && pytest -v

# Arrêter
docker compose down

# Nettoyer (reset DB)
rm -rf data/ && docker compose up --build
```

---

## 🚨 Règles Importantes

Lire **absolument** [`CLAUDE.md`](./CLAUDE.md) et [`.claude/rules/`](./.claude/rules/)

**TL;DR:**
1. Orchestrator = orchestration uniquement (pas de logique métier)
2. Ne jamais modifier backends depuis orchestrator
3. Clients générés (pas d'URLs hardcodées)
4. Git commits depuis les repos techniques
5. Docker Compose = configuration source de vérité

---

## 🛣️ Prochaines Étapes

- [ ] Tests unitaires (pytest coverage > 80%)
- [ ] JWT authentification
- [ ] PostgreSQL upgrade
- [ ] Monitoring (Prometheus)
- [ ] CI/CD (GitHub Actions)
- [ ] Intégration Claude API complète

---

## 💬 Support

- **Problème technique?** → Voir [`docs/DEPLOYMENT.md`](./docs/DEPLOYMENT.md) → Troubleshooting
- **Veux contribuer?** → Lire [`docs/CONTRIBUTING.md`](./docs/CONTRIBUTING.md)
- **Questions API?** → Voir [`docs/API.md`](./docs/API.md)

---

**Version:** 0.1.0 MVP  
**Dernière update:** 2026-04-07
