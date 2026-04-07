# Architecture — Annuaire Contacts

## Vue d'ensemble

```
Client (HTTP / MCP)
  ↓
Backend FastAPI (CRUD contacts)
  ↓
SQLite DB
```

## Services

### Backend (FastAPI)
- Modèles Pydantic pour Contact
- Endpoints CRUD
- Base de données SQLite

### MCP (Serveur)
- Interface pour requêtes IA
- Appels au backend
- Réponses structurées pour Claude

## Flux típique

1. **Requête CRUD** → Backend expose `/contacts`
2. **Requête MCP** → Serveur MCP → appelle backend → retourne résultat
3. **Persistance** → SQLite (data/contacts.db)

## Variables d'environnement

| Var | Backend | MCP | Valeur |
|-----|---------|-----|--------|
| `DATABASE_URL` | ✓ |  | `sqlite:///./contacts.db` |
| `BACKEND_URL` |  | ✓ | `http://backend:8000` |
| `MCP_SERVER_URL` | ✓ |  | `http://mcp:8001` |
