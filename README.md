# Annuaire de Contacts — Orchestrateur

Système simple de gestion contacts avec API FastAPI + serveur MCP.

## Démarrage rapide

```bash
# Depuis la racine (annuaire-contacts/)
docker-compose up --build

# Backend: http://localhost:8000
# MCP: http://localhost:8001
# Docs API: http://localhost:8000/docs
```

## Architecture

```
orchestrator/          ← Ce dossier (config + doc)
backend-fastapi/       ← API REST (gestion contacts)
mcp-fast-mcp/          ← Serveur MCP (accès IA)
docker-compose.yml     ← Orchestration des services
```

## Schéma Contact

```json
{
  "id": "uuid",
  "nom": "string",
  "email": "string",
  "telephone": "string",
  "adresse": "string",
  "organisation": "string",
  "tags": ["string"]
}
```

## API Endpoints (Backend)

- `GET /contacts` — Lister tous les contacts
- `GET /contacts/{id}` — Détails d'un contact
- `POST /contacts` — Créer un contact
- `PUT /contacts/{id}` — Mettre à jour
- `DELETE /contacts/{id}` — Supprimer

## MCP (À venir)

Serveur pour requêtes IA: recherche, synthèse, suggestions.
