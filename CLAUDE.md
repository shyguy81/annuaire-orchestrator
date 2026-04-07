# Orchestrateur — Annuaire de Contacts

## Vue d'ensemble

Ce dossier contient l'orchestrateur Python du projet **annuaire-contacts**.
Il coordonne les interactions entre les différents composants du système.

## Architecture du projet

```
annuaire-contacts/
├── orchestrator/        ← ce dossier (modifiable)
├── backend-fastapi/     ← API REST Python/FastAPI (NON modifiable depuis l'orchestrateur)
└── mcp-fast-mcp/        ← Serveur MCP (NON modifiable depuis l'orchestrateur)
```

## Rôle de l'orchestrateur

- Orchestration des flux entre le backend FastAPI et le serveur MCP
- Point d'entrée principal pour l'exécution du système
- Coordination et intégration des services

## Contraintes importantes

- Le dossier `../backend-fastapi` est **en lecture seule** depuis l'orchestrateur : ne pas modifier son code.
- Le dossier `../mcp-fast-mcp` est **en lecture seule** depuis l'orchestrateur : ne pas modifier son code.
- Toute modification fonctionnelle doit passer par les interfaces exposées (API HTTP, protocole MCP).

## 🚨 Règles Essentielles

Voir [.claude/rules/annuaire-contacts-rules.md](./.claude/rules/annuaire-contacts-rules.md) pour les 5 règles non-négociables.
