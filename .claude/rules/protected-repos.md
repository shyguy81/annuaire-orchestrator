# Repos Techniques Protégés

**Référencé par [Règle 2](./annuaire-contacts-rules.md#règle-2-ne-jamais-modifier-le-code-des-repos-liés).**

Liste exhaustive des repos avec leur propre `.git`, jamais modifiables depuis `orchestrator/`.

| Repo | Rôle | Statut |
| --- | --- | --- |
| `annuaire-fastapi/` | Backend API (logique métier, PostgreSQL) | Actif |
| `annuaire-cli/` | CLI Rust (client généré + interface MCP stdio) | Actif |
| `mcp-fast-mcp/` | Serveur MCP (Nginx, TLS) | Actif — sera remplacé par `annuaire-mcp` |
| `cli-llm/` | Espace de test `annuaire-cli` (sessions Claude Code) | Actif |

**Portée identique pour les 4 repos:**

- ❌ Éditer leur code depuis `orchestrator/`
- ❌ `git commit`/`git push` depuis `orchestrator/` (pas de `.git` de ces repos ici)
- ✅ Lire leur code depuis `orchestrator/` (diagnostic, doc, exécuter leurs scripts existants)
- ✅ Exécuter des scripts déjà présents dans ces repos (ex: `annuaire-cli/scripts/generate-client.sh`) — l'exécution n'est pas une modification, mais tout **résultat modifié** (fichiers générés, code source) se commit **depuis le repo concerné**, jamais depuis `orchestrator/`

**Comment modifier un de ces repos:**

```bash
cd ../annuaire-cli          # ou annuaire-fastapi, mcp-fast-mcp, cli-llm
# éditer le code ICI
git add <fichiers>
git commit -m "..."
git push
# revenir à orchestrator pour documenter si besoin
```

**Cas particulier — fichiers générés (Règle 3):**
Exécuter un générateur (ex: `generate-client.sh`) depuis `orchestrator/` est toléré pour diagnostic/vérification rapide, mais le commit du résultat (`shared/src/generated/*.rs`) doit se faire depuis `annuaire-cli/`, jamais depuis `orchestrator/`.

---

**Dernière mise à jour:** 2026-07-02
