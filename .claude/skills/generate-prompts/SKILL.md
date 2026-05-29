---
name: generate-prompts
description: Génère des prompts pour piloter les dossiers techniques (`annuaire-fastapi/`, `mcp-fast-mcp/`, `annuaire-cli/`) via des **prompts jetables** déposés dans `.github/prompts/` de chaque repo.
---

# Skill: Prompts Technique

Piloter les dossiers techniques (`annuaire-fastapi/`, `mcp-fast-mcp/`, `annuaire-cli/`) via des **prompts jetables** déposés dans `.github/prompts/` de chaque repo.

## 🎯 Principes

- **Règle 2 du projet:** Ne jamais modifier les repos techniques depuis l'orchestrator
- **Pattern:** Créer un prompt auto-exécutable avec instruction de renommage intégrée
- **Isolation:** Chaque modification reste commitée dans son propre repo
- **Traçabilité:** Chaque prompt auto-documente son cycle de vie

## 📋 Workflow

### Étape 1: Créer et déposer le prompt dans le repo technique

````bash
# Naviguer dans le repo technique
cd ../annuaire-fastapi

# Créer le prompt avec instruction complète
cat > .github/prompts/edit--inbox--fix-contact-validation.md << 'EOF'
# Fix: Améliorer la validation des contacts

## Contexte
La validation actuelle n'accepte pas les noms avec accents.

## Tâche
- Modifier `models.py` pour accepter les accents
- Ajouter un test dans `test_models.py`
- Mettre à jour la doc

## Validation
- Tests: `pytest tests/` passe
- Linting: `flake8` clean

## ✅ Fin du traitement
Une fois les modifications terminées, renommer ce fichier:
```bash
mv .github/prompts/edit--inbox--fix-contact-validation.md \
   .github/prompts/edit--done--fix-contact-validation.md
````

EOF

```

Le prompt est prêt. Ouvrir le fichier dans VS Code et exécuter le prompt avec Copilot.

## 🗂️ Structure attendue

Chaque repo technique doit avoir:

```

.github/
├── prompts/
│ ├── edit--inbox--xxx.md ← À traiter
│ ├── edit--inbox--yyy.md ← À traiter
│ ├── edit--done--old-fix.md ← Archivé
│ └── refactor--done--api.md ← Archivé

````

## 🏷️ Conventions de nommage

| Préfixe             | Sens                          |
| ------------------- | ----------------------------- |
| `edit--inbox--`     | Nouvelle modification à faire |
| `edit--done--`      | Modification exécutée         |
| `refactor--inbox--` | Refactorisation à faire       |
| `refactor--done--`  | Refactorisation exécutée      |
| `feature--inbox--`  | Feature à implémenter         |
| `feature--done--`   | Feature implémentée           |

## 💡 Exemples de prompts

### Template complet

```markdown
# [Type]: [Titre court]

## Contexte (optionnel)
Explications utiles...

## Tâche
- Étape 1
- Étape 2
- Étape 3

## Validation
- Critère de succès 1
- Critère de succès 2

## ✅ Fin du traitement
Renommer ce fichier:
```bash
mv .github/prompts/[type]--inbox--[nom].md \
   .github/prompts/[type]--done--[nom].md
````

````

### Exemple 1: Correction simple

```markdown
# Fix: Typo dans la doc

Corriger "teh" → "the" dans docs/API.md

## ✅ Fin du traitement
```bash
mv .github/prompts/edit--inbox--typo-doc.md \
   .github/prompts/edit--done--typo-doc.md
````

````

### Exemple 2: Ajout de fonction

```markdown
# Feature: Export JSON des contacts

## Tâche
- Ajouter endpoint `GET /contacts/export.json` dans `main.py`
- Retourner tous les contacts en JSON
- Ajouter test dans `test_api.py`

## Validation
- `pytest` passe
- `curl http://localhost:8000/contacts/export.json` fonctionne

## ✅ Fin du traitement
```bash
mv .github/prompts/feature--inbox--export-json.md \
   .github/prompts/feature--done--export-json.md
````

````

### Exemple 3: Refactorisation

```markdown
# Refactor: Découpler la logique de validation

## Contexte
`models.py` est responsable à la fois de la validation ET du mapping DB.

## Tâche
- Créer `validators.py` avec logique de validation
- Importer depuis `models.py`
- Vérifier tests passent

## Validation
- `pytest` 100% pass
- Coverage > 80%

## ✅ Fin du traitement
```bash
mv .github/prompts/refactor--inbox--decouple-validation.md \
   .github/prompts/refactor--done--decouple-validation.md
````

```

## 🔄 Cycle de vie

```

Créer prompt (--inbox--)
↓
Ouvrir dans VS Code
↓
Invoquer @prompt-name
↓
Copilot exécute les modifications
↓
Renommer en --done--
↓
Commit depuis le repo technique

````

## ⚙️ Intégration avec orchestrator

Depuis l'orchestrator:

1. **Créer des prompts** dans les repos techniques
2. **Vérifier l'avancement** en checkant `.github/prompts/` de chaque repo
3. **Coordonner multi-repos** en créant des prompts en cascade
4. **Ne pas modifier directement** — toutes les modifications restent dans le repo technique

### Exemple: Coordination multi-repos

```markdown
# Coordination: Ajouter authentification JWT

Créer ces prompts dans leurs repos respectifs:
- [ ] `feature--inbox--jwt-auth.md` dans `annuaire-fastapi`
- [ ] `feature--inbox--jwt-client.md` dans `mcp-fast-mcp`
- [ ] `feature--inbox--jwt-cli.md` dans `annuaire-cli`

Ordre d'exécution: backend → mcp → cli
````

## ✅ Bonnes pratiques

À faire:

- Un prompt = une tâche atomique
- Inclure contexte + validation dans le prompt
- Renommer après validation
- Commit depuis le repo technique

À éviter:

- Prompts trop vagues ou complexes
- Modifications depuis orchestrator
- Oublier de renommer en `--done--`
- Commit depuis l'orchestrator

## 📚 Voir aussi

- [Règle 2: Ne jamais modifier depuis orchestrator](./../rules/annuaire-contacts-rules.md#règle-2-ne-jamais-modifier-le-code-des-repos-liés)
- [Règle 4: Commits depuis repos techniques](./../rules/annuaire-contacts-rules.md#règle-4-commiter-depuis-les-repos-techniques-pas-depuis-orchestrator)
