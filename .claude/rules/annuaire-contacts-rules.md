# Rules — annuaire-contacts/orchestrator

**5 règles essentielles pour ce projet multi-repos.**

Architecture:

```
annuaire-contacts/
├── orchestrator/          ← CE DOSSIER (orchestration + doc)
├── annuaire-fastapi/      ← Backend (logique métier)
├── annuaire-cli/          ← CLI nouveau
├── mcp-fast-mcp/          ← MCP actuel (remplacé par annuaire-mcp)
└── cli-llm/               ← Espace de test CLI
```

---

## 🚨 Règles Non-Négociables

### Règle 1: **Orchestrator = orchestration uniquement**

**Énoncé:**  
L'orchestrateur découple les services et route les appels. Il ne contient pas de logique métier.

**Portée:**

- ✅ Docker Compose, gestion des services
- ✅ Coordination des flux (FastAPI → MCP)
- ✅ Config centralisée
- ❌ Logique métier (va dans annuaire-fastapi)
- ❌ Implémentation MCP (va dans mcp-fast-mcp)

**Conséquence:**

- ❌ Logique métier ici = duplication, drift
- ❌ Impossible de réutiliser les services indépendamment
- ❌ Tests fragmentés

**Comment Faire:**

```python
# ❌ MAUVAIS (logique métier dans orchestrator)
@app.post("/contacts")
def create_contact(name: str):
    # Validation, processing, DB...
    contact = process_and_validate(name)
    return contact

# ✅ BON (orchestrator route, backend implémente)
@app.post("/contacts")
async def create_contact(name: str):
    # Route vers FastAPI backend
    response = await backend_client.post("/contacts", json={"name": name})
    return response
```

---

### Règle 2: **Ne JAMAIS modifier le code des repos liés**

**Énoncé:**  
`annuaire-fastapi/`, `annuaire-cli/`, `mcp-fast-mcp/` et `cli-llm/` ne se modifient **jamais** depuis ce dossier.

**Détail:** Voir **[protected-repos.md](./protected-repos.md)** — liste exhaustive des repos protégés, portée exacte, cas particulier des fichiers générés (Règle 3).

**Pourquoi:**

- Repos technique = source de truth
- Orchestrator = documentation + coordination
- Sinon: git history perdue, CI/CD échoue

**Conséquence:**

- ❌ Modifs depuis ici = pas committées dans leur repo
- ❌ Services restent en version ancienne
- ❌ Impossible d'auditer les changements

**Comment Faire:**

```bash
# ❌ MAUVAIS
# cd ../annuaire-fastapi
# → éditer src/main.py
# → commit depuis orchestrator

# ✅ BON
# 1. cd ../annuaire-fastapi   (ou annuaire-cli, mcp-fast-mcp, cli-llm)
# 2. Modifier le code ICI
# 3. git commit & push DEPUIS ce repo
# 4. Revenir à orchestrator pour documenter

# Ou si révisions ponctuelles:
# 1. Créer un prompt en local: .github/prompts/edit--inbox--fix-xxx.md
# 2. cd ../annuaire-fastapi
# 3. @edit--inbox--fix-xxx
# 4. Renommer en edit--done--
# 5. Commit depuis annuaire-fastapi
```

---

### Règle 3: **Les clients générés sont la source de vérité**

**Énoncé:**  
Pour appeler le backend FastAPI ou le serveur MCP, utiliser **toujours** les clients générés (pydantic, httpx), jamais d'URLs hardcodées.

**Pourquoi:**

- Contrats API = générés depuis OpenAPI specs
- Évite drift orchestrator ↔ services réels
- Centralisé = facile à adapter dev/staging/prod

**Conséquence:**

- ❌ URLs hardcodées = contrats cassés après changement API
- ❌ Types non vérifiés = runtime errors
- ❌ Déploiement fragile

**Comment Faire:**

```python
# ❌ MAUVAIS
import httpx
response = await httpx.post("http://backend:8000/contacts", json={"name": "Jean"})

# ✅ BON
from backend.client import ContactsApi
api = ContactsApi(base_url="http://backend:8000")
response = await api.create_contact(name="Jean")

# Avec génération OpenAPI:
# 1. annuaire-fastapi expose son OpenAPI à /openapi.json
# 2. cd orchestrator && python scripts/generate-client.py
# 3. Importer depuis client généré
```

---

### Règle 4: **Commiter DEPUIS les repos techniques, pas depuis orchestrator**

