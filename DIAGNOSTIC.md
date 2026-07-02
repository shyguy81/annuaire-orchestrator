# Diagnostic — Changements d'Architecture (2026-07-02)

## Résumé des Ajustements

La structure du projet a évolué depuis la dernière documentation. Voici les changements identifiés.

---

## 📁 Structure Réelle vs Documentée

### Backend FastAPI

| Aspect | Documentation | Réalité | Statut |
|--------|---|---|---|
| **Base de données** | SQLite (`./data/contacts.db`) | PostgreSQL 15 (port 5432) | ❌ **CHANGÉ** |
| **Variables env** | `DATABASE_URL=sqlite:///...` | `DATABASE_URL=postgresql+psycopg2://...` | ❌ **CHANGÉ** |
| **Port API** | 8000 | 8000 | ✅ Correct |
| **healthcheck** | N/A | `/health` endpoint | ✅ Nouveau |
| **Container DB** | N/A | `annuaire-postgres` | ✅ Nouveau |
| **main.py** | Monolithique | Modularisé (routes séparées) | ❌ **CHANGÉ** |
| **Swagger** | Tags absents | Tags Swagger UI par route | ✅ Nouveau |
| **Tests** | `tests/` | `.github/tests/` | ❌ **CHANGÉ** |

**Docker-compose:** Le `docker-compose.yml` du backend inclut maintenant PostgreSQL avec:
- Healthcheck (`pg_isready`)
- Volume `postgres_data` pour persistence
- Dépendance: backend attend postgres healthy

**Note:** Projet a migré MariaDB → PostgreSQL (précédemment SQLite → MariaDB). Voir MASTER-BACKLOG.md pour l'historique de cette migration.

---

### MCP (mcp-fast-mcp)

| Aspect | Documentation | Réalité | Statut |
|--------|---|---|---|
| **Port MCP** | 8001 | 8000 (interne seulement) | ❌ **CHANGÉ** |
| **Accès externe** | Direct (8001) | Via Nginx (80/443) | ❌ **CHANGÉ** |
| **Nginx** | Optionnel | Intégré au compose | ✅ Nouveau |
| **Certificats TLS** | N/A | `docker/certs/` (mkcert) | ✅ Nouveau |
| **Config Nginx** | N/A | `docker/nginx/` (conf.d) | ✅ Nouveau |
| **Network** | N/A | `mcp-network` (bridge) | ✅ Nouveau |
| **Backend URL** | `http://localhost:8000` | `http://localhost:8000` | ✅ Correct |

**Docker-compose:** Le `docker-compose.yml` du MCP inclut:
- Service `annuaire-mcp` (port 8000 interne)
- Service `nginx` (ports 80/443 externes)
- Réseau `mcp-network` (bridge)
- Dépendance: nginx dépend de annuaire-mcp

---

### Structure Globale

```
annuaire-contacts/
├── orchestrator/              ✅ Documentation + règles
├── annuaire-fastapi/           ✅ Service FastAPI
│   ├── docker-compose.yml     ← CHANGÉ (PostgreSQL inclus)
│   ├── database.py            ← CHANGÉ (PostgreSQL/psycopg2)
│   ├── main.py                ← CHANGÉ (modularisé, routes en fichiers séparés)
│   ├── models.py
│   ├── .github/tests/         ← CHANGÉ (déplacé depuis tests/)
│   └── Dockerfile
│
├── mcp-fast-mcp/              ✅ Service MCP
│   ├── docker-compose.yml     ← CHANGÉ (Nginx inclus)
│   ├── docker/                ← NOUVEAU
│   │   ├── certs/             ← Certificats TLS (mkcert)
│   │   └── nginx/
│   │       ├── nginx.conf     ← Config Nginx
│   │       └── conf.d/
│   │           └── default.conf ← VHost Nginx
│   ├── backend_client.py      ✅ Client httpx (règle 3)
│   ├── main.py
│   ├── models.py
│   ├── scripts/               ← VIDE (pas utilisé)
│   └── Dockerfile
│
└── data/                      ← NOUVEAU (dossier partagé)
```

