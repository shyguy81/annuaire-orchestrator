---
name: generate-prompts
description: Génère des prompts pour piloter les dossiers techniques (`annuaire-fastapi/`, `mcp-fast-mcp/`, `annuaire-cli/`) via des **prompts jetables** déposés dans `.github/prompts/` de chaque repo.
---

# Skill: Prompts Technique

Piloter les dossiers techniques (`annuaire-fastapi/`, `mcp-fast-mcp/`, `annuaire-cli/`) via des **prompts jetables** déposés dans `.github/prompts/` de chaque repo.

## 🎯 Principes

- **Règle 2 du projet:** Ne jamais modifier les repos techniques depuis l'orchestrator
- **Pattern:** Créer un prompt → naviguer dans le repo → exécuter le prompt → archiver
- **Isolation:** Chaque modification reste commitée dans son propre repo
- **Traçabilité:** `git log` reflète l'historique complet

## 📋 Workflow

### Étape 1: Créer un prompt jetable dans le repo technique

```bash
# Exemple: modifier le backend
cd ../annuaire-fastapi

# Créer le prompt d'inbox
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
EOF
```

### Étape 2: Inviter Copilot avec le prompt

```bash
# Dans VS Code, au sein du repo annuaire-fastapi
# Ouvrir le prompt créé et invoquer: @edit--inbox--fix-contact-validation
```

### Étape 3: Archiver après exécution

```bash
# Une fois les modifications terminées
mv .github/prompts/edit--inbox--fix-contact-validation.md \
   .github/prompts/edit--done--fix-contact-validation.md
```

### Étape 4: Committer les modifications

```bash
# Après validation du prompt, committer tout:
git add .
git commit -m "[message pertinent décrivant les changements]"
git push
```

### Étape 5: Revenir à orchestrator

```bash
cd ../annuaire-orchestrator
# Documentation ou coordination suivante
```

## 🗂️ Structure attendue

Chaque repo technique (`annuaire-fastapi/`, `mcp-fast-mcp/`, `annuaire-cli/`) doit avoir:

```
.github/
├── prompts/
│   ├── edit--inbox--xxx.md          ← À traiter
│   ├── edit--inbox--yyy.md          ← À traiter
│   ├── edit--done--old-fix.md       ← Archivé
│   └── refactor--done--api.md       ← Archivé
└── workflows/                        ← CI/CD (optionnel)
```

### Conventions de nommage

| Préfixe             | Sens                          |
| ------------------- | ----------------------------- |
| `edit--inbox--`     | Nouvelle modification à faire |
| `edit--done--`      | Modification exécutée         |
| `refactor--inbox--` | Refactorisation à faire       |
| `refactor--done--`  | Refactorisation exécutée      |
| `feature--inbox--`  | Feature à implémenter         |
| `feature--done--`   | Feature implémentée           |

## 💡 Exemples de prompts

### Template de prompt complet

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

## Après validation
Renommer le fichier:
```bash
mv .github/prompts/[type]--inbox--[nom].md \
   .github/prompts/[type]--done--[nom].md
```
```

### Exemple 1: Correction simple

```markdown
# Fix: Typo dans la doc

Corriger "teh" → "the" dans [docs/API.md](docs/API.md)

## Après validation
```bash
mv .github/prompts/edit--inbox--typo-doc.md \
   .github/prompts/edit--done--typo-doc.md
```
```

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

## Après validation
```bash
mv .github/prompts/feature--inbox--export-json.md \
   .github/prompts/feature--done--export-json.md
```
```

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

## Après validation
```bash
mv .github/prompts/refactor--inbox--decouple-validation.md \
   .github/prompts/refactor--done--decouple-validation.md
```
```

## 🔄 Cycle de vie d'un prompt

```
inbox (À faire)
    ↓
Ouvert dans Copilot
    ↓
Modifications appliquées
    ↓
Tests passent
    ↓
done (Archivé)
    ↓
git commit depuis le repo technique
```

## ⚙️ Intégration avec orchestrator

**Depuis l'orchestrator**, on peut:

1. **Documenter la tâche** → crée un prompt dans le bon repo
2. **Vérifier l'avancement** → checker `.github/prompts/` de chaque repo
3. **Coordonner plusieurs repos** → créer prompts en cascade
4. **Pas modifier directement** → les modifications restent dans leur repo

### Exemple: Coordination multi-repos

```markdown
# Coordination: Ajouter authentification JWT

## Repos affectés

1. **annuaire-fastapi:** Implémenter auth JWT
2. **mcp-fast-mcp:** Utiliser le nouveau client auth
3. **annuaire-cli:** Passer le token aux appels

## Ordre d'exécution

1. Backend d'abord (endpoints)
2. MCP ensuite (client)
3. CLI en dernier (CLI)

## Prompts à créer

- [ ] `.github/prompts/feature--inbox--jwt-auth.md` dans `annuaire-fastapi`
- [ ] `.github/prompts/feature--inbox--jwt-client.md` dans `mcp-fast-mcp`
- [ ] `.github/prompts/feature--inbox--jwt-cli.md` dans `annuaire-cli`
```

## 🎓 Bonnes pratiques

✅ **À faire:**

- Un prompt = une tâche atomique
- Inclure contexte + validation
- Archiver après exécution
- Commit depuis le repo technique

❌ **À éviter:**

- Prompts trop vagues
- Modifications depuis orchestrator (même accidentelles)
- Oublier d'archiver
- Commit depuis l'orchestrator

## 📚 Voir aussi

- [Règle 2: Ne jamais modifier depuis orchestrator](./../rules/annuaire-contacts-rules.md#règle-2-ne-jamais-modifier-le-code-des-repos-liés)
- [Règle 4: Commits depuis repos techniques](./../rules/annuaire-contacts-rules.md#règle-4-commiter-depuis-les-repos-techniques-pas-depuis-orchestrator)
