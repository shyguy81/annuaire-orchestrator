# Master Backlog — Annuaire Contacts (Centralisé)

**Unique source of truth pour toutes les tâches du projet.**

---

## Méthodologie

**ICE = (Impact × Confidence) / Effort**

- **Impact:** 1-10 (valeur utilisateur)
- **Confidence:** 0-100% (certitude faisabilité)
- **Effort:** 1-10 (charge de travail en jours)
- **Score:** ≥ 8.0 = P0 | 5-8 = P1 | < 5 = P2

**Status:**
- 🔴 Ready = prêt à démarrer
- 🟡 Blocked = en attente de dépendance
- 🟢 In Progress = en cours
- ✅ Done = terminé

---

## Phase 1: RAP System Core (MVP) — P0

### RAP-1.0: Database Migration (MariaDB → PostgreSQL)

| Métrique | Valeur |
|----------|--------|
| Impact | 10 |
| Confidence | 95% |
| Effort | 3 |
| **ICE Score** | **31.67** |
| Priorité | 🔴 P0 |
| Status | 🔴 Ready |
| Dépendances | Aucune |
| Bloc | RAP-1.1, RAP-1.2, RAP-1.3, RAP-1.4 |

**Description:**
- Remplacer service MariaDB par PostgreSQL 15+ dans `docker-compose.yml`
- Mettre à jour connection string (dialecte SQLAlchemy: `postgresql+psycopg2://`)
- Configurer PostgreSQL env vars (POSTGRES_DB, POSTGRES_USER, POSTGRES_PASSWORD)
- Ajouter healthcheck PostgreSQL dans compose
- Vérifier volumes persistants PostgreSQL
- Supprimer toutes références MariaDB du projet (docs, configs, code)
- Mettre à jour annuaire-fastapi avec driver PostgreSQL (psycopg2)

**Livrable:**
- `docker-compose.yml` (service PostgreSQL)
- `.env` ou variables d'environnement
- Tests: connexion PostgreSQL OK, healthcheck validé
- Vérifier annuaire-fastapi démarre sans erreur

---

### RAP-1.1: Database Models + ORM (RelationshipProfile, Interaction, RelationshipAction)

| Métrique | Valeur |
|----------|--------|
| Impact | 10 |
| Confidence | 100% |
| Effort | 3 |
| **ICE Score** | **33.3** |
| Priorité | 🔴 P0 |
| Status | 🟡 Blocked |
| Dépendances | RAP-1.0 |
| Bloc | RAP-1.2, RAP-1.3, RAP-1.4 |

**Description:**
- Ajouter 3 entités SQLAlchemy: RelationshipProfileDB, InteractionDB, RelationshipActionDB
- Ajouter Pydantic schemas (Create, Update, Response pour chaque)
- Définir Enums: relationship_type, proximity_level, business_potential, interaction_type, action_type, action_status, priority
- Configurer cascades `ondelete="CASCADE"` sur toutes ForeignKeys
- Tester migration schema avec PostgreSQL

**Livrable:**
- `models.py` (extend ~200 lignes)
- Tests: CREATE table, FK constraints, ENUM values (PostgreSQL compatible)

---

### RAP-1.2: API Endpoints — Relationship Profile (3 routes)

| Métrique | Valeur |
|----------|--------|
| Impact | 9 |
| Confidence | 100% |
| Effort | 2 |
| **ICE Score** | **45.0** |
| Priorité | 🔴 P0 |
| Status | 🟡 Blocked |
| Dépendances | RAP-1.1 |
| Bloc | RAP-1.5 |

**Routes:**
- `POST /contacts/{contact_id}/relationship-profile` → 201 Created
- `GET /contacts/{contact_id}/relationship-profile` → 200 / 404
- `PATCH /contacts/{contact_id}/relationship-profile` → 200 / 404

**Validation:**
- contact_id existe (FK check)
- Un seul profil par contact (unique constraint)
- relationship_type in enum
- trust_level 1-5

**Livrable:**
- Endpoints dans `main.py` (~50 lignes)
- Erreurs 400/404/409 appropriées

---

### RAP-1.3: API Endpoints — Interactions (2 routes)

| Métrique | Valeur |
|----------|--------|
| Impact | 8 |
| Confidence | 100% |
| Effort | 2 |
| **ICE Score** | **40.0** |
| Priorité | 🔴 P0 |
| Status | 🟡 Blocked |
| Dépendances | RAP-1.1 |
| Bloc | RAP-1.5 |

