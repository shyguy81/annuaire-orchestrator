# Quickstart — Annuaire Contacts

## Démarrage

```bash
cd ..  # Aller à la racine (annuaire-contacts/)
docker-compose up --build
```

**Services prêts:**
- Backend: http://localhost:8000
- MCP: http://localhost:8001
- Docs API: http://localhost:8000/docs

## Tests rapides

### 1. Créer un contact (Backend)

```bash
curl -X POST http://localhost:8000/contacts \
  -H "Content-Type: application/json" \
  -d '{
    "nom": "Jean Dupont",
    "email": "jean@example.com",
    "telephone": "+33612345678",
    "organisation": "Acme Corp",
    "tags": ["vip", "client"]
  }'
```

### 2. Lister les contacts (Backend)

```bash
curl http://localhost:8000/contacts
```

### 3. Chercher via MCP

```bash
curl -X POST http://localhost:8001/search \
  -H "Content-Type: application/json" \
  -d '{
    "query": "Jean",
    "limit": 10
  }'
```

### 4. Résumé d'un contact (MCP)

```bash
# Récupérer l'ID du contact créé, puis:
curl -X POST http://localhost:8001/summary \
  -H "Content-Type: application/json" \
  -d '{
    "contact_id": "<ID_CONTACT>"
  }'
```

### 5. Suggestions (MCP)

```bash
curl -X POST http://localhost:8001/suggestions \
  -H "Content-Type: application/json" \
  -d '{
    "partial_name": "Jea",
    "limit": 5
  }'
```

## Docs

- **Backend**: http://localhost:8000/docs (Swagger UI)
- **Architecture**: `docs/ARCHITECTURE.md`
- **README**: `README.md`
