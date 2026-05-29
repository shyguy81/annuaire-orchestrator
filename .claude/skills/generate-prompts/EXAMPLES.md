# Exemples de Prompts

## ✏️ Exemple 1: Correction simple

````markdown
---
name: typo-doc
description: Corriger typos dans la documentation
agent: agent
---

# Fix: Typo dans la doc

Corriger "teh" → "the" dans docs/API.md

## ✅ Fin du traitement

```bash
TIMESTAMP=$(date +%Y%m%d-%H%M)
mv .github/prompts/edit--inbox--typo-doc-$TIMESTAMP.prompt.md \
   .github/prompts/edit--done--typo-doc-$TIMESTAMP.prompt.md
```
````

````

## ✨ Exemple 2: Ajout de fonction

```markdown
---
name: export-json
description: Ajouter endpoint pour exporter contacts en JSON
agent: agent
---

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
TIMESTAMP=$(date +%Y%m%d-%H%M)
mv .github/prompts/feature--inbox--export-json-$TIMESTAMP.prompt.md \
   .github/prompts/feature--done--export-json-$TIMESTAMP.prompt.md
````

````

## 🔄 Exemple 3: Refactorisation

```markdown
---
name: decouple-validation
description: Découpler la logique de validation dans un fichier dédié
agent: agent
---

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
TIMESTAMP=$(date +%Y%m%d-%H%M)
mv .github/prompts/refactor--inbox--decouple-validation-$TIMESTAMP.prompt.md \
   .github/prompts/refactor--done--decouple-validation-$TIMESTAMP.prompt.md
````

````

## 🎓 Exemple 4: Refactorisation complexe avec contexte

```markdown
# Refactor: Migrer vers Pydantic v2

## Contexte
Nous upgradons Pydantic de v1 à v2 pour bénéficier:
- Performance améliorée
- Validation plus stricte
- Support JSON Schema natif

## Tâche
- Mettre à jour imports de `pydantic` v1 → v2
- Adapter les modèles (Field, BaseModel, etc.)
- Vérifier tous les tests passent
- Mettre à jour la doc

## Validation
- `pytest` 100% pass
- Aucun avertissement deprecation
- Types vérifiés

## ✅ Fin du traitement
```bash
mv .github/prompts/refactor--inbox--pydantic-v2.md \
   .github/prompts/refactor--done--pydantic-v2.md
````

```

```