**Routes:**
- `POST /contacts/{contact_id}/interactions` → 201 Created
- `GET /contacts/{contact_id}/interactions` → 200 (list, paginated)

**Paramètres query GET:**
- `?skip=0&limit=100`
- `?type=email` (filter by interaction_type)
- `?since=2026-01-01` (filter by date)

**Livrable:**
- Endpoints dans `main.py` (~40 lignes)
- Pagination support

---

### RAP-1.4: API Endpoints — Relationship Actions (5 routes)

| Métrique | Valeur |
|----------|--------|
| Impact | 10 |
| Confidence | 100% |
| Effort | 3 |
| **ICE Score** | **33.3** |
| Priorité | 🔴 P0 |
| Status | 🟡 Blocked |
| Dépendances | RAP-1.1 |
| Bloc | RAP-1.5 |

**Routes:**
- `POST /contacts/{contact_id}/relationship-actions` → 201 Created
- `GET /relationship-actions` → 200 (list, queryable: status, priority, contact_id)
- `GET /relationship-actions/due` → 200 (today + overdue)
- `PATCH /relationship-actions/{action_id}` → 200 (update any field)
- `PATCH /relationship-actions/{action_id}/complete` → 200 (mark done + set completed_at)

**Paramètres query GET:**
- `?status=todo` (filter)
- `?priority=high` (filter)
- `?contact_id=xxx` (filter)

**Livrable:**
- Endpoints dans `main.py` (~80 lignes)
- Query filtering logic

---

### RAP-1.5: API Endpoint — RAP Dashboard

| Métrique | Valeur |
|----------|--------|
| Impact | 9 |
| Confidence | 95% |
| Effort | 2 |
| **ICE Score** | **42.75** |
| Priorité | 🔴 P0 |
| Status | 🟡 Blocked |
| Dépendances | RAP-1.2, RAP-1.3, RAP-1.4 |
| Bloc | Aucune |

**Route:**
- `GET /rap/dashboard` → 200

**Response Model:**
```json
{
  "due_today": 5,
  "overdue": 2,
  "active_relations": 12,
  "high_potential": 8,
  "recent_interactions": 23,
  "timestamp": "2026-05-29T14:30:00Z"
}
```

**Aggregations:**
- due_today: COUNT(actions) WHERE due_date = TODAY AND status = 'todo'
- overdue: COUNT(actions) WHERE due_date < TODAY AND status = 'todo'
- active_relations: COUNT(DISTINCT contact_id) WHERE proximity_level IN ('warm', 'active', 'close')
- high_potential: COUNT(DISTINCT contact_id) WHERE business_potential = 'high'
- recent_interactions: COUNT(interactions) WHERE interaction_date > TODAY - 7 days

**Livrable:**
- Endpoint dans `main.py` (~40 lignes)
- SQL aggregations optimisées

---

### RAP-1.6: Tests d'Intégration (pytest)

| Métrique | Valeur |
|----------|--------|
| Impact | 8 |
| Confidence | 90% |
| Effort | 4 |
| **ICE Score** | **18.0** |
| Priorité | 🟡 P1 |
| Status | 🟡 Blocked |
| Dépendances | RAP-1.1-1.5 |
| Bloc | Aucune |

**Tests à couvrir:**
- CRUD RelationshipProfile (201, 200, 404, 400)
- CRUD Interactions (201, 200, 404)
- CRUD RelationshipActions (201, 200, 404, complete flow)
- DELETE Contact → cascade check (profile, interactions, actions gone)
- Dashboard aggregations (counts correct)
- Invalid contact_id → 404
- Duplicate profile → conflict

**Livrable:**
- `test_rap.py` (pytest fixtures + ~50 test cases)
- conftest.py (DB fixtures PostgreSQL)

---

### RAP-1.7: Documentation API (OpenAPI + Exemples)

| Métrique | Valeur |
|----------|--------|
| Impact | 6 |
| Confidence | 100% |
| Effort | 2 |
| **ICE Score** | **30.0** |
| Priorité | 🟡 P1 |
| Status | 🟡 Blocked |
| Dépendances | RAP-1.1-1.5 |
| Bloc | Aucune |

**Contenu:**
- FastAPI auto-génère Swagger (via `/docs`)
- Ajouter docstrings pour chaque endpoint
- Exemples JSON pour chaque route
- Documenter enums + validations
- Schema descriptions

**Livrable:**
- Docstrings dans `main.py`
- Swagger auto-generated

---

## Backlog Maintenance (Existant)

### MAINT-1: Clarifier `data/` folder

