# Diagrammes RAP System v1

Trois diagrammes Mermaid (compilés via mmdc en haute résolution) présentent l'évolution du système vers le RAP.

## 1. Schéma de Base de Données (ER Diagram)

**Fichier:** `rap-system-schema.png` (4720 × 2030 px)

Montre la structure relationnelle des 4 entités:

```
CONTACT (existant)
  ├── 1:1 RELATIONSHIP_PROFILE (nouveau)
  ├── 1:N INTERACTION (nouveau)
  └── 1:N RELATIONSHIP_ACTION (nouveau)
```

### Champs par entité:

**CONTACT**
- `id` (UUID, PK)
- `nom`, `email` (unique), `telephone`, `adresse`, `organisation`
- `tags` (JSON)
- `created_at`, `updated_at`

**RELATIONSHIP_PROFILE** (1:1)
- `id` (UUID, PK)
- `contact_id` (FK, unique → 1:1)
- `relationship_type` : prospect|client|partner|referrer|mentor|other
- `proximity_level` : cold|warm|active|close
- `business_potential` : unknown|low|medium|high
- `trust_level` : 1-5
- `context`, `notes`

**INTERACTION** (1:N)
- `id` (UUID, PK)
- `contact_id` (FK)
- `interaction_type` : email|call|meeting|linkedin|event|note
- `interaction_date` (indexed, searchable)
- `summary`, `detected_need`, `value_given`, `next_step`

**RELATIONSHIP_ACTION** (1:N)
- `id` (UUID, PK)
- `contact_id` (FK)
- `action_type` : follow_up|send_resource|invite_coffee|propose_audit|introduce|other
- `due_date` (indexed, filterable)
- `status` : todo|done|skipped
- `priority` : low|medium|high
- `description`, `outcome`
- `completed_at` (null until marked done)

---

## 2. Endpoints API (Flowchart)

**Fichier:** `rap-api-endpoints.png` (4768 × 1372 px)

Montre 12 nouveaux endpoints groupés par domaine + interactions:

### CONTACT (existant, inchangé)
- `POST /contacts` → créer
- `GET /contacts` → lister
- `GET /contacts/{id}` → détail
- `PUT /contacts/{id}` → mettre à jour
- `DELETE /contacts/{id}` → supprimer (cascade!)
- `GET /contacts/search/{query}` → chercher

### RELATIONSHIP_PROFILE (nouveau)
- `POST /contacts/{cid}/relationship-profile` → 201
- `GET /contacts/{cid}/relationship-profile` → 200/404
- `PATCH /contacts/{cid}/relationship-profile` → 200/404

### INTERACTIONS (nouveau)
- `POST /contacts/{cid}/interactions` → 201
- `GET /contacts/{cid}/interactions` → 200 (list)

### RELATIONSHIP_ACTIONS (nouveau)
- `POST /contacts/{cid}/relationship-actions` → 201
- `GET /relationship-actions` → 200 (list, queryable: status, priority)
- `GET /relationship-actions/due` → 200 (today + overdue)
- `PATCH /relationship-actions/{aid}` → 200 (update any field)
- `PATCH /relationship-actions/{aid}/complete` → 200 (mark done + set completed_at)

### RAP DASHBOARD (nouveau)
- `GET /rap/dashboard` → 200
  - `due_today` (int) — actions dues aujourd'hui
  - `overdue` (int) — actions en retard
  - `active_relations` (int) — contacts avec proximity in [warm, active, close]
  - `high_potential` (int) — contacts avec business_potential=high
  - `recent_interactions` (int) — interactions des 7 derniers jours

---

## 3. Workflow (Sequence Diagram)

**Fichier:** `rap-workflow.png` (2556 × 3838 px)

Montre les 6 scénarios clés:

### 1️⃣ Créer Contact + Profil Relationnel
```
POST /contacts → Contact créé (UUID)
POST /contacts/{cid}/relationship-profile → Profil attaché
```

### 2️⃣ Enregistrer Interaction
```
POST /contacts/{cid}/interactions → Interaction enregistrée
```

### 3️⃣ Créer Action de Suivi
```
POST /contacts/{cid}/relationship-actions → Action todo créée
```

### 4️⃣ Consulter Dashboard
```
GET /rap/dashboard → Métriques agrégées (counts)
```

### 5️⃣ Compléter Action
```
PATCH /relationship-actions/{aid}/complete → status=done + completed_at=NOW()
```

### 6️⃣ Supprimer Contact (Cascades)
```
DELETE /contacts/{cid} → DELETE RelationshipProfile, Interactions, Actions
```

---

## Fichiers Générés

```
rapido-contacts/annuaire-orchestrator/
├── rap-system-schema.mmd         → Source Mermaid (ER)
├── rap-system-schema.png         → PNG compilé (4720×2030)
├── rap-api-endpoints.mmd         → Source Mermaid (Flowchart)
├── rap-api-endpoints.png         → PNG compilé (4768×1372)
├── rap-workflow.mmd              → Source Mermaid (Sequence)
├── rap-workflow.png              → PNG compilé (2556×3838)
└── RAP-DIAGRAMS.md               → Ce fichier
```

---

## Comment Utiliser

### Visualiser les PNG
```bash
# Linux
open rap-system-schema.png          # macOS
xdg-open rap-system-schema.png      # Linux

# Tous les diagrammes
file rap-*.png
```

### Régénérer après modification
```bash
# Si vous modifiez les .mmd, recompiler:
mmdc -i rap-system-schema.mmd -o rap-system-schema.png --width 2400 --height 1800 --scale 2
mmdc -i rap-api-endpoints.mmd -o rap-api-endpoints.png --width 2400 --height 1800 --scale 2
mmdc -i rap-workflow.mmd -o rap-workflow.png --width 2400 --height 1600 --scale 2
```

### Intégrer dans une présentation
- Haute résolution (4K-ready)
- Format PNG (transparent OK)
- Adaptés pour slides et documentation

---

## Références

- **ER Diagram:** Montre les relations 1:1 et 1:N, avec enums
- **Endpoints Flowchart:** Hiérarchie des 12 nouveaux endpoints + cascade DELETE
- **Workflow Sequence:** 6 scénarios courants (CRUD + Dashboard + Cascade)

**Généré:** 2026-05-29  
**Mode:** Haute résolution (mmdc scale 2)