---

## 🔄 Changements Par Service

### Backend FastAPI

**Fichiers modifiés:**
- `docker-compose.yml` — Service PostgreSQL (remplace MariaDB)
- `database.py` — `postgresql+psycopg2://` (SQLAlchemy)
- `requirements.txt` — `psycopg2-binary` (plus de `pymysql`)
- `main.py` — Modularisé en routes séparées
- Tests déplacés vers `.github/tests/`

**Ports:**
- DB: `5432:5432` (PostgreSQL, mapping direct)
- API: `8000:8000` (identique)

**Variables d'environnement critiques:**
```env
DATABASE_URL=postgresql+psycopg2://annuaire_user:annuaire_password@postgres:5432/annuaire_contacts
LOG_LEVEL=INFO
```

---

### MCP (mcp-fast-mcp)

**Fichiers modifiés:**
- `docker-compose.yml` — Ajout Nginx reverse proxy
- `docker/nginx/nginx.conf` — Configuration Nginx globale
- `docker/nginx/conf.d/default.conf` — VHost pour MCP
- `docker/certs/` — Certificats autosignés (mkcert)

**Ports:**
- ✗ Application: `8000` (interne seulement, via Nginx)
- ✗ Nginx HTTP: `80:80`
- ✗ Nginx HTTPS: `443:443`
- ❌ Plus de port `8001` direct!

**Variables d'environnement:**
```env
BACKEND_URL=http://localhost:8000
LOG_LEVEL=INFO
```

---

## 🚨 Implications pour Orchestration

### Avant (Autonome Simple)

```bash
# Terminal 1
cd annuaire-fastapi && docker compose up

# Terminal 2
cd mcp-fast-mcp && docker compose up

# Appels
curl http://localhost:8000/health      # Backend
curl http://localhost:8001/health      # MCP
```

### Après (Nginx + PostgreSQL)

```bash
# Terminal 1
cd annuaire-fastapi && docker compose up
# → Démarre postgres + backend

# Terminal 2
cd mcp-fast-mcp && docker compose up
# → Démarre annuaire-mcp + nginx

# Appels
curl http://localhost:8000/health      # Backend OK
curl http://localhost/health           # MCP via Nginx (port 80)
curl https://localhost/health          # MCP via Nginx TLS (port 443)
```

---

## 📋 Checklist Migration Documentation

- [ ] **SERVICES.md** — Mettre à jour ports et config MCP/Nginx/PostgreSQL
- [ ] **ARCHITECTURE.md** — Mettre à jour schéma (PostgreSQL, Nginx)
- [ ] **API.md** — Vérifier endpoints (tags Swagger ajoutés, endpoints RAP-1.4/1.5)
- [ ] **MCP.md** — Mettre à jour ports d'accès
- [ ] **README.md** — Mettre à jour quick start
- [x] **CLAUDE.md** — Mis à jour (PostgreSQL au lieu de MariaDB)
- [ ] **QUICKSTART.md** — Mettre à jour exemples curl

---

## 📝 Notes

### À Investiguer

1. **data/ folder** — Quel est son usage exact?
   - Volume partagé entre services?
   - Données persistantes?

2. **scripts/ folder** — Pourquoi vide dans mcp-fast-mcp?
   - Prévu pour des scripts d'installation?
   - À documenter ou supprimer?

3. **Backend ↔ MCP Network** — Communication intra-container?
   - Backend local sur `8000` ou sur réseau `mcp-network`?
   - Si séparé par network, BACKEND_URL doit être `http://backend:8000`

### Contraintes Nginx

- ✅ Ports 80/443 = ports standards (production-ready)
- ✅ TLS avec mkcert = développement local
- ⚠️ En production: certificats Let's Encrypt ou CA interne
- ⚠️ Vérifier la config CORS si accès depuis navigateur

---

**Génération:** 2026-04-07
**Mise à jour:** 2026-07-02 (migration PostgreSQL confirmée, modularisation main.py, réorg tests)
**Scope:** annuaire-contacts/orchestrator