| Métrique | Valeur |
|----------|--------|
| Impact | 5 |
| Confidence | 60% |
| Effort | 1 |
| **ICE Score** | **3.0** |
| Priorité | 🟢 P2 |
| Status | 🔴 Ready |
| Dépendances | Aucune |
| Bloc | Aucune |

**Description:**
- Déterminer si `data/` = volumes Docker persistants
- Vérifier utilisation dans docker-compose.yml
- Documenter ou supprimer

**Livrable:**
- Clarification dans CLAUDE.md ou suppression du dossier

---

### MAINT-2: Nettoyer `mcp-fast-mcp/scripts/` (dossier vide)

| Métrique | Valeur |
|----------|--------|
| Impact | 3 |
| Confidence | 100% |
| Effort | 1 |
| **ICE Score** | **3.0** |
| Priorité | 🟢 P2 |
| Status | 🔴 Ready |
| Dépendances | Aucune |
| Bloc | Aucune |

**Description:**
- Documenter usage si dossier intentionnel
- Supprimer si dossier inutile

---

### MAINT-3: Network Isolation (Backend ↔ MCP)

| Métrique | Valeur |
|----------|--------|
| Impact | 6 |
| Confidence | 80% |
| Effort | 2 |
| **ICE Score** | **24.0** |
| Priorité | 🟡 P1 |
| Status | 🔴 Ready |
| Dépendances | Aucune |
| Bloc | Aucune |

**Description:**
- Clarifier réseau Docker dans docker-compose.yml
- Backend et MCP sur même network ou isolés?
- Documenter pattern de communication intra-compose

**Livrable:**
- docker-compose.yml commenté (networks section)
- Docs/ARCHITECTURE.md update

---

### MAINT-4: Production TLS (Remplacer mkcert)

| Métrique | Valeur |
|----------|--------|
| Impact | 7 |
| Confidence | 85% |
| Effort | 3 |
| **ICE Score** | **19.85** |
| Priorité | 🟡 P1 |
| Status | 🔴 Ready |
| Dépendances | Aucune |
| Bloc | Aucune |

**Description:**
- Remplacer certificats mkcert (dev) par Let's Encrypt ou CA interne
- Configurer Nginx pour prod TLS
- Documenter rotation certificats

**Livrable:**
- TLS config docs
- Scripts de rotation certificats

---

## Phase 2: Enhancements (Post-MVP)

### ENH-2.1: Analytics Dashboard (Graphiques)

| Métrique | Valeur |
|----------|--------|
| Impact | 7 |
| Confidence | 70% |
| Effort | 5 |
| **ICE Score** | **9.8** |
| Priorité | 🟡 P1 |
| Status | 🔴 Ready |
| Dépendances | RAP-1.5 (Dashboard API) |
| Bloc | Aucune |

**Description:**
- Ajouter graphiques au dashboard (Plotly/ChartJS)
- Tendances actions (todo/done/overdue par semaine)
- Distribution relations par type
- Timeline interactions

**Livrable:**
- Frontend charts components

---

### ENH-2.2: Bulk Import/Export (CSV)

| Métrique | Valeur |
|----------|--------|
| Impact | 6 |
| Confidence | 75% |
| Effort | 4 |
| **ICE Score** | **11.25** |
| Priorité | 🟡 P1 |
| Status | 🔴 Ready |
| Dépendances | RAP-1.1-1.5 |
| Bloc | Aucune |

**Description:**
- Endpoints POST/GET pour import/export CSV
- Contacts + relationships + interactions + actions
- Validation bulk insert

**Routes:**
- `POST /bulk/import` (multipart CSV)
- `GET /bulk/export` (stream CSV)

**Livrable:**
- Endpoints dans `main.py`
- CSV parsing logic

---

### ENH-2.3: Notification System (Email)

| Métrique | Valeur |
|----------|--------|
| Impact | 8 |
| Confidence | 60% |
| Effort | 5 |
| **ICE Score** | **9.6** |
| Priorité | 🟡 P1 |
| Status | 🔴 Ready |
| Dépendances | RAP-1.4 (Actions API) |
| Bloc | Aucune |

**Description:**
- Alertes email pour actions dues
- Notifs journalières (due_today, overdue)
- Configuration SMTP

**Livrable:**
- Background tasks (Celery ou APScheduler)
- Email templates

---

## Résumé par Priorité

### 🔴 P0 (MVP — À faire IMMÉDIATEMENT)

