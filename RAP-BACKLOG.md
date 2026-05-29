# RAP System Backlog (avec Scoring ICE)

## Méthodologie Scoring

**ICE = Impact × Confidence / Effort**

- **Impact (1-10):** Valeur pour l'utilisateur (0=none, 10=critical)
- **Confidence (0-100%):** Certitude qu'on peut livrer la solution
- **Effort (1-10):** Charge de travail (1=trivial, 10=massive)

**Score ICE** = (Impact × Confidence) / Effort

→ **Seuil MVP:** ICE ≥ 5.0  
→ **Seuil P0:** ICE ≥ 8.0

---

## Phase 1: RAP System Core (MVP)

### Task 1.1: DB Models + ORM (RelationshipProfile, Interaction, RelationshipAction)

| Métrique | Score |
|----------|-------|
| Impact | 10 |
| Confidence | 100% |
| Effort | 3 |
| **ICE Score** | **33.3** |

**Description:**
- Ajouter 3 entités ORM (SQLAlchemy) dans `models.py`
- Ajouter Pydantic schemas (Create, Update, Response)
- Ajouter Enums (relationship_type, interaction_type, action_type, etc.)
- Vérifier cascades `ondelete="CASCADE"`

**Dépendances:** Aucune  
**Bloc:** Task 1.2, 1.3, 1.4  
**Priorité:** 🔴 P0 (dépendance critique)

---

### Task 1.2: API Endpoints - Relationship Profile (3 routes)

| Métrique | Score |
|----------|-------|
| Impact | 9 |
| Confidence | 100% |
| Effort | 2 |
| **ICE Score** | **45.0** |

**Description:**
- `POST /contacts/{cid}/relationship-profile` (201)
- `GET /contacts/{cid}/relationship-profile` (200/404)
- `PATCH /contacts/{cid}/relationship-profile` (200/404)

**Dépendances:** Task 1.1  
**Bloc:** Task 1.5  
**Priorité:** 🔴 P0

---

### Task 1.3: API Endpoints - Interactions (2 routes)

| Métrique | Score |
|----------|-------|
| Impact | 8 |
| Confidence | 100% |
| Effort | 2 |
| **ICE Score** | **40.0** |

**Description:**
- `POST /contacts/{cid}/interactions` (201)
- `GET /contacts/{cid}/interactions` (200, paginated)

**Dépendances:** Task 1.1  
**Bloc:** Task 1.5  
**Priorité:** 🔴 P0

---

### Task 1.4: API Endpoints - Relationship Actions (5 routes)

| Métrique | Score |
|----------|-------|
| Impact | 10 |
| Confidence | 100% |
| Effort | 3 |
| **ICE Score** | **33.3** |

**Description:**
- `POST /contacts/{cid}/relationship-actions` (201)
- `GET /relationship-actions` (200, queryable: status, priority)
- `GET /relationship-actions/due` (200, today + overdue)
- `PATCH /relationship-actions/{aid}` (200, update)
- `PATCH /relationship-actions/{aid}/complete` (200, mark done)

**Dépendances:** Task 1.1  
**Bloc:** Task 1.5  
**Priorité:** 🔴 P0

---

### Task 1.5: API Endpoint - RAP Dashboard

| Métrique | Score |
|----------|-------|
| Impact | 9 |
| Confidence | 95% |
| Effort | 2 |
| **ICE Score** | **42.75** |

**Description:**
- `GET /rap/dashboard` (200)
- Retourner: due_today, overdue, active_relations, high_potential, recent_interactions

**Dépendances:** Task 1.2, 1.3, 1.4  
**Bloc:** Aucune  
**Priorité:** 🔴 P0

---

### Task 1.6: Tests d'Intégration (API + DB)

| Métrique | Score |
|----------|-------|
| Impact | 8 |
| Confidence | 90% |
| Effort | 4 |
| **ICE Score** | **18.0** |

**Description:**
- `test_rap.py` avec pytest
- CRUD tests pour chaque entité
- Cascade delete test
- Dashboard metrics test

**Dépendances:** Task 1.1-1.5  
**Bloc:** Aucune  
**Priorité:** 🟡 P1 (non-bloquant)

---

### Task 1.7: Documentation API (OpenAPI + exemples)

| Métrique | Score |
|----------|-------|
| Impact | 6 |
| Confidence | 100% |
| Effort | 2 |
| **ICE Score** | **30.0** |

**Description:**
- FastAPI auto-génère OpenAPI (Swagger)
- Ajouter exemples JSON pour chaque endpoint
- Documenter enums + validations

**Dépendances:** Task 1.1-1.5  
**Bloc:** Aucune  
**Priorité:** 🟡 P1 (documentation)

---

## Backlog Existant (Maintenance)

### Maintenance 1: Clarifier `data/` folder

| Métrique | Score |
|----------|-------|
| Impact | 5 |
| Confidence | 60% |
| Effort | 1 |
| **ICE Score** | **3.0** |

**Description:** Déterminer si `data/` = volumes Docker ou supprimable  
**Priorité:** 🟢 P2 (low-priority cleanup)

