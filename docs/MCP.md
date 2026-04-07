# MCP Server — Documentation Complète

**Base URL:** `http://localhost:8001`

**Rôle:** Wrapper FastAPI autour du backend. Fournit des endpoints optimisés pour requêtes IA/agent.

**Design:** Tous les endpoints MCP appelent le backend via httpx async client.

---

## Modèles de Réponse

### ContactInfo (MCP)

```json
{
  "id": "string (UUID)",
  "nom": "string",
  "email": "string",
  "telephone": "string or null",
  "adresse": "string or null",
  "organisation": "string or null",
  "tags": ["string"]
}
```

### SearchResponse

```json
{
  "query": "string",
  "results": [ContactInfo],
  "count": "int"
}
```

### ContactSummary

```json
{
  "id": "string",
  "nom": "string",
  "email": "string",
  "organisation": "string or null",
  "tags": ["string"],
  "summary": "string (texte généré)"
}
```

### SuggestionsResponse

```json
{
  "partial_name": "string",
  "suggestions": ["string"],
  "count": "int"
}
```

---

## Endpoints

### 1. Health Check

```
GET /health
```

**Réponse (200):**
```json
{
  "status": "ok",
  "service": "mcp"
}
```

---

### 2. Chercher des Contacts (MCP)

```
POST /search
Content-Type: application/json
```

**Purpose:** Wrapper autour de `GET /contacts/search/{query}` du backend. Optimisé pour requêtes IA.

**Body:**
```json
{
  "query": "string (required)",
  "limit": "int (default: 10)"
}
```

**Réponse (200):**
```json
{
  "query": "alice",
  "results": [
    {
      "id": "acd52471-6104-48d2-94b0-02c7b51ba678",
      "nom": "Alice Dupont",
      "email": "alice@example.com",
      "telephone": "+33612345678",
      "adresse": "123 Rue de Paris",
      "organisation": "Acme Corp",
      "tags": ["vip", "client"]
    }
  ],
  "count": 1
}
```

**Cas d'usage:**
- Agent IA cherche un contact par nom
- Intégration Claude: "Cherche Alice"
- Autocomplete frontend

**Exemple curl:**
```bash
curl -X POST http://localhost:8001/search \
  -H "Content-Type: application/json" \
  -d '{
    "query": "alice",
    "limit": 10
  }'
```

---

### 3. Résumé d'un Contact

```
POST /summary
Content-Type: application/json
```

**Purpose:** Récupère un contact et génère un résumé textuel pour IA.

**Body:**
```json
{
  "contact_id": "string (required, UUID)"
}
```

**Réponse (200):**
```json
{
  "id": "acd52471-6104-48d2-94b0-02c7b51ba678",
  "nom": "Alice Dupont",
  "email": "alice@example.com",
  "organisation": "Acme Corp",
  "tags": ["vip", "client"],
  "summary": "Alice Dupont (alice@example.com) - Acme Corp - Tel: +33612345678 - Tags: vip, client"
}
```

**Erreurs:**
- `404 Not Found` — Contact n'existe pas

**Cas d'usage:**
- Agent IA lit un résumé structuré
- Génération de rapport
- Context enrichissement

**Exemple curl:**
```bash
curl -X POST http://localhost:8001/summary \
  -H "Content-Type: application/json" \
  -d '{
    "contact_id": "acd52471-6104-48d2-94b0-02c7b51ba678"
  }'
```

---

### 4. Suggestions Autocomplete

```
POST /suggestions
Content-Type: application/json
```

**Purpose:** Suggère des noms basés sur partial match (autocomplete).

**Body:**
```json
{
  "partial_name": "string (required)",
  "limit": "int (default: 5)"
}
```

**Réponse (200):**
```json
{
  "partial_name": "ali",
  "suggestions": [
    "Alice Dupont",
    "Alicia Martin"
  ],
  "count": 2
}
```

**Algorithme:** Recherche sur `nom` ilike `%partial_name%`

**Cas d'usage:**
- Autocomplete frontend
- Agent cherche contact par partial
- Désambiguation multi-résultats

**Exemple curl:**
```bash
curl -X POST http://localhost:8001/suggestions \
  -H "Content-Type: application/json" \
  -d '{
    "partial_name": "ali",
    "limit": 5
  }'
```

---

### 5. Lister tous les Contacts (MCP)

```
GET /contacts?limit=100
```

**Purpose:** Liste complète des contacts (pour agents IA nécessitant contexte).

**Query Parameters:**
| Paramètre | Type | Default | Description |
|-----------|------|---------|-------------|
| `limit` | int | 100 | Nombre maximum |

**Réponse (200):**
```json
{
  "contacts": [
    {
      "id": "acd52471-6104-48d2-94b0-02c7b51ba678",
      "nom": "Alice Dupont",
      ...
    }
  ],
  "count": 1
}
```

**Cas d'usage:**
- Agent IA chargement contexte global
- Export de données
- Synchronisation

**Exemple curl:**
```bash
curl "http://localhost:8001/contacts?limit=100"
```

---

## Fluxes Typiques (Agent IA)

### Scenario 1: Chercher + Résumé

```
1. POST /search {"query": "Alice"}
   → Récupère list contacts matching

2. POST /summary {"contact_id": "..."}
   → Récupère summary détaillé pour affichage
```

### Scenario 2: Autocomplete + Action

```
1. POST /suggestions {"partial_name": "ali"}
   → Affiche suggestions dropdown

2. User sélectionne → POST /summary
   → Affiche détails contact
```

### Scenario 3: Contexte Global

```
1. GET /contacts {"limit": 100}
   → Agent charge tous les contacts en mémoire

2. Agent peut faire raisonnement sur dataset complet
   → Ex: "Cherche tous les VIPs de Acme Corp"
```

---

## Architecture (Backend → MCP)

```
Client (IA)
  ↓
MCP Server (8001)
  ├─ /search → httpx → Backend GET /contacts/search/{query}
  ├─ /summary → httpx → Backend GET /contacts/{id}
  ├─ /suggestions → httpx → Backend GET /contacts/search/{partial}
  └─ /contacts → httpx → Backend GET /contacts?limit=100
  ↓
Backend (8000)
  ↓
SQLite DB
```

---

## Intégration Claude API (Futur)

```python
from anthropic import Anthropic

client = Anthropic()

# MCP en tant que tool
tools = [
  {
    "name": "search_contacts",
    "description": "Cherche des contacts par nom/email/organisation",
    "input_schema": {
      "type": "object",
      "properties": {
        "query": {"type": "string"}
      }
    }
  },
  {
    "name": "get_contact_summary",
    "description": "Résumé détaillé d'un contact",
    "input_schema": {
      "type": "object",
      "properties": {
        "contact_id": {"type": "string"}
      }
    }
  }
]

# Appel Claude avec tools
response = client.messages.create(
  model="claude-opus-4-6",
  max_tokens=1024,
  tools=tools,
  messages=[
    {"role": "user", "content": "Qui est Alice et ses infos?"}
  ]
)
```

---

## Debugging

**Logs MCP:**
```bash
docker compose logs -f mcp
```

**Test endpoint:**
```bash
curl -v http://localhost:8001/health
```

**Vérifier backend connectivity:**
```bash
# Depuis container MCP
docker compose exec mcp curl http://backend:8000/health
```