| Task | ICE | Effort | Status |
|------|-----|--------|--------|
| RAP-1.0 PostgreSQL Migration | 31.67 | 3j | 🔴 Ready |
| RAP-1.1 Models | 33.3 | 3j | 🟡 Blocked (→1.0) |
| RAP-1.2 Profile API | 45.0 | 2j | 🟡 Blocked (→1.1) |
| RAP-1.3 Interactions API | 40.0 | 2j | 🟡 Blocked (→1.1) |
| RAP-1.4 Actions API | 33.3 | 3j | 🟡 Blocked (→1.1) |
| RAP-1.5 Dashboard | 42.75 | 2j | 🟡 Blocked (→1.2-4) |

**Σ Effort:** 15 jours | **Σ ICE:** 226.02 | **Timeline:** 3-4 semaines

---

### 🟡 P1 (Post-MVP — Après livraison P0)

| Task | ICE | Effort | Status |
|------|-----|--------|--------|
| RAP-1.6 Tests | 18.0 | 4j | 🟡 Blocked (→1.1-5) |
| RAP-1.7 Docs | 30.0 | 2j | 🟡 Blocked (→1.1-5) |
| MAINT-3 Network | 24.0 | 2j | 🔴 Ready |
| MAINT-4 TLS | 19.85 | 3j | 🔴 Ready |
| ENH-2.1 Analytics | 9.8 | 5j | 🔴 Ready (→RAP-1.5) |
| ENH-2.2 Bulk | 11.25 | 4j | 🔴 Ready (→RAP-1.1-5) |
| ENH-2.3 Notifications | 9.6 | 5j | 🔴 Ready (→RAP-1.4) |

**Σ Effort:** 25 jours | **Σ ICE:** 122.5

---

### 🟢 P2 (Backlog Indéfini — À ignorer pour MVP)

| Task | ICE | Effort | Status |
|------|-----|--------|--------|
| MAINT-1 data/ | 3.0 | 1j | 🔴 Ready |
| MAINT-2 scripts/ | 3.0 | 1j | 🔴 Ready |

**Σ Effort:** 2 jours | **Σ ICE:** 6.0

---

## Ordre d'Exécution (Dépendances)

```
Phase 1 MVP (P0):
  1. RAP-1.0 PostgreSQL Migration ✓ (débloque tout)
     ↓
  2. RAP-1.1 Models ✓ (débloque endpoints)
     ↓
  3. RAP-1.2 Profile API
     RAP-1.3 Interactions API (parallèle)
     RAP-1.4 Actions API
     ↓
  4. RAP-1.5 Dashboard (agrège les 3)
     ↓
  5. RAP-1.6 Tests (validation)
  6. RAP-1.7 Docs (auto-gen)

Phase 2 (P1 + P2):
  7. MAINT-3, MAINT-4 (ops/security)
  8. ENH-2.1, 2.2, 2.3 (features)
  9. MAINT-1, 2 (cleanup)
```

---

## Checklist Livraison MVP (P0)

- [ ] Task RAP-1.0 terminée (PostgreSQL running, no MariaDB references)
- [ ] Task RAP-1.1 terminée (models + enums + migrations PostgreSQL)
- [ ] Tasks RAP-1.2-1.4 terminées (11 endpoints)
- [ ] Task RAP-1.5 terminée (dashboard aggregations)
- [ ] Task RAP-1.6 terminée (tests passent)
- [ ] Task RAP-1.7 terminée (Swagger OK)
- [ ] Cascade DELETE fonctionne (delete contact → clean all children)
- [ ] Tous les endpoints retournent les codes HTTP attendus
- [ ] PostgreSQL schema match ORM models
- [ ] Docker `compose up` démarre sans erreur
- [ ] Endpoints accessibles sur ports attendus
- [ ] Zéro références MariaDB dans le code

---

## Historique

- **2026-05-29:** Ajout RAP-1.0 PostgreSQL Migration — remplacement total MariaDB → PostgreSQL
- **2026-05-29:** Création MASTER-BACKLOG.md (consolidation RAP-BACKLOG.md + CHANGELOG TODO + MAINT + ENH)
- **2026-05-29:** RAP-DIAGRAMS.md (diagrammes architecture)
- **2026-05-29:** RAP-BACKLOG.md → déprécié (migrer vers MASTER-BACKLOG.md)

---

**Généré:** 2026-05-29  
**Format:** Markdown centralisé  
**Prochaine Review:** Après livraison P0 (2026-06-15 estimé)  
**Owner:** Teddy (tjeanbaptiste@dyweb.fr)
