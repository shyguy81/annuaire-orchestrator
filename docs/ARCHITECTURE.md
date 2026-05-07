# Architecture — Annuaire Contacts

## Vue d'ensemble

```
Client HTTP (8000)          Client IA (HTTPS 443)
  ↓                               ↓
Backend FastAPI                Nginx Reverse Proxy
  ├─ CRUD contacts           ↓
  └─ MariaDB (3307)          MCP Server
       ↓                      (port 8000 interne)
  (via httpx)                    ↓
     ↑←←←←←←←←←←←←←←←←←←(Backend URL: 8000)
```

## Services (Autonomes)

### Backend (FastAPI)
- **Dossier:** `backend-fastapi/`
- **Port:** 8000 (API)
- **Port DB:** 3307 (MariaDB, local mapping 3307→3306 interne)
- **Docker:** `backend-fastapi/docker-compose.yml` (inclut MariaDB)
- **Modèles Pydantic** pour Contact
- **Endpoints CRUD** + search
- **Base de données:** MariaDB 11.2 (container: `annuaire-mariadb`)

### MCP (Serveur IA)
- **Dossier:** `mcp-fast-mcp/`
- **Port Application:** 8000 (interne seulement)
- **Ports Nginx:** 80 (HTTP), 443 (HTTPS) — accès public via Nginx reverse proxy
- **Docker:** `mcp-fast-mcp/docker-compose.yml` (inclut Nginx + certificats TLS)
- **Interface IA:** /search, /summary, /suggestions (via Nginx)
- **TLS:** Certificats autosignés dans `docker/certs/` (mkcert)
- **Config Nginx:** `docker/nginx/nginx.conf` + `docker/nginx/conf.d/default.conf`
- **Appels backend:** Via httpx async client (backend_client.py)
- **Réponses:** Structurées pour agents

## Flux Typique

```
1. Requête Utilisateur
   ↓
2. Backend CRUD: POST /contacts
   ↓
3. Persistance: SQLite
   ↓
4. Réponse (Contact)
```

```
1. Requête IA
   ↓
2. MCP: POST /search
   ↓
3. → httpx → Backend GET /contacts/search/{query}
   ↓
4. ← Résultat structuré
```

## Variables d'environnement

### Backend

| Variable | Default | Description |
|----------|---------|-------------|
| `DATABASE_URL` | `mysql+pymysql://annuaire_user:annuaire_password@mariadb:3306/annuaire_contacts` | String connexion MariaDB |
| `LOG_LEVEL` | `INFO` | Niveau de log |

**MariaDB Credentials:**
- User: `annuaire_user`
- Password: `annuaire_password`
- Database: `annuaire_contacts`
- Host: `mariadb` (intra-compose)
- Port: `3306` (interne), `3307` (mapping local)

### MCP

| Variable | Default | Description |
|----------|---------|-------------|
| `BACKEND_URL` | `http://localhost:8000` | URL du backend (appels internes) |
| `LOG_LEVEL` | `INFO` | Niveau de log |

**Nginx:**
- HTTP: port 80
- HTTPS: port 443 (TLS via certificats `docker/certs/`)
- VHost: `annuaire-mcp.local.docker.dev` (dans config Nginx)

## Autonomie des Services

Chaque service:
- ✅ A son propre `docker-compose.yml`
- ✅ Peut démarrer indépendamment
- ✅ Expose ses propres ports
- ✅ Gère sa propre config

**Orchestration:** Manuelle ou via script (voir `docs/SERVICES.md`)

## Repos & Fichiers

```
annuaire-contacts/
├── orchestrator/
│   ├── docs/                    ← Documentation (ce fichier)
│   ├── CLAUDE.md                ← Règles & rôle orchestrator
│   ├── DIAGNOSTIC.md            ← Changements architecture
│   ├── README.md                ← Quick start
│   └── .claude/rules/           ← 5 règles non-négociables
│
├── backend-fastapi/
│   ├── docker-compose.yml       ← Service (MariaDB inclus)
│   ├── Dockerfile
│   ├── main.py                  ← Endpoints FastAPI
│   ├── models.py                ← Schéma Pydantic
│   ├── database.py              ← Connexion MariaDB
│   ├── requirements.txt
│   ├── .env.example
│   └── .git/                    ← Repo autonome
│
├── mcp-fast-mcp/
│   ├── docker-compose.yml       ← Service (Nginx inclus)
│   ├── Dockerfile
│   ├── main.py                  ← Endpoints MCP
│   ├── models.py                ← Schémas réponse MCP
│   ├── backend_client.py        ← Client httpx vers backend
│   ├── requirements.txt
│   ├── .env.example
│   ├── docker/                  ← CONFIG NGINX + CERTS
│   │   ├── certs/               ← Certificats TLS (mkcert)
│   │   │   ├── *.pem
│   │   │   ├── *-key.pem
│   │   │   └── .gitkeep
│   │   └── nginx/
│   │       ├── nginx.conf       ← Config Nginx globale
│   │       └── conf.d/
│   │           └── default.conf ← VHost pour annuaire-mcp
│   ├── scripts/                 ← (vide, non utilisé)
│   ├── .github/                 ← Copilot instructions
│   └── .git/                    ← Repo autonome
│
└── data/                        ← Données partagées (persistence volumes)
    └── (volumes MariaDB/cache)
```