---

### Maintenance 2: Clean `mcp-fast-mcp/scripts/`

| Métrique | Score |
|----------|-------|
| Impact | 3 |
| Confidence | 100% |
| Effort | 1 |
| **ICE Score** | **3.0** |

**Description:** Documenter ou supprimer dossier vide  
**Priorité:** 🟢 P2 (cosmetic)

---

### Maintenance 3: Network Isolation

| Métrique | Score |
|----------|-------|
| Impact | 6 |
| Confidence | 80% |
| Effort | 2 |
| **ICE Score** | **24.0** |

**Description:** Clarifier réseau Backend ↔ MCP dans docker-compose  
**Priorité:** 🟡 P1 (ops)

---

### Maintenance 4: Production TLS

| Métrique | Score |
|----------|-------|
| Impact | 7 |
| Confidence | 85% |
| Effort | 3 |
| **ICE Score** | **19.85** |

**Description:** Remplacer mkcert (dev) par Let's Encrypt ou CA interne  
**Priorité:** 🟡 P1 (pre-prod)

---

## Phase 2: Enhancements (Post-MVP)

### Enhancement 2.1: Analytics Dashboard (Graphiques)

| Métrique | Score |
|----------|-------|
| Impact | 7 |
| Confidence | 70% |
| Effort | 5 |
| **ICE Score** | **9.8** |

**Description:** Ajouter graphiques (Plotly/ChartJS) au dashboard  
**Dépendances:** Task 1.5  
**Priorité:** 🟡 P1 (post-MVP)

---

### Enhancement 2.2: Bulk Actions (Import/Export)

| Métrique | Score |
|----------|-------|
| Impact | 6 |
| Confidence | 75% |
| Effort | 4 |
| **ICE Score** | **11.25** |

**Description:** Importer/exporter contacts + relations en CSV  
**Priorité:** 🟡 P1 (post-MVP)

---

### Enhancement 2.3: Notification System

| Métrique | Score |
|----------|-------|
| Impact | 8 |
| Confidence | 60% |
| Effort | 5 |
| **ICE Score** | **9.6** |

**Description:** Alertes email pour actions dues  
**Dépendances:** Task 1.4  
**Priorité:** 🟡 P1 (post-MVP)

---

## Résumé par Phase

### Phase 1: RAP Core (MVP) — Livrable

| Task | ICE | Priorité | Status |
|------|-----|----------|--------|
| 1.1 Models | 33.3 | 🔴 P0 | ⏳ Ready |
| 1.2 Profile API | 45.0 | 🔴 P0 | ⏳ Ready |
| 1.3 Interactions API | 40.0 | 🔴 P0 | ⏳ Ready |
| 1.4 Actions API | 33.3 | 🔴 P0 | ⏳ Ready |
| 1.5 Dashboard | 42.75 | 🔴 P0 | ⏳ Ready |
| 1.6 Tests | 18.0 | 🟡 P1 | ⏳ Post-code |
| 1.7 Docs | 30.0 | 🟡 P1 | ⏳ Auto-gen |

**Total ICE (P0):** ~224.3  
**Effort (P0):** ~13 jours-dev  
**Timeline:** ~2-3 semaines

---

### Backlog: Maintenance + Phase 2

| Task | ICE | Priorité | Status |
|------|-----|----------|--------|
| M1 data/ | 3.0 | 🟢 P2 | 🔴 Blocked |
| M2 scripts/ | 3.0 | 🟢 P2 | 🔴 Blocked |
| M3 Network | 24.0 | 🟡 P1 | ⏳ Ready |
| M4 TLS | 19.85 | 🟡 P1 | ⏳ Ready |
| E2.1 Analytics | 9.8 | 🟡 P1 | ⏳ Post-MVP |
| E2.2 Bulk | 11.25 | 🟡 P1 | ⏳ Post-MVP |
| E2.3 Notifications | 9.6 | 🟡 P1 | ⏳ Post-MVP |

---

## Ordre Recommandé (Dépendances)

```
1. Task 1.1 (Models) ← dépendance bloquante
   ↓
2. Tasks 1.2, 1.3, 1.4 (API Endpoints) ← parallélisables
   ↓
3. Task 1.5 (Dashboard) ← agrège les 3
   ↓
4. Task 1.6 (Tests) ← validation finale
5. Task 1.7 (Docs) ← auto-généré via Swagger
```

**Démarrage immédiat:** Task 1.1 (Models)

---

## Critères de Livraison Phase 1

- ✅ 3 modèles ORM + 3 schemas Pydantic créés
- ✅ 11 endpoints implémentés (Profile 3 + Interactions 2 + Actions 5 + Dashboard 1)
- ✅ Cascades DELETE fonctionnelles
- ✅ Tests unitaires passent (Task 1.6)
- ✅ Swagger OpenAPI documenté
- ✅ Intégration Backend → annuaire-fastapi ✅

---

**Généré:** 2026-05-29  
**Review Date:** Après livraison Phase 1  
**Prochaine Review:** 2026-06-15
