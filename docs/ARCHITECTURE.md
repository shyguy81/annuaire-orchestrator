# Architecture — Annuaire Contacts

## Vue d'ensemble

```
Client (HTTP)
  ↓
Backend FastAPI (8000)
  ├─ CRUD contacts
  └─ SQLite DB
       ↑
       │ (via httpx)
   MCP Server (8001)
      ↑
      │
Client IA / Agent
```

## Services (Autonomes)

### Backend (FastAPI)
- **Dossier:** `backend-fastapi/`
- **Port:** 8000
- **Docker:** `backend-fastapi/docker-compose.yml`
- **Modèles Pydantic** pour Contact
- **Endpoints CRUD** + search
- **Base de données:** SQLite

### MCP (Serveur IA)
- **Dossier:** `mcp-fast-mcp/`
- **Port:** 8001
- **Docker:** `mcp-fast-mcp/docker-compose.yml`
- **Interface IA:** /search, /summary, /suggestions
- **Appels backend:** Via httpx async client
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
| `DATABASE_URL` | `sqlite:///./data/contacts.db` | String connexion DB |
| `LOG_LEVEL` | `INFO` | Niveau de log |

### MCP

| Variable | Default | Description |
|----------|---------|-------------|
| `BACKEND_URL` | `http://localhost:8000` | URL du backend (appels internes) |
| `LOG_LEVEL` | `INFO` | Niveau de log |

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
│   ├── docs/              ← Documentation (ce fichier)
│   ├── CLAUDE.md          ← Règles
│   └── README.md
├── backend-fastapi/
│   ├── docker-compose.yml ← Service backend autonome
│   ├── main.py
│   ├── models.py
│   ├── database.py
│   └── requirements.txt
└── mcp-fast-mcp/
    ├── docker-compose.yml ← Service MCP autonome
    ├── main.py
    ├── models.py
    ├── backend_client.py
    └── requirements.txt
```
