# Bonnes Pratiques — Prompts Jetables

## 🚨 Règle 2: Ne JAMAIS modifier depuis l'orchestrator

**Énoncé:** `annuaire-fastapi/`, `mcp-fast-mcp/` et `annuaire-cli/` ne se modifient **jamais** depuis l'orchestrator.

**Pourquoi:** Repos technique = source de truth. Orchestrator = documentation + coordination seulement.

**Conséquences si non-respectée:**

- ❌ Modifications perdues (pas committées dans le bon repo)
- ❌ Git history incohérente
- ❌ CI/CD échoue
- ❌ Services restent en version ancienne
- ❌ Impossible d'auditer les changements

**Comment Faire:**

```bash
# ❌ MAUVAIS
cd ../annuaire-fastapi
# modifier src/main.py
# commit depuis orchestrator

# ✅ BON
# 1. Créer prompt dans .github/prompts/ du repo technique
# 2. Ouvrir et exécuter depuis VS Code
# 3. Renommer --inbox-- → --done--
# 4. Commit DEPUIS ce repo technique
```

**Cette skill = solution:** Déposer les prompts **dans le repo technique** pour que Copilot les exécute et commit là-bas.

## ✅ À faire

### Atomicité

- **Un prompt = une tâche** bien définie et indépendante
- Inclure contexte + validation + renommage
- Tâche complète dans le prompt (pas de suite attendue)

### Qualité

- **En-tête YAML** — `name`, `description`, `agent` obligatoires
  ```yaml
  ---
  name: fix-contact-validation
  description: Améliorer la validation des contacts
  agent: agent
  ---
  ```
- **Contexte clair** — pourquoi cette modification?
- **Validation explicite** — comment vérifier que ça marche?
- **Renommage inclus** — instruction dans le prompt

### Workflow

- Créer le prompt dans le repo technique avec timestamp en suffixe
- Extension `.prompt.md` (pas `.md`)
- Ouvrir et exécuter depuis VS Code
- Renommer `--inbox--` → `--done--` après validation
- Commit depuis ce repo (jamais depuis orchestrator)

## ❌ À éviter

### Problèmes courants

- **Prompts trop vagues** — "améliorer le code" n'est pas clair
- **Prompts en cascade** — dépendre d'un autre prompt pour fonctionner
- **Modifications depuis orchestrator** — même accidentellement
- **Oublier l'en-tête YAML** — `name`, `description`, `agent` obligatoires
- **Oublier le renommage** — archivage manuel oublié
- **Oublier le timestamp** — traçabilité perdue
- **Mauvaise extension** — utiliser `.md` au lieu de `.prompt.md`
- **Commit depuis l'orchestrator** — history incohérente

### Antipatterns

- Mélanger plusieurs tâches dans un prompt
- Prescrire l'implémentation exacte (laisser Copilot décider)
- Prompts sans contexte de pourquoi
- Validation impossible à vérifier

## 💡 Conseils

### Pour des prompts efficaces

- **Générer le timestamp** avant création: `TIMESTAMP=$(date +%Y%m%d-%H%M)`
- **En-tête YAML** en premier (name, description, agent)
- Inclure le timestamp dans le **nom du fichier** (pas dans `name`)
- Extension `.prompt.md` (pas `.md`)
- Commencer par le **Contexte** (pourquoi?)
- Puis la **Tâche** (quoi? comment?)
- Puis la **Validation** (comment je sais que c'est bon?)
- Terminer par **Fin du traitement** (renommage avec timestamp)

### Pour la coordination

- Créer les prompts dans **l'ordre d'exécution**
- Documenter les dépendances (ex: "après JWT")
- Vérifier l'avancement en checkant `.github/prompts/`

### Pour la maintenance

- **Archiver régulièrement** les `--done--`
- Créer une **checklist** des prompts à faire
- **Documenter les prompts bloquants** au-delà de 1 jour
