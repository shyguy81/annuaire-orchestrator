# Changelog — annuaire-contacts/orchestrator

## [2026-04-07] — Architecture Update: MariaDB + Nginx

### Changed

#### Backend FastAPI
- ✨ **Database:** SQLite → **MariaDB 11.2**
  - Container `annuaire-mariadb` démarré automatiquement
  - Port mapping: `3307:3306` (local:interne)
  - Healthcheck intégré au compose
  - Credentials: `annuaire_user` / `annuaire_password`
  - Database: `annuaire_contacts`
- 🔧 **Environment Variables:** 
  - `DATABASE_URL=mysql+pymysql://annuaire_user:annuaire_password@mariadb:3306/annuaire_contacts`
  - Volume: `mariadb_data` pour persistence
- 📊 **docker-compose.yml:** Inclut service MariaDB avec dépendance + healthcheck

#### MCP (mcp-fast-mcp)
- ✨ **Nginx Reverse Proxy:** Nouveau service intégré au compose
  - Ports publics: `80` (HTTP) + `443` (HTTPS)
  - MCP app interne sur port `8000` seulement
  - Hostname: `annuaire-mcp.local.docker.dev`
- 🔒 **TLS Certificates:** Certificats autosignés (mkcert)
  - Stockés dans `docker/certs/`
  - À remplacer en production (Let's Encrypt/CA interne)
- ⚙️ **Nginx Configuration:**
  - `docker/nginx/nginx.conf` — Config globale
  - `docker/nginx/conf.d/default.conf` — VHost pour annuaire-mcp
- 🌐 **Network:** `mcp-network` (bridge) pour communication intra-compose
- 📊 **docker-compose.yml:** Services `annuaire-mcp` + `nginx` + réseau

### Updated Documentation

- 📄 **ARCHITECTURE.md** — Schéma, services, variables env, structure fichiers
- 📄 **SERVICES.md** — Ports, endpoints, config Nginx, checklist démarrage
- 📄 **CLAUDE.md** — Diagramme architecture mis à jour
- 📄 **DIAGNOSTIC.md** — NOUVEAU: Vue détaillée des changements

### Structure Changes

```
mcp-fast-mcp/
├── docker/               ← NOUVEAU
│   ├── certs/           ← Certificats TLS (mkcert)
│   └── nginx/           ← Config Nginx
│       ├── nginx.conf
│       └── conf.d/default.conf
└── scripts/             ← Vide (non utilisé actuellement)
```

### Migration Guide

#### For Backend Users
```bash
# Avant: SQLite local
DATABASE_URL=sqlite:///./data/contacts.db

# Après: MariaDB (automatique dans docker-compose)
DATABASE_URL=mysql+pymysql://annuaire_user:annuaire_password@mariadb:3306/annuaire_contacts
```

#### For MCP Access
```bash
# Avant: Port 8001 direct
curl http://localhost:8001/health

# Après: Port 80 (HTTP) ou 443 (HTTPS) via Nginx
curl http://localhost/health
curl -k https://localhost/health      # -k ignore certificat auto-signé
```

### Testing Checklist

- [ ] Backend démarre avec MariaDB
- [ ] MCP démarre avec Nginx
- [ ] Backend API répondre sur port 8000
- [ ] MCP répondre sur port 80 (HTTP) et 443 (HTTPS)
- [ ] MCP peut appeler backend (BACKEND_URL=http://localhost:8000)
- [ ] Nginx reverse proxy fonctionne
- [ ] TLS fonctionne (même avec certificat auto-signé)

### Known Issues / TODO

- [ ] `data/` folder — Usage unclear (données partagées? Quoi persister?)
- [ ] `mcp-fast-mcp/scripts/` — Vide, but presente → documenter ou supprimer?
- [ ] Network isolation — Backend & MCP sur réseaux différents? À clarifier si comm intra-compose
- [ ] Production TLS — Remplacer mkcert par Let's Encrypt ou CA interne

### Breaking Changes

- ❌ Port MCP: `8001` → `80/443` (via Nginx)
- ❌ Database backend: SQLite → MariaDB
- ❌ Access patterns: Direct localhost:8001 → localhost/ (via Nginx)

---

## [2026-04-06] — Initial Setup

- ✅ Backend FastAPI (SQLite)
- ✅ MCP Server (port 8001)
- ✅ Autonomous services (separate docker-compose.yml)
- ✅ Documentation (SERVICES.md, ARCHITECTURE.md, API.md, MCP.md)
