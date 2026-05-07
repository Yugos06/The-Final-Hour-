# Phase 1: Foundation - Context

**Gathered:** 2026-05-07
**Status:** Ready for planning

<domain>
## Phase Boundary

Phase 1 livre l'infrastructure de chargement : le mod se lance proprement dans HoI4, le bookmark 2026 est sélectionnable, tous les sprites GFX résolvent vers leurs fichiers .dds sans pink squares. Aucun gameplay n'est implémenté — c'est la fondation sur laquelle toutes les phases suivantes s'appuient.

</domain>

<decisions>
## Implementation Decisions

### Assets GFX (.dds)

- **D-01:** Les fichiers .dds existent localement hors du repo git — ils doivent être copiés dans le repo lors de la Phase 1.
- **D-02:** Structure de dossiers standard HoI4 :
  - `interface/goals/` — icônes de focus (74×74 px, DXT5)
  - `interface/ideas/` — icônes d'ideas/spirits (94×93 px)
  - `interface/eventpictures/` — images d'événements (600×240 px)
- **D-03:** Tous les sprites GFX déclarés en Phase 1 (GFX-01 à GFX-04 — focus, ideas, événements). Pas de report aux phases suivantes. Zéro pink square dès le premier chargement.

### Code alpha hérité

- **D-04:** Supprimer entièrement le code avec préfixe `fh_fra_` dès Phase 1 : `common/national_focus/fh_france_focus.txt`, `common/ideas/fh_france_ideas.txt`, `common/decisions/fh_france_decisions.txt`, `common/decisions/categories/fh_france_decision_categories.txt`, `common/on_actions/fh_on_actions.txt`. Table rase complète.
- **D-05:** Créer immédiatement les fichiers squelettes avec les nouveaux noms et headers minimaux valides :
  - `common/national_focus/FRA_focus.txt` (focus_tree vide mais valide)
  - `common/ideas/FRA_ideas.txt` (ideas block vide)
  - `events/FRA_events.txt` (namespace + commentaire placeholder)
  - `common/on_actions/FRA_on_actions.txt` (on_actions block vide)
  Le mod doit charger sans erreurs de syntaxe après Phase 1.

### Shadow file — arbre de focus vanilla France

- **D-06:** Stratégie par nation avec facteur de priorité — `FRA_focus.txt` déclare `country = { tag = FRA factor = 10 }` pour surpasser l'arbre vanilla France. **Pas de `replace_path = "common/national_focus"`** en Phase 1 — les autres nations gardent leurs arbres vanilla intacts.
- **D-07:** Modèle évolutif : chaque nation reçoit son fichier quand son contenu est prêt. Pour les nations non encore travaillées, l'arbre vanilla reste actif. Des arbres "intermédiaires" pourront être ajoutés si une nation vanilla est trop incompatible avec le scénario 2026.

### Bookmark 2026

- **D-08:** GER reste visible dans le bookmark avec une description "En développement" — le joueur voit que la nation arrive, le mod est honnête sur son état alpha.
- **D-09:** Image de fond vanilla (`GFX_select_date_1936`) conservée en Phase 1. Elle sera remplacée par un visuel custom quand les assets correspondants seront intégrés (phase ou itération ultérieure).

### VS Code / Workspace

- **D-10:** Configurer `.vscode/settings.json` avec `"files.encoding": "utf8bom"` pour les `.yml` — obligatoire pour que HoI4 lise correctement les fichiers de localisation. Vérification BOM (`ef bb bf`) sur tout nouveau fichier yml.

</decisions>

<canonical_refs>
## Canonical References

**Les agents en aval DOIVENT lire ces fichiers avant de planifier ou d'implémenter.**

### Roadmap & Requirements
- `.planning/ROADMAP.md` §Phase 1 — Goal, Requirements (FOUND-01/02/03, GFX-01/02/03/04), Success Criteria
- `.planning/REQUIREMENTS.md` §Foundation & Infrastructure, §GFX / Assets Visuels — specs exactes des requirements Phase 1

### Design & Architecture
- `PLAN.md` — Design complet du mod (structure fichiers, nommage, contenu focus tree) — référence pour comprendre où Phase 1 s'insère
- `CLAUDE.md` — Conventions de nommage (`FRA_` prefix), contraintes HoI4, structure des fichiers mod

### Fichiers existants à supprimer/remplacer
- `common/national_focus/fh_france_focus.txt` — ancien code alpha à supprimer
- `common/ideas/fh_france_ideas.txt` — ancien code alpha à supprimer
- `common/decisions/fh_france_decisions.txt` — ancien code alpha à supprimer
- `common/on_actions/fh_on_actions.txt` — ancien code alpha à supprimer
- `common/bookmarks/fh_2026.txt` — à mettre à jour (description narrative, GER WIP mention)
- `descriptor.mod` — à nettoyer (entrées dupliquées)

</canonical_refs>

<code_context>
## Existing Code Insights

### Patterns établis
- `descriptor.mod` utilise déjà `replace_path="common/bookmarks"` — le bookmark mod remplace complètement le vanilla. Pattern à ne pas changer.
- `common/defines/00_fh_defines.lua` — définit la timeline 2026–2036, à conserver tel quel.
- Préfixe `FRA_` est la convention décidée (vs l'ancien `fh_fra_`) — cohérent avec PLAN.md et le standard HoI4.

### Points d'intégration
- `interface/fh_goals.gfx` (à créer) — déclarera les `GFX_focus_FRA_*` sprites référencés par `FRA_focus.txt` en Phase 5
- `interface/fh_ideas.gfx` (à créer) — déclarera les `GFX_idea_FRA_*` sprites référencés par `FRA_ideas.txt` en Phase 2
- Les fichiers squelettes créés en Phase 1 seront complétés par les phases suivantes sans renommage

### Contraintes HoI4
- Fichiers localisation `.yml` : UTF-8 avec BOM obligatoire, commencer par `l_french:` ou `l_english:`
- Focus tree : `cost = 10` = 70 jours en jeu
- DDS format : DXT5 pour les icônes avec transparence

</code_context>

<specifics>
## Specific Ideas

- L'utilisateur envisage un **grand mod multi-nations** — la stratégie de shadow par nation (D-06/D-07) est pensée pour scaler proprement sans tout casser à chaque nouvelle nation.
- Pour les nations non encore travaillées, prévoir des arbres "transitoires" si le scénario 2026 rend l'arbre vanilla trop incohérent (à évaluer au cas par cas, pas en Phase 1).
- Inspirations visuelles : **The Fire Rises** (icônes cinématiques, style propagande) et **Road to 56** (profondeur de branches). Les assets GFX existants doivent refléter cette DA.

</specifics>

<deferred>
## Deferred Ideas

- Arbres de focus "transitoires" pour les nations non travaillées (GER, etc.) — à évaluer lors de la phase GER, pas en Phase 1.
- Image de fond custom pour le bookmark 2026 — différée jusqu'à ce que l'asset correspondant soit identifié dans les fichiers locaux.
- `replace_path = "common/national_focus"` — option envisageable quand 5+ nations auront du contenu.

</deferred>

---

*Phase: 1-Foundation*
*Context gathered: 2026-05-07*
