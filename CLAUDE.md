# Orchestrateur — Annuaire de Contacts

Dossier de **documentation + règles** (pas de code ni orchestration Docker).

## Architecture du projet

```
annuaire-contacts/
├── orchestrator/              ← ce dossier (DOC + RÈGLES)
│   ├── docs/                  → Toute la documentation
│   ├── CLAUDE.md              → Ce fichier (règles projet)
│   └── .claude/rules/         → 5 règles non-négociables
│
├── backend-fastapi/           ← Service autonome (port 8000)
│   ├── docker-compose.yml     → Démarrage: cd backend-fastapi && docker compose up
│   ├── main.py, models.py
│   └── Dockerfile
│
└── mcp-fast-mcp/              ← Service autonome (port 8001)
    ├── docker-compose.yml     → Démarrage: cd mcp-fast-mcp && docker compose up
    ├── main.py, models.py
    └── Dockerfile
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
2. Ne JAMAIS modifier backend ou MCP depuis ici
3. Clients générés (pas d'URLs hardcodées)
4. Commits depuis repos techniques
5. Docker Compose = source de vérité

## 📚 Documentation

- **[docs/SERVICES.md](./docs/SERVICES.md)** — Démarrage autonome des services
- **[docs/API.md](./docs/API.md)** — Backend endpoints
- **[docs/MCP.md](./docs/MCP.md)** — MCP endpoints pour IA
- **[docs/ARCHITECTURE.md](./docs/ARCHITECTURE.md)** — Design système
- **[README.md](./README.md)** — Quick start
