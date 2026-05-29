---
name: generate-prompts
description: Génère des prompts pour piloter les dossiers techniques (`annuaire-fastapi/`, `mcp-fast-mcp/`, `annuaire-cli/`) via des **prompts jetables** déposés dans `.github/prompts/` de chaque repo.
---

# Skill: Prompts Technique

Piloter les dossiers techniques via des **prompts jetables** auto-documentés et exécutables.

## 📋 Contexte: Règle 2 (Fondation)

### Énoncé

`annuaire-fastapi/`, `mcp-fast-mcp/` et `annuaire-cli/` ne se modifient **jamais** depuis l'orchestrator.

### Pourquoi?

- **Repos technique = source de truth** des modifications
- **Orchestrator = documentation + coordination** seulement
- Sinon: git history perdue, CI/CD échoue, changements perdus

### Conséquences si non-respectée

- ❌ Modifs depuis ici = pas committées dans leur repo
- ❌ Services restent en version ancienne
- ❌ Impossible d'auditer les changements
- ❌ Git history incohérente

### Comment respecter cette règle?

**Solution de cette skill:** Déposer les prompts **directement dans le repo technique** (`.github/prompts/`) pour que Copilot les exécute là-bas et commit dans le bon repo.

## 🎯 Principes

- **Pattern:** Créer un prompt auto-exécutable (renommage inclus)
- **Isolation:** Chaque modification reste commitée dans son repo
- **Traçabilité:** Chaque prompt auto-documente son cycle de vie
- **Respect de Règle 2:** Modifications dans le repo technique, pas depuis l'orchestrator

## 📖 Référence rapide

| Besoin               | Fichier                                  | Contenu                                            |
| -------------------- | ---------------------------------------- | -------------------------------------------------- |
| **Structure**        | [STRUCTURE.md](./STRUCTURE.md)           | Dossier `.github/prompts/`, conventions de nommage |
| **Guide complet**    | [GUIDE.md](./GUIDE.md)                   | Étapes pour créer et exécuter un prompt            |
| **Exemples**         | [EXAMPLES.md](./EXAMPLES.md)             | 4 exemples concrets (fix, feature, refactor)       |
| **Bonnes pratiques** | [BEST-PRACTICES.md](./BEST-PRACTICES.md) | À faire / À éviter + conseils                      |

## 🚀 Quick Start

```bash
# 1. Naviguer dans le repo technique
cd ../annuaire-fastapi

# 2. Créer le prompt
TIMESTAMP=$(date +%Y%m%d-%H%M)
cat > .github/prompts/edit--inbox--[nom]-$TIMESTAMP.prompt.md << 'PROMPT'
---
name: edit-[nom]
description: [Description courte]
agent: agent
---

# Fix/Feature/Refactor: [Titre]

## Contexte
...

## Tâche
- ...

## Validation
- ...

## ✅ Fin du traitement
mv .github/prompts/edit--inbox--[nom]-$TIMESTAMP.prompt.md \
   .github/prompts/edit--done--[nom]-$TIMESTAMP.prompt.md
PROMPT

# 3. Ouvrir et invoquer depuis VS Code
# 4. Renommer --inbox-- → --done--
# 5. Commit
```