**Énoncé:**  
Les `git commit` & `git push` se font **depuis le repo concerné** (`annuaire-fastapi/`, `annuaire-cli/`, `mcp-fast-mcp/` ou `cli-llm/`), jamais depuis orchestrator.

**Pourquoi:**

- Git history = dans les repos techniques
- Orchestrator n'a pas `.git` des autres repos
- Évite la confusion entre plusieurs repos

**Conséquence:**

- ❌ Commit depuis orchestrator = git error
- ❌ Changes lost, history incohérente
- ❌ CI/CD échoue

**Comment Faire:**

```bash
# ❌ MAUVAIS
cd orchestrator
# modifier ../annuaire-fastapi/src/main.py
git commit -m "fix: ..."
git push

# ✅ BON
cd ../annuaire-fastapi
# modifier src/main.py
git add src/main.py
git commit -m "fix: ..."
git push
```

---

### Règle 5: **Docker Compose comme interface de coordination**

**Énoncé:**  
L'orchestrateur expose un `docker-compose.yml` qui démarre tous les services. Configuration centralisée.

**Portée:**

- ✅ Services Docker (backend, MCP, orchestrator)
- ✅ Variables env partagées
- ✅ Volumes + réseaux
- ❌ Logique métier
- ❌ Config spécifique à un service (va dans le repo du service)

**Conséquence:**

- ❌ Config fragmentée = difficult à reproduire
- ❌ Services démarrés manuellement = erreurs
- ❌ Impossible onboard un nouveau dev rapidement

**Comment Faire:**

```yaml
# ✅ docker-compose.yml
version: "3.9"

services:
  backend:
    build:
      context: ../annuaire-fastapi
      dockerfile: Dockerfile
    environment:
      - MCP_SERVER_URL=http://mcp:8001
      - LOG_LEVEL=INFO
    ports:
      - "8000:8000"
    depends_on:
      - mcp

  mcp:
    build:
      context: ../mcp-fast-mcp
      dockerfile: Dockerfile
    ports:
      - "8001:8001"

  orchestrator:
    build: .
    environment:
      - BACKEND_URL=http://backend:8000
      - MCP_URL=http://mcp:8001
    ports:
      - "8080:8080"
    depends_on:
      - backend
      - mcp
```

---

## 📋 Checklist d'Application

Avant de commencer à travailler, vérifier:

- [ ] **Règle 1:** Je n'ajoute pas de logique métier dans orchestrator
- [ ] **Règle 2:** Je n'édite pas les repos liés depuis ce dossier ([liste des repos protégés](./protected-repos.md))
- [ ] **Règle 3:** J'utilise les clients générés (pas d'URLs hardcodées)
- [ ] **Règle 4:** Je fais `git commit` depuis le bon repo (voir [protected-repos.md](./protected-repos.md))
- [ ] **Règle 5:** Les configs centrales sont dans `docker-compose.yml`

---

## 🆘 Si Tu Oublies Une Règle

| Erreur                             | Symptôme                        | Solution                            |
| ---------------------------------- | ------------------------------- | ----------------------------------- |
| Logique métier dans orchestrator   | Code dupliqué, tests fragmentés | Déplacer vers annuaire-fastapi       |
| Modifier repos liés depuis orchestrator | git history perdue          | cd vers le repo concerné + commit depuis là (voir [protected-repos.md](./protected-repos.md)) |
| URLs hardcodées                    | Runtime errors, drift API       | Utiliser client généré              |
| Commit depuis orchestrator         | Git push fail, confusion        | cd vers le repo concerné + commit/push |
| Config fragmentée                  | Services ne démarrent pas       | Centraliser dans docker-compose.yml |

---

## 🔗 Liens

- **[CLAUDE.md](../../CLAUDE.md)** — Contexte global du projet
- **[docs/RULES.md](../../docs/RULES.md)** — Rules du projet (copie locale)
- **[docker-compose.yml](../../docker-compose.yml)** — Orchestration des services
- **[protected-repos.md](./protected-repos.md)** — Liste des repos protégés (Règle 2)
- **../../annuaire-fastapi** — Backend FastAPI (logique métier)
- **../../annuaire-cli** — CLI Rust
- **../../mcp-fast-mcp** — Serveur MCP (protocole IA)
- **../../cli-llm** — Espace de test annuaire-cli

---

**Dernière mise à jour:** 2026-07-02  
**Adapté pour:** annuaire-contacts/orchestrator
