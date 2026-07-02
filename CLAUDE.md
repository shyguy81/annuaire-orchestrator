# Orchestrateur — Annuaire de Contacts

Dossier de **documentation + règles** (pas de code ni orchestration Docker).

## Architecture du projet

```
annuaire-contacts/
├── orchestrator/              ← ce dossier (DOC + RÈGLES)
│   ├── docs/                  → Documentation complète
│   ├── CLAUDE.md              → Ce fichier (règles projet)
│   ├── DIAGNOSTIC.md          → Changements récents
│   └── .claude/rules/         → 5 règles non-négociables
│
├── annuaire-fastapi/          ← Service backend (port 8000 API + MariaDB 3306)
│   ├── docker-compose.yml     → Démarrage: cd annuaire-fastapi && docker compose up
│   │                             (inclut MariaDB + healthcheck)
│   ├── main.py, models.py, database.py
│   ├── Dockerfile
│   └── .git/                  → Repo autonome (remplace mcp-fast-mcp avec annuaire-mcp)
│
├── annuaire-cli/              ← CLI nouveau (interfaces annuaire-fastapi)
│   ├── main.py, commands/
│   ├── Dockerfile
│   └── .git/                  → Repo autonome
│
├── mcp-fast-mcp/              ← Service MCP actuel (port 80/443 via Nginx)
│   ├── docker-compose.yml     → Démarrage: cd mcp-fast-mcp && docker compose up
│   ├── main.py, models.py
│   ├── backend_client.py      → Client httpx vers annuaire-fastapi
│   ├── docker/                → CONFIG NGINX + CERTIFICATS TLS
│   │   ├── certs/
│   │   └── nginx/
│   ├── Dockerfile
│   └── .git/                  → Repo autonome
│   ⚠️  Sera remplacé par annuaire-mcp
│
├── cli-llm/                   ← Espace de test annuaire-cli (Claude Code)
│   └── .git/                  → Repo autonome
│
└── data/                      ← Volumes Docker (à clarifier)
```

## Rôle de l'orchestrateur

✅ **Documentation** (guides, exemples, API specs)  
✅ **Règles** (5 contraintes non-négociables)  
✅ **Coordination** logique (pas de docker-compose à la racine)  

❌ **PAS** d'orchestration Docker  
❌ **PAS** de logique métier  
❌ **PAS** de modification des services techniques

## 🚨 Règles Essentielles (5)

Voir [.claude/rules/annuaire-contacts-rules.md](./.claude/rules/annuaire-contacts-rules.md)

**TL;DR:**
1. Orchestrator = orchestration + doc uniquement
2. Ne JAMAIS modifier annuaire-fastapi, annuaire-cli, mcp-fast-mcp depuis ici
3. Clients générés (pas d'URLs hardcodées)
4. Commits depuis repos techniques
5. Docker Compose = source de vérité

## Commandes

Aucun code/build/lint/test dans ce repo (doc uniquement). Démarrage des services se fait depuis leurs repos respectifs :

```bash
# Backend (API + MariaDB)
cd ../annuaire-fastapi && docker compose up --build
# → http://localhost:8000/docs (Swagger), http://localhost:8000/health

# MCP (+ Nginx, reverse proxy 80/443) — après que le backend soit prêt
cd ../mcp-fast-mcp && docker compose up --build
# → http://localhost/health, https://localhost/health (certif auto-signé)
```

Tests/lint du code métier : voir `annuaire-fastapi` (`pytest -v`) et `mcp-fast-mcp` — jamais lancés depuis ici.

## Drift doc vs réalité

`DIAGNOSTIC.md` liste les écarts connus entre docs et code réel (ex : backend documenté SQLite mais réellement MariaDB, MCP documenté port 8001 mais réellement derrière Nginx 80/443). Vérifier ce fichier avant de faire confiance à un détail de config dans `docs/`.

## Autres fichiers utiles

- **[MASTER-BACKLOG.md](./MASTER-BACKLOG.md)** — Source de vérité unique des tâches (méthodologie ICE, priorités P0/P1/P2)
- **[k8s/](./k8s/)** — Manifests Kubernetes (déploiement au-delà de Docker Compose)

## 📚 Documentation

- **[docs/SERVICES.md](./docs/SERVICES.md)** — Démarrage autonome des services
- **[docs/API.md](./docs/API.md)** — Backend endpoints
- **[docs/MCP.md](./docs/MCP.md)** — MCP endpoints pour IA
- **[docs/ARCHITECTURE.md](./docs/ARCHITECTURE.md)** — Design système
- **[README.md](./README.md)** — Quick start
