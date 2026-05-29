# Annuaire de Contacts — Orchestrateur

Système de gestion contacts avec API FastAPI + serveur MCP pour intégration IA.

**État:** MVP fonctionnel (v0.1.0)  
**Stack:** FastAPI + SQLAlchemy + FastAPI MCP wrapper  
**Deploy:** Docker Compose (dev) / À venir: production

---

## 🚀 Démarrage (5 min)

**Services autonomes (chaque dossier = son docker-compose.yml)**

```bash
# Terminal 1: Backend + MariaDB
cd annuaire-fastapi && docker compose up --build

# Terminal 2: MCP + Nginx (une fois backend prêt)
cd mcp-fast-mcp && docker compose up --build
```

**Accès:**
- Backend API: http://localhost:8000
- Backend Swagger UI: http://localhost:8000/docs
- MCP HTTP: http://localhost (port 80, via Nginx)
- MCP HTTPS: https://localhost (port 443, certificat auto-signé)
- Database: MariaDB port 3307 (localhost:3307 → container:3306)

→ **Détails:** Voir [`docs/SERVICES.md`](./docs/SERVICES.md) et [`DIAGNOSTIC.md`](./DIAGNOSTIC.md)

**Tester:**
```bash
# Créer un contact (Backend)
curl -X POST http://localhost:8000/contacts \
  -H "Content-Type: application/json" \
  -d '{"nom":"Alice","email":"alice@example.com"}'

# Chercher via MCP (HTTP, via Nginx)
curl -X POST http://localhost/search \
  -H "Content-Type: application/json" \
  -d '{"query":"Alice","limit":10}'

# Chercher via MCP (HTTPS, certificat auto-signé)
curl -k -X POST https://localhost/search \
  -H "Content-Type: application/json" \
  -d '{"query":"Alice","limit":10}'
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
│   ├── docs/           → Documentation complète
│   ├── DIAGNOSTIC.md   → Changements récents
│   └── CHANGELOG.md    → Historique
├── annuaire-fastapi/     ← API REST (CRUD contacts)
│                          + MariaDB database
├── mcp-fast-mcp/        ← Serveur MCP (recherche IA)
│                          + Nginx reverse proxy
│                          + TLS certificates
└── data/               → Données persistantes (volumes)
```

**Flux:**
```
HTTP Client (8000)               IA Client (HTTPS 443)
  ↓                                 ↓
Backend API                     Nginx Reverse Proxy (80/443)
  ↓                                 ↓
MariaDB Database            MCP Server (8000 internal)
                                    ↓
                            Backend API (httpx client)
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

### MCP (80/443 via Nginx) — Pour IA/Agents

| Méthode | Endpoint | Description |
|---------|----------|-------------|
| `POST` | `/search` | Chercher contacts |
| `POST` | `/summary` | Résumé d'un contact |
| `POST` | `/suggestions` | Autocomplete noms |
| `GET` | `/contacts` | Lister (contexte IA) |
| `GET` | `/health` | Health check |

*Accès: HTTP (port 80) ou HTTPS (port 443, certificat auto-signé)*

→ **Détails complets:** [`docs/MCP.md`](./docs/MCP.md)

---

## ⚙️ Configuration

### Variables d'environnement

Copier `.env.example` → `.env` et adapter (optionnel, valeurs par défaut existantes):

```bash
cp annuaire-fastapi/.env.example annuaire-fastapi/.env
cp mcp-fast-mcp/.env.example mcp-fast-mcp/.env
```

**Backend (MariaDB):**
```env
DATABASE_URL=mysql+pymysql://annuaire_user:annuaire_password@mariadb:3306/annuaire_contacts
LOG_LEVEL=INFO
```
*Note: MariaDB credentials et database sont gérés par docker-compose.yml*

**MCP (Nginx + TLS):**
```env
BACKEND_URL=http://localhost:8000  # Appels internes vers backend
LOG_LEVEL=INFO
```
*Note: Nginx TLS via certificats dans docker/certs/ (mkcert autosignés)*

---

## 🧪 Commandes Utiles

```bash
# Démarrer
docker compose up --build

# Logs temps réel
docker compose logs -f

# Tests (backend)
cd annuaire-fastapi && pytest -v

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

**Immédiat:**
- [ ] Vérifier: Backend ↔ MCP communication sur réseau Docker
- [ ] Clarifier: Usage du dossier `data/` (persistence volumes?)
- [ ] Nettoyer: `mcp-fast-mcp/scripts/` (vide actuellement)

**Court-terme:**
- [ ] Tests unitaires (pytest coverage > 80%)
- [ ] Endpoint `/health` sur tous les services
- [ ] Documentation des flux internes (Backend ↔ MCP)

**Moyen-terme:**
- [ ] JWT authentification
- [ ] Monitoring (Prometheus)
- [ ] CI/CD (GitHub Actions)
- [ ] Production TLS (Let's Encrypt ou CA interne au lieu de mkcert)

**Long-terme:**
- [ ] PostgreSQL upgrade (si scalabilité nécessaire)
- [ ] Intégration Claude API complète
- [ ] Replica/scaling horizontale des services

---

## 📝 Changements Récents (2026-04-07)

**Architecture Update:**
- ✨ Backend: SQLite → **MariaDB 11.2** (automatique dans docker-compose)
- ✨ MCP: Port 8001 → **80/443 via Nginx** (reverse proxy + TLS)
- ✨ Ajout: `docker/` folder (certs TLS + nginx config)

**→ Voir détails:** [`DIAGNOSTIC.md`](./DIAGNOSTIC.md) et [`CHANGELOG.md`](./CHANGELOG.md)

---

## 💬 Support

- **Problème technique?** → Voir [`docs/DEPLOYMENT.md`](./docs/DEPLOYMENT.md) → Troubleshooting
- **Veux contribuer?** → Lire [`docs/CONTRIBUTING.md`](./docs/CONTRIBUTING.md)
- **Questions API?** → Voir [`docs/API.md`](./docs/API.md)

---

**Version:** 0.1.0 MVP  
**Dernière update:** 2026-04-07
