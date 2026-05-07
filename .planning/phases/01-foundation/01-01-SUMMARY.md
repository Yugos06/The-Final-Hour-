---
phase: 01-foundation
plan: 01
subsystem: infra
tags: [hoi4-mod, clausewitz-script, focus-tree, shadow-file, skeleton]

requires: []
provides:
  - descriptor.mod cible HoI4 1.18.* (corrigé depuis 1.14.*)
  - Shadow file common/national_focus/france.txt ecrasant l'arbre vanilla french_focus
  - Squelette focus_tree FRA_focus_tree avec priorité FRA (factor=0 + modifier add=10)
  - Squelette ideas FRA vide (Phase 2 remplira)
  - Squelette on_actions FRA vide (Phase 3 remplira)
  - Namespace fh_france déclaré dans events/FRA_events.txt (Phase 3 remplira)
  - Suppression complète du code alpha fh_fra_ (6 fichiers)
affects:
  - 01-02 (bookmark et GFX — utilise les mêmes conventions FRA_)
  - 01-03 (localisation — s'appuie sur les IDs FRA_ établis)
  - Phase 2 (Ideas & Esprits Nationaux — complète FRA_ideas.txt)
  - Phase 3 (Events & on_actions — complète FRA_on_actions.txt + FRA_events.txt)
  - Phase 5 (Focus Tree — complète FRA_focus.txt)

tech-stack:
  added: []
  patterns:
    - "Shadow file HoI4 : même nom relatif que le vanilla (france.txt) pour override par chemin identique"
    - "Focus tree skeleton : factor=0 + modifier add=10 tag=FRA pour priorité correcte"
    - "Namespace fh_france conservé dans FRA_events.txt (pas de renommage inter-phases)"
    - "Tous les .txt Clausewitz en UTF-8 sans BOM (vérification hexdump obligatoire)"

key-files:
  created:
    - common/national_focus/france.txt
    - common/national_focus/FRA_focus.txt
    - common/ideas/FRA_ideas.txt
    - common/on_actions/FRA_on_actions.txt
    - events/FRA_events.txt
  modified:
    - descriptor.mod (supported_version 1.14.* -> 1.18.*)
  deleted:
    - common/national_focus/fh_france_focus.txt
    - common/ideas/fh_france_ideas.txt
    - common/decisions/fh_france_decisions.txt
    - common/decisions/categories/fh_france_decision_categories.txt
    - common/on_actions/fh_on_actions.txt
    - events/fh_france_events.txt

key-decisions:
  - "D-04 : Code alpha fh_fra_ supprimé entièrement — table rase pour reconstruction propre"
  - "D-05 : Squelettes FRA_focus.txt/FRA_ideas.txt/FRA_on_actions.txt/FRA_events.txt créés minimalement valides"
  - "D-06 : Stratégie factor=0 + modifier add=10 tag=FRA dans FRA_focus.txt (surpasse vanilla french_focus)"
  - "FOUND-01 : Shadow file france.txt avec id=french_focus et country{factor=0} efface l'arbre vanilla"
  - "Commentaires squelettes sans code Clausewitz actif (évite faux positifs dans grep-based acceptance criteria)"

patterns-established:
  - "Shadow override HoI4 : le fichier mod doit avoir EXACTEMENT le même chemin relatif que le vanilla"
  - "Squelette focus_tree HoI4 : id + country{factor/modifier} + default=no + reset_on_civilwar=yes est le minimum valide"
  - "Namespace events : fh_france est conservé comme namespace commun (D-05 implicite)"

requirements-completed:
  - FOUND-01

duration: 2min
completed: 2026-05-07
---

# Phase 1 Plan 01: Foundation — Reconstruction infrastructure scripts Summary

**Suppression du code alpha fh_fra_ (6 fichiers, 1571 lignes), correction version HoI4 1.14.*->1.18.*, shadow file france.txt + 4 squelettes FRA_ minimalement valides posant la fondation pour les Phases 2-5**

## Performance

- **Duration:** 2 min
- **Started:** 2026-05-07T15:49:34Z
- **Completed:** 2026-05-07T15:51:34Z
- **Tasks:** 3 / 3
- **Files modified:** 7 (1 modifié + 5 créés + 6 supprimés)

## Accomplishments

- Suppression propre des 6 fichiers alpha `fh_fra_*.txt` (1571 lignes de code alpha supprimées)
- `descriptor.mod` mis à jour de `1.14.*` vers `1.18.*` — le mod ne déclenchera plus l'avertissement "mod obsolète" dans le launcher Paradox
- Shadow file `common/national_focus/france.txt` créé : efface l'arbre vanilla `french_focus` par mécanisme d'override chemin identique HoI4
- `FRA_focus.txt` : squelette `FRA_focus_tree` avec priorité `factor=0 + modifier add=10 tag=FRA` — HoI4 sélectionnera cet arbre pour FRA (score 10 vs score 0 du shadow vanilla)
- 3 squelettes additionnels créés (ideas/on_actions/events) — syntaxiquement valides, prêts à être complétés par les Phases 2 et 3
- Encodage UTF-8 sans BOM vérifié par hexdump sur tous les fichiers

## Task Commits

1. **Task 1 : Supprimer le code alpha fh_fra_ et mettre à jour descriptor.mod** — `2f874a3` (chore)
2. **Task 2 : Créer le shadow file france.txt et le squelette FRA_focus.txt** — `5b304e3` (feat)
3. **Task 3 : Créer les squelettes FRA_ideas.txt, FRA_on_actions.txt et FRA_events.txt** — `e951885` (feat)

## Files Created/Modified

### Supprimés (code alpha D-04)
- `common/national_focus/fh_france_focus.txt` — arbre alpha fh_france_focus (615 lignes)
- `common/ideas/fh_france_ideas.txt` — esprits nationaux alpha
- `common/decisions/fh_france_decisions.txt` — décisions alpha Assemblée Nationale
- `common/decisions/categories/fh_france_decision_categories.txt` — catégories décisions alpha
- `common/on_actions/fh_on_actions.txt` — on_startup/on_daily avec références fh_fra_
- `events/fh_france_events.txt` — events alpha namespace fh_france

### Modifié
- `descriptor.mod` — `supported_version` : `"1.14.*"` → `"1.18.*"`

### Créés
- `common/national_focus/france.txt` — Shadow file : `focus_tree { id=french_focus, country{factor=0}, default=no }`
- `common/national_focus/FRA_focus.txt` — Squelette : `FRA_focus_tree` avec `factor=0+modifier{add=10,tag=FRA}`, `reset_on_civilwar=yes`, sans bloc focus
- `common/ideas/FRA_ideas.txt` — Squelette : `ideas { country { } }` (Phase 2)
- `common/on_actions/FRA_on_actions.txt` — Squelette : `on_actions { on_startup { effect { } } }` (Phase 3)
- `events/FRA_events.txt` — Squelette : `add_namespace = fh_france` (Phase 3)

## Decisions Made

- **D-04 appliquée :** Table rase complète du code alpha — 6 fichiers supprimés, aucun code migratable identifié (les patterns utiles sont documentés dans RESEARCH.md pour les phases suivantes)
- **D-05 appliquée :** Squelettes minimaux valides créés — commentaires ajustés pour ne pas contenir de code Clausewitz actif (évite faux positifs dans les tests d'acceptance par grep)
- **D-06 appliquée :** Stratégie `factor=0 + modifier add=10 tag=FRA` dans FRA_focus.txt — conforme au pattern vanilla vérifié
- **FOUND-01 appliquée :** Shadow file `france.txt` (PAS `00_france.txt`) — override correct par chemin relatif identique

## Deviations from Plan

### Auto-fixed Issues

**1. [Rule 2 - Correctness] Commentaires squelettes ajustés pour éviter faux positifs grep**
- **Found during:** Task 2 et Task 3 (vérification acceptance criteria)
- **Issue:** Les commentaires `# PHASE 5 : focus = { id = FRA_lheure_la_plus_sombre ... }` et `# PHASE 3 : country_event = { id = fh_france.001 ... }` contenaient les patterns `focus = {` et `country_event` que les acceptance criteria testaient à 0 occurrence
- **Fix:** Commentaires réécrits sans les patterns testés : `# PHASE 5 : ajouter le focus FRA_lheure_la_plus_sombre ici` et `# PHASE 3 : ajouter les events fh_france.001, fh_france.002, etc.`
- **Files modified:** `common/national_focus/FRA_focus.txt`, `events/FRA_events.txt`
- **Verification:** Tous les acceptance criteria passent après correction
- **Committed in:** `5b304e3` (Task 2), `e951885` (Task 3)

---

**Total deviations:** 1 auto-fixed (Rule 2 — correctness, évite confusion code/commentaire)
**Impact on plan:** Correction mineure de style commentaire. Aucun impact sur la syntaxe Clausewitz ni le comportement HoI4.

## Issues Encountered

Aucun problème bloquant. Le mismatch version 1.14 vs 1.18 documenté dans RESEARCH.md (Piège 2) était prévu et corrigé normalement.

## User Setup Required

**ACTION MANUELLE REQUISE** : Le fichier de registration Paradox `/home/yugos06/.local/share/Paradox Interactive/Hearts of Iron IV/mod/The Final Hour.mod` duplique `descriptor.mod`. Il doit être mis à jour pour cibler `supported_version="1.18.*"`.

**Options :**
1. Relancer le launcher Paradox — il régénère automatiquement ce fichier depuis `descriptor.mod`
2. Éditer manuellement le fichier et changer `supported_version="1.14.*"` en `supported_version="1.18.*"`

Sans cette mise à jour, le launcher peut continuer à afficher l'avertissement "mod obsolète" même si `descriptor.mod` est correct.

## Next Phase Readiness

- Plans 01-02 (bookmark + GFX) et 01-03 (localisation) peuvent démarrer — les conventions FRA_ sont établies
- Phase 2 (Ideas) peut compléter `common/ideas/FRA_ideas.txt` — squelette prêt
- Phase 3 (Events & on_actions) peut compléter `common/on_actions/FRA_on_actions.txt` et `events/FRA_events.txt` — squelettes prêts
- Phase 5 (Focus Tree) peut compléter `common/national_focus/FRA_focus.txt` — squelette avec priorité FRA établie
- Aucun bloqueur pour la suite

---
*Phase: 01-foundation*
*Completed: 2026-05-07*
