# Documentation Index

Bienvenue dans l'annuaire-contacts! Voici comment naviguer la documentation.

---

## 📖 Démarrage Rapide

**Je suis nouveau et je veux démarrer:**
1. Voir [`ARCHITECTURE-DIAGRAM.md`](./ARCHITECTURE-DIAGRAM.md) (2 min, diagramme visuel)
2. Lire [`README.md`](../README.md) (3 min)
3. Lancer services (voir [`SERVICES.md`](./SERVICES.md)) (2 min)
4. Essayer exemples curl dans [`QUICKSTART.md`](../QUICKSTART.md) (5 min)

**Total:** ~12 min pour comprendre l'architecture et avoir un système fonctionnel

---

## 📚 Guides Complets

| Document | Contenu | Pour Qui |
|----------|---------|----------|
| **[ARCHITECTURE-DIAGRAM.md](./ARCHITECTURE-DIAGRAM.md)** | 📊 Diagramme visuel + composants + flux | Tous (vue globale d'abord) |
| **[SERVICES.md](./SERVICES.md)** | Démarrage autonome, options lancement | Tous (lire en premier après README) |
| **[API.md](./API.md)** | Endpoints backend détaillés, schemas, exemples | Développeurs backend, intégrateurs |
| **[MCP.md](./MCP.md)** | Endpoints MCP pour IA, use cases, architecture | Développeurs IA, agents, intégrateurs |
| **[ARCHITECTURE.md](./ARCHITECTURE.md)** | Design système, flux, services | Architectes, leads techniques |
| **[DEPLOYMENT.md](./DEPLOYMENT.md)** | Production setup, scaling, monitoring | DevOps, SRE, infra |
| **[CONTRIBUTING.md](./CONTRIBUTING.md)** | Workflow dev, conventions, PR process | Contributeurs, développeurs |

---

## 🚀 Tâches Courantes

### Je veux démarrer les services

1. **Lire:** `SERVICES.md` (5 min)
2. **Terminal 1:** `cd annuaire-fastapi && docker compose up`
3. **Terminal 2:** `cd mcp-fast-mcp && docker compose up`
4. **Tester:** Exemples curl dans QUICKSTART.md

---

### Je veux ajouter un nouvel endpoint

1. **Lire:** `CONTRIBUTING.md` → "Ajouter une Nouvelle Fonctionnalité"
2. **Implémenter:** Code dans `annuaire-fastapi/` ou `mcp-fast-mcp/`
3. **Tester:** `docker compose up --build` depuis le dossier
4. **Documenter:** Ajouter détails dans API.md ou MCP.md
5. **Commit:** Depuis le repo technique

---

### Je veux déployer en production

1. **Lire:** `DEPLOYMENT.md` → section pertinente
2. **Configurer:** Variables `.env` pour chaque service
3. **Lancer:** Services indépendants ou avec Docker network
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

### Accès Services (Après démarrage depuis chaque dossier)

**Backend (port 8000):**
- Swagger UI: http://localhost:8000/docs
- ReDoc: http://localhost:8000/redoc
- Health: http://localhost:8000/health

**MCP (via Nginx port 80/443):**
- Health HTTP: http://localhost/health
- Health HTTPS: https://localhost/health (certificat auto-signé, use `-k`)
- Search HTTP: http://localhost/search
- Search HTTPS: https://localhost/search

**Database:**
- MariaDB: `localhost:3307` (mapping dev, container: 3306)
- Credentials: `annuaire_user` / `annuaire_password`
- Database: `annuaire_contacts`

### Documentation Technique
- **Backend:** `../annuaire-fastapi/` (code source)
- **MCP:** `../mcp-fast-mcp/` (code source)
- **Règles:** `../.claude/rules/annuaire-contacts-rules.md` (5 règles)

### Fichiers Config
- `annuaire-fastapi/.env.example` — Template variables backend
- `mcp-fast-mcp/.env.example` — Template variables MCP
- `annuaire-fastapi/docker-compose.yml` — Service backend
- `mcp-fast-mcp/docker-compose.yml` — Service MCP
- `annuaire-fastapi/requirements.txt` — Dépendances backend
- `mcp-fast-mcp/requirements.txt` — Dépendances MCP

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

- [x] Diagrammes architecture (Mermaid) — `ARCHITECTURE-DIAGRAM.md`
- [ ] Video tutorial (démarrage 5 min)
- [ ] API OpenAPI spec téléchargeable
- [ ] Exemples Postman/Insomnia
- [ ] FAQ (questions fréquentes)
- [ ] Glossaire (terminologie)
- [ ] Schéma de données (ERD)

---

**Dernière mise à jour:** 2026-04-07  
**Versión:** MVP 0.1.0
