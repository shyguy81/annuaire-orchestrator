# Guide — Créer et Exécuter un Prompt

## ⚠️ Règle 2: Ne jamais modifier depuis l'orchestrator

Les modifications **doivent se faire dans le repo technique**, pas depuis l'orchestrator.

**Énoncé:** `annuaire-fastapi/`, `mcp-fast-mcp/` et `annuaire-cli/` ne se modifient **jamais** depuis ce dossier (l'orchestrator).

**Pourquoi:** Repos technique = source de truth. Git history et CI/CD en dépendent.

**Flux correct:**

```
orchestrator → créer prompt dans repo technique
→ VS Code ouvre et exécute le prompt
→ modifications appliquées dans repo technique
→ commit DEPUIS repo technique (pas depuis orchestrator)
```

## 🎯 Étapes

### 1️⃣ Naviguer vers le repo technique

```bash
cd ../annuaire-fastapi
# ou
cd ../mcp-fast-mcp
# ou
cd ../annuaire-cli
```

### 2️⃣ Créer le prompt

````bash
TIMESTAMP=$(date +%Y%m%d-%H%M)
cat > .github/prompts/edit--inbox--fix-contact-validation-$TIMESTAMP.prompt.md << 'EOF'
---
name: fix-contact-validation
description: Améliorer la validation des contacts pour accepter accents
agent: agent
---

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
mv .github/prompts/edit--inbox--fix-contact-validation-$TIMESTAMP.prompt.md \
   .github/prompts/edit--done--fix-contact-validation-$TIMESTAMP.prompt.md
````

EOF

````

### 3️⃣ Ouvrir le prompt dans VS Code

```bash
# Le fichier est créé et prêt à être ouvert
# Depuis VS Code: File → Open et naviguer vers .github/prompts/
````

### 4️⃣ Invoquer Copilot

```
Ouvrir le prompt → Chat → @edit--inbox--fix-contact-validation
```

### 5️⃣ Exécuter les modifications

Copilot applique les changements suggérés dans le prompt.

### 6️⃣ Renommer le fichier

Une fois que les modifications sont terminées et testées:

```bash
TIMESTAMP=$(date +%Y%m%d-%H%M)
mv .github/prompts/edit--inbox--fix-contact-validation-$TIMESTAMP.prompt.md \
   .github/prompts/edit--done--fix-contact-validation-$TIMESTAMP.prompt.md
```

### 7️⃣ Committer

```bash
git add .
git commit -m "fix: améliorer validation contacts"
git push
```
