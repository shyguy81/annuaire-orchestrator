# API Backend — Documentation Complète

**Base URL:** `http://localhost:8000`

**Authentification:** Aucune (MVP). À ajouter en production.

---

## Modèles

### Contact

```json
{
  "id": "string (UUID)",
  "nom": "string (required, indexed)",
  "email": "string (required, unique, indexed)",
  "telephone": "string (optional)",
  "adresse": "string (optional)",
  "organisation": "string (optional, indexed)",
  "tags": ["string"] (optional, default: []),
  "created_at": "ISO 8601 datetime",
  "updated_at": "ISO 8601 datetime"
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
  "status": "ok"
}
```

---

### 2. Créer un Contact

```
POST /contacts
Content-Type: application/json
```

**Body (required):**
```json
{
  "nom": "string (required)",
  "email": "string (required, valid email)",
  "telephone": "string (optional)",
  "adresse": "string (optional)",
  "organisation": "string (optional)",
  "tags": ["string"] (optional)
}
```

**Réponse (201 Created):**
```json
{
  "id": "acd52471-6104-48d2-94b0-02c7b51ba678",
  "nom": "Alice Dupont",
  "email": "alice@example.com",
  "telephone": "+33612345678",
  "adresse": "123 Rue de Paris",
  "organisation": "Acme Corp",
  "tags": ["vip", "client"],
  "created_at": "2026-04-07T07:30:37",
  "updated_at": "2026-04-07T07:30:37"
}
```

**Erreurs:**
- `400 Bad Request` — Email déjà existant ou format invalide
- `422 Unprocessable Entity` — Validation échouée

**Exemple curl:**
```bash
curl -X POST http://localhost:8000/contacts \
  -H "Content-Type: application/json" \
  -d '{
    "nom": "Alice Dupont",
    "email": "alice@example.com",
    "telephone": "+33612345678",
    "organisation": "Acme Corp",
    "tags": ["vip"]
  }'
```

---

### 3. Lister les Contacts

```
GET /contacts?skip=0&limit=100
```

**Query Parameters:**
| Paramètre | Type | Default | Description |
|-----------|------|---------|-------------|
| `skip` | int | 0 | Nombre de contacts à sauter (pagination) |
| `limit` | int | 100 | Nombre maximum à retourner |

**Réponse (200):**
```json
[
  {
    "id": "acd52471-6104-48d2-94b0-02c7b51ba678",
    "nom": "Alice Dupont",
    "email": "alice@example.com",
    ...
  }
]
```

**Exemple curl:**
```bash
curl "http://localhost:8000/contacts?skip=0&limit=50"
```

---

### 4. Récupérer un Contact

```
GET /contacts/{contact_id}
```

**Path Parameters:**
| Paramètre | Type | Description |
|-----------|------|-------------|
| `contact_id` | string | UUID du contact |

**Réponse (200):**
```json
{
  "id": "acd52471-6104-48d2-94b0-02c7b51ba678",
  "nom": "Alice Dupont",
  "email": "alice@example.com",
  ...
}
```

**Erreurs:**
- `404 Not Found` — Contact n'existe pas

**Exemple curl:**
```bash
curl "http://localhost:8000/contacts/acd52471-6104-48d2-94b0-02c7b51ba678"
```

---

### 5. Mettre à Jour un Contact

```
PUT /contacts/{contact_id}
Content-Type: application/json
```

**Path Parameters:**
| Paramètre | Type | Description |
|-----------|------|-------------|
| `contact_id` | string | UUID du contact |

**Body (optional — tous les champs optionnels):**
```json
{
  "nom": "string",
  "email": "string",
  "telephone": "string",
  "adresse": "string",
  "organisation": "string",
  "tags": ["string"]
}
```

**Réponse (200):**
```json
{
  "id": "acd52471-6104-48d2-94b0-02c7b51ba678",
  "nom": "Alice Dupont (updated)",
  ...
  "updated_at": "2026-04-07T08:00:00"
}
```

**Erreurs:**
- `404 Not Found` — Contact n'existe pas
- `400 Bad Request` — Email en doublon

**Exemple curl:**
```bash
curl -X PUT "http://localhost:8000/contacts/acd52471-6104-48d2-94b0-02c7b51ba678" \
  -H "Content-Type: application/json" \
  -d '{"organisation": "New Corp"}'
```

---

### 6. Supprimer un Contact

```
DELETE /contacts/{contact_id}
```

**Path Parameters:**
| Paramètre | Type | Description |
|-----------|------|-------------|
| `contact_id` | string | UUID du contact |

**Réponse (204 No Content):**
```
(vide)
```

**Erreurs:**
- `404 Not Found` — Contact n'existe pas

**Exemple curl:**
```bash
curl -X DELETE "http://localhost:8000/contacts/acd52471-6104-48d2-94b0-02c7b51ba678"
```

---

### 7. Chercher des Contacts

```
GET /contacts/search/{query}
```

**Path Parameters:**
| Paramètre | Type | Description |
|-----------|------|-------------|
| `query` | string | Terme de recherche (nom/email/organisation) |

**Recherche:** Case-insensitive, full-text sur:
- `nom` (ex: "jean" → "Jean Dupont")
- `email` (ex: "alice" → "alice@example.com")
- `organisation` (ex: "acme" → "Acme Corp")

**Réponse (200):**
```json
[
  {
    "id": "acd52471-6104-48d2-94b0-02c7b51ba678",
    "nom": "Alice Dupont",
    ...
  }
]
```

**Exemple curl:**
```bash
curl "http://localhost:8000/contacts/search/alice"
```

---

## Codes HTTP

| Code | Signification |
|------|---------------|
| 200 | Succès (GET, PUT) |
| 201 | Créé (POST) |
| 204 | Succès sans contenu (DELETE) |
| 400 | Requête invalide (email doublon, format) |
| 404 | Ressource non trouvée |
| 422 | Validation échouée |

---

## Documentation Interactive

**Swagger UI:** Accès à `http://localhost:8000/docs`

**ReDoc:** Accès à `http://localhost:8000/redoc`

(Généré automatiquement par FastAPI depuis le code)
