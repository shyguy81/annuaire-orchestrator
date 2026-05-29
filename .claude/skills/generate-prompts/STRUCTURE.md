# Structure des Prompts

## 🗂️ Dossier `.github/prompts/`

Chaque repo technique doit avoir:

```
.github/
├── prompts/
│   ├── edit--inbox--xxx-20260529-1430.prompt.md          ← À traiter
│   ├── edit--inbox--yyy-20260529-1530.prompt.md          ← À traiter
│   ├── edit--done--old-fix-20260528-1000.prompt.md       ← Archivé
│   └── refactor--done--api-20260527-0930.prompt.md       ← Archivé
```

## 🏷️ Conventions de nommage

| Préfixe             | État    | Extension  | Format complet                                   |
| ------------------- | ------- | ---------- | ------------------------------------------------ |
| `edit--inbox--`     | À faire | .prompt.md | `edit--inbox--[nom]-YYYYMMDD-HHMM.prompt.md`     |
| `edit--done--`      | Fait    | .prompt.md | `edit--done--[nom]-YYYYMMDD-HHMM.prompt.md`      |
| `refactor--inbox--` | À faire | .prompt.md | `refactor--inbox--[nom]-YYYYMMDD-HHMM.prompt.md` |
| `refactor--done--`  | Fait    | .prompt.md | `refactor--done--[nom]-YYYYMMDD-HHMM.prompt.md`  |
| `feature--inbox--`  | À faire | .prompt.md | `feature--inbox--[nom]-YYYYMMDD-HHMM.prompt.md`  |
| `feature--done--`   | Fait    | .prompt.md | `feature--done--[nom]-YYYYMMDD-HHMM.prompt.md`   |

## 📝 Structure minimale d'un prompt

````markdown
---
name: [type]-[nom]
description: [Description courte du prompt]
agent: agent
---

# [Type]: [Titre]

## Contexte (optionnel)

...

## Tâche

- ...

## Validation

- ...

## ✅ Fin du traitement

```bash
TIMESTAMP=$(date +%Y%m%d-%H%M)
mv .github/prompts/[type]--inbox--[nom]-$TIMESTAMP.prompt.md \
   .github/prompts/[type]--done--[nom]-$TIMESTAMP.prompt.md
```
````

```

```
