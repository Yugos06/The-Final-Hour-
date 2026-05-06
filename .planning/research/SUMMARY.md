# Research Summary — The Final Hour (HoI4 Mod)

## Recommended Stack

| Technologie | Usage | Note |
|---|---|---|
| Clausewitz Script `.txt` (UTF-8, sans BOM) | Toute la logique gameplay | Standard engine, stable depuis HoI4 1.9.x |
| Paradox YAML `.yml` (UTF-8 **avec** BOM) | Toutes les chaînes de texte | Le BOM est OBLIGATOIRE — sans lui, le fichier est silencieusement ignoré |
| DDS textures (DXT5/BC3, avec alpha) | Icônes focus 74x74px, ideas 94x93px | Seul format accepté par le moteur |
| VS Code + extension CWTools | Éditeur avec vérification syntaxe HoI4 | Standard communautaire |
| GIMP 2.10+ (plugin DDS intégré) | Authoring DDS | Export DXT5 correct |
| `error.log` + console HoI4 | Validation principale | `reload focus`, `reload localisation`, `event fh_france.10` |

**Version cible :** `supported_version = "1.14.*"` — syntaxe stable, pas de changement attendu.

---

## Table Stakes Features

**Manquants dans l'alpha (bloqueurs v1) :**

| Feature | Statut | Priorité |
|---|---|---|
| `bypass` blocks sur tous les focus avec `available` | **ABSENT** — soft-lock IA | Critique |
| `cancel_if_invalid = yes` sur focus mutex | **ABSENT** — focus fantôme | Haute |
| GFX icônes custom intégrées | **ABSENT** — assets existent, non câblés | Haute |
| Mutex `mutually_exclusive` symétrique | **BUG** — `fh_fra_france_souveraine`/`fh_fra_rn_path` asymétrique | Haute |
| Localisation anglaise complète | **ABSENT** — fichier EN contient du FR | Haute |
| Supprimer `on_daily` spirit enforcement | **ANTI-PATTERN** — perf + bug | Moyenne |
| Trunk focus `y = -1` → `y = 0` | **BUG UX** — invisible au scroll initial | Moyenne |

**Différenciateurs (avis positifs) :**
- Localisation narrative ("pourquoi" pas seulement "quoi") — déjà bien dans le FR existant, maintenir ce niveau
- Décisions Assemblée Nationale exclusives par chemin (`visible = { has_completed_focus = ... }`)
- Unlock Fédération Européenne à 4 conditions simultanées
- Options d'événements qui posent des flags avec conséquences architecturales

---

## Ordre de Développement Recommandé

Basé sur le graphe de dépendances (chaque couche ne peut référencer que les couches au-dessus) :

**Couche 1 — Fondation (pas de dépendances) :**
1. `common/defines/00_fh_defines.lua` — déjà fait
2. `common/decisions/categories/fh_france_decision_categories.txt` — manquant = crash
3. `common/ideas/fh_france_ideas.txt` — les IDs doivent exister avant tout
4. `interface/fh_goals.gfx`, `fh_ideas.gfx` — déclarations GFX avant toute référence
5. `gfx/interface/goals/*.dds` — fichiers textures physiques
6. **`common/national_focus/00_france.txt` (fichier vide)** — shadow file obligatoire avec préfixe `FRA_`
7. Squelette localisation (`.yml` avec BOM) — remplir en parallèle

**Couche 2 — Logique :**

8. `events/fh_france_events.txt` — référence les IDs ideas et GFX
9. `common/on_actions/fh_on_actions.txt` — référence les IDs ideas

**Couche 3 — Intégration :**

10. `common/decisions/fh_france_decisions.txt` — référence events + catégories
11. `common/national_focus/fh_france_focus.txt` — référence tout ; écrit en dernier
12. `common/bookmarks/fh_2026.txt` — tags pays et clés localisation

---

## Top 5 Pièges (par priorité)

**1. `bypass` blocks absents** — sans eux, l'IA soft-lock sur les focus avec `available`. Ajouter `bypass = { always = yes }` ou une condition de fallback sur chaque focus qui a un bloc `available`. Impact le plus élevé pour la correction v1.

**2. Collision préfixe `FRA_` avec le jeu de base** — vanilla livre des IDs comme `FRA_the_maginot_line`. Solution obligatoire : créer un fichier VIDE `common/national_focus/00_france.txt` dans le mod pour shadow le fichier vanilla. Sans ça, comportement imprévisible. Ne PAS utiliser `replace_path`.

**3. Bug asymétrie mutex** — `fh_fra_france_souveraine` ne déclare pas `mutually_exclusive` vers `fh_fra_rn_path`. Le joueur peut compléter les deux chemins. Audit complet après écriture de tous les focus.

**4. BOM sur les fichiers localisation** — actuellement correct dans l'alpha. DOIT être maintenu. Configurer VS Code : `"files.encoding": "utf8bom"` pour `*.yml`. Vérifier avec `hexdump -C fichier.yml | head -1` → premiers bytes doivent être `ef bb bf`.

**5. `on_daily` anti-pattern** — le bloc `on_daily` actuel ré-applique `FRA_esprit_francais` chaque jour pour chaque pays. Remplacer par `on_startup` + `on_ruling_party_change` + `on_civil_war_end`.

---

## Décisions Clés Avant de Démarrer

| Décision | Recommandation |
|---|---|
| Préfixe `FRA_` + shadow file | Utiliser `FRA_` + créer `00_france.txt` vide — PROJECT.md a déjà décidé, le shadow file est la pièce manquante |
| Trunk focus position | Déplacer `y = -1` → `y = 0` dans la reconstruction |
| `reset_on_civilwar` | Décider avant Phase 3 — `yes` peut sembler punitif avec la profondeur narrative |
| `guarantee_cost = -1.0` | Vérifier si ce modifier existe dans vanilla avant de reconstruire l'idea |
| Localisation anglaise | Écrire EN et FR simultanément dès le départ — ne JAMAIS battre à la fin |

---

## Conflits Résolus

**Conflit préfixe `FRA_` :** STACK.md l'endorse, PITFALLS.md signale le risque de collision. Les deux ont raison. **Résolution :** `FRA_` + shadow file `00_france.txt` vide. Le shadow file élimine entièrement le risque.

**Conflit `y = -1` :** STACK.md dit "pattern valide dans les grands mods". FEATURES.md + PITFALLS.md disent "bug visuel". **Résolution :** Déplacer à `y = 0` dans la reconstruction — inapproprié à cette échelle.

---

*Recherche complète — confiance : HAUTE. Tous les findings sont basés sur l'inspection directe des fichiers du projet.*
