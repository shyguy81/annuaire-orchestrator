# Diagnostic — Changements d'Architecture (2026-04-07)

## Résumé des Ajustements

La structure du projet a évolué depuis la dernière documentation. Voici les changements identifiés.

---

## 📁 Structure Réelle vs Documentée

### Backend FastAPI

| Aspect | Documentation | Réalité | Statut |
|--------|---|---|---|
| **Base de données** | SQLite (`./data/contacts.db`) | MariaDB (port 3307→3306 interne) | ❌ **CHANGÉ** |
| **Variables env** | `DATABASE_URL=sqlite:///...` | `DATABASE_URL=mysql+pymysql://...` | ❌ **CHANGÉ** |
| **Port API** | 8000 | 8000 | ✅ Correct |
| **healthcheck** | N/A | `/health` endpoint | ✅ Nouveau |
| **Container DB** | N/A | `annuaire-mariadb` | ✅ Nouveau |

**Docker-compose:** Le `docker-compose.yml` du backend inclut maintenant MariaDB avec:
- Healthcheck
- Volume `mariadb_data` pour persistence
- Dépendance: backend attend mariadb healthy

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
├── backend-fastapi/           ✅ Service FastAPI
│   ├── docker-compose.yml     ← CHANGÉ (MariaDB inclus)
│   ├── database.py            ← CHANGÉ (MySQL/MariaDB)
│   ├── main.py
│   ├── models.py
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
- `docker-compose.yml` — Ajout service MariaDB
- `database.py` — Migration SQLite → MariaDB/MySQL
- `.env.example` — Nouvelles variables (DB credentials)

**Ports:**
- ✗ Portage: `3307:3306` (mapping local 3307 → container 3306)
- API: `8000:8000` (identique)

**Variables d'environnement critiques:**
```env
DATABASE_URL=mysql+pymysql://annuaire_user:annuaire_password@mariadb:3306/annuaire_contacts
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
cd backend-fastapi && docker compose up

# Terminal 2
cd mcp-fast-mcp && docker compose up

# Appels
curl http://localhost:8000/health      # Backend
curl http://localhost:8001/health      # MCP
```

### Après (Nginx + MariaDB)

```bash
# Terminal 1
cd backend-fastapi && docker compose up
# → Démarre mariadb + backend

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

- [ ] **SERVICES.md** — Mettre à jour ports et config MCP/Nginx
- [ ] **ARCHITECTURE.md** — Mettre à jour schéma (MariaDB, Nginx)
- [ ] **API.md** — Vérifier endpoints (inchangés)
- [ ] **MCP.md** — Mettre à jour ports d'accès
- [ ] **README.md** — Mettre à jour quick start
- [ ] **CLAUDE.md** — Mettre à jour diagramme architecture
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
**Scope:** annuaire-contacts/orchestrator
