# Documentation Index

Bienvenue dans l'annuaire-contacts! Voici comment naviguer la documentation.

---

## 📖 Démarrage Rapide

**Je suis nouveau et je veux démarrer:**
1. Lire [`README.md`](../README.md) (3 min)
2. Lancer `docker compose up` (2 min)
3. Essayer les exemples curl dans [`QUICKSTART.md`](../QUICKSTART.md) (5 min)

**Total:** ~10 min pour avoir un système fonctionnel

---

## 📚 Guides Complets

| Document | Contenu | Pour Qui |
|----------|---------|----------|
| **[API.md](./API.md)** | Endpoints backend détaillés, schemas, exemples | Développeurs backend, intégrateurs |
| **[MCP.md](./MCP.md)** | Endpoints MCP pour IA, use cases, architecture | Développeurs IA, agents, intégrateurs |
| **[ARCHITECTURE.md](./ARCHITECTURE.md)** | Design système, flux, services | Architectes, leads techniques |
| **[DEPLOYMENT.md](./DEPLOYMENT.md)** | Production setup, scaling, monitoring | DevOps, SRE, infra |
| **[CONTRIBUTING.md](./CONTRIBUTING.md)** | Workflow dev, conventions, PR process | Contributeurs, développeurs |

---

## 🚀 Tâches Courantes

### Je veux ajouter un nouvel endpoint

1. **Lire:** `CONTRIBUTING.md` → "Ajouter une Nouvelle Fonctionnalité"
2. **Implémenter:** Code dans `backend-fastapi/` ou `mcp-fast-mcp/`
3. **Tester:** Exemples curl dans QUICKSTART
4. **Documenter:** Ajouter détails dans API.md ou MCP.md
5. **Commit:** Depuis le repo technique, pas orchestrator

---

### Je veux déployer en production

1. **Lire:** `DEPLOYMENT.md` → section pertinente
2. **Configurer:** Variables `.env` et `docker-compose.yml`
3. **Backup:** Setup automatisé
4. **Monitor:** Health checks + logs

---

### Je veux intégrer avec une IA (Claude)

1. **Lire:** `MCP.md` → "Endpoints" + "Intégration Claude API"
2. **Tester:** Exemples POST dans QUICKSTART
3. **Implémenter:** Claude SDK + tools MCP

---

### Je veux comprendre l'architecture

1. **Lire:** `ARCHITECTURE.md` (5 min)
2. **Lire:** `docs/` des repos techniques
3. **Lancer:** `docker compose up && docker compose logs`

---

## 📋 Checklists

### Avant de Push du Code
- [ ] Tests passent (`pytest`)
- [ ] Code suit conventions (voir CONTRIBUTING.md)
- [ ] Documentation mise à jour
- [ ] Commit depuis le bon repo
- [ ] `docker compose up --build` fonctionne

### Avant de Déployer en Prod
- [ ] Health checks configurés
- [ ] Authentification activée (JWT)
- [ ] Database: PostgreSQL (pas SQLite)
- [ ] Backup automatisé
- [ ] Monitoring + alertes
- [ ] Logs centralisés

### Avant d'Intégrer une IA
- [ ] MCP endpoints testés
- [ ] Schemas validés
- [ ] Rate limiting en place
- [ ] Error handling complet

---

## 🔗 Liens Rapides

### Accès Services (Après `docker compose up`)
- Swagger UI: http://localhost:8000/docs
- ReDoc: http://localhost:8000/redoc
- MCP Health: http://localhost:8001/health
- Database: `./data/contacts.db`

### Documentation Technique
- **Backend:** `../backend-fastapi/` (code source)
- **MCP:** `../mcp-fast-mcp/` (code source)
- **Règles:** `../.claude/rules/annuaire-contacts-rules.md` (5 règles)

### Fichiers Config
- `.env.example` — Template variables
- `docker-compose.yml` — Orchestration
- `requirements.txt` — Dépendances (backend + mcp)

---

## 📞 Troubleshooting

**Services ne démarrent pas:**
→ Voir `DEPLOYMENT.md` → "Troubleshooting"

**Je ne comprends pas un endpoint:**
→ Voir `API.md` (backend) ou `MCP.md` (IA)

**Je veux ajouter une feature:**
→ Lire `CONTRIBUTING.md` → "Workflow de Développement"

---

## 🗺️ Roadmap Doc

- [ ] Video tutorial (démarrage 5 min)
- [ ] Diagrammes architecture (Mermaid)
- [ ] API OpenAPI spec téléchargeable
- [ ] Exemples Postman/Insomnia
- [ ] FAQ (questions fréquentes)
- [ ] Glossaire (terminologie)

---

**Dernière mise à jour:** 2026-04-07  
**Versión:** MVP 0.1.0
