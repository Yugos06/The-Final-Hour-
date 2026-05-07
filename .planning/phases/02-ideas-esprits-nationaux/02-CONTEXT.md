# Phase 2: Ideas & Esprits Nationaux - Context

**Gathered:** 2026-05-07
**Status:** Ready for planning

<domain>
## Phase Boundary

Phase 2 implémente les 11 esprits nationaux France dans `common/ideas/FRA_ideas.txt` — IDs corrects avec préfixe `FRA_`, modifiers valides, et assignables via console (`give_idea = FRA_*`). Les sprites GFX et les fichiers .dds sont déjà déclarés et présents (Phase 1 validée). Cette phase définit le CONTENU des esprits (modifiers, allowed, cancel conditions) et leur localisation initiale. Le câblage `on_startup` (quels esprits sont ajoutés automatiquement au lancement) est physiquement dans `FRA_on_actions.txt` mais les décisions sur lesquels sont actifs au départ sont capturées ici pour guider Phase 3.

</domain>

<decisions>
## Implementation Decisions

### Esprits actifs dès le démarrage (1er janvier 2026)

Ces esprits sont présents sans aucun focus ou décision — ils représentent l'état initial de la France en 2026. Leur câblage via `on_startup` dans `FRA_on_actions.txt` sera implémenté en Phase 3.

- **D-01:** `FRA_assemblee_nationale` — actif dès le départ. La coalition instable (577 sièges, aucune majorité) existe au 1er janvier 2026.
- **D-02:** `FRA_esprit_francais` — actif dès le départ. Esprit de base permanent de la nation, ne se perd jamais.
- **D-03:** `FRA_crise_energetique_spirit` — actif dès le départ. La crise énergétique 2026 est le contexte narratif immédiat, crée une pression de gameplay dès le premier tour.
- **D-04:** Les 4 esprits militaires (`FRA_armee_de_terre_spirit`, `FRA_marine_spirit`, `FRA_armee_air_spirit`, `FRA_dissuasion_nucleaire_spirit`) — actifs dès le départ en état **faible** (armée fonctionnelle mais sous-financée). Ils seront renforcés progressivement par les effects des focus militaires en Phase 5.

### Esprits ajoutés via focus/événements (pas au démarrage)

- **D-05:** `FRA_ve_republique_spirit` — ajouté par le focus `FRA_ve_republique_renforcee`.
- **D-06:** `FRA_regime_dexception_spirit` — ajouté par le focus `FRA_regime_dexception` (mutex avec D-05).
- **D-07:** `FRA_relance_nucleaire_spirit` — ajouté par le focus `FRA_relance_nucleaire_civil`.
- **D-08:** `FRA_economie_guerre_spirit` — ajouté par le focus `FRA_economie_de_guerre`.

### Mécanique de progression des esprits militaires

- **D-09:** Les esprits militaires utilisent le pattern **esprit unique évolutif** : un seul bloc idea par branche, les effects des focus militaires en Phase 5 modifient les modifiers via `modifier = { ... }` dans les effects de focus (`add_ideas` + `remove_ideas` ou via variables). Phase 2 définit uniquement l'état de base (faible).

### Modifiers de FRA_assemblee_nationale (PARL-01)

- **D-10:** `political_power_gain = -1.0` — flat, réduit le gain PP d'environ 50% (base France ~2/jour). Prévisible, lisible dans l'UI HoI4.
- **D-11:** `stability_factor = -0.05` — malus de stabilité supplémentaire représentant l'instabilité politique profonde de la coalition.

### Modifiers de FRA_crise_energetique_spirit (IDEA-03)

- **D-12:** Punitifs pour forcer une action rapide : `industrial_capacity_factory = -0.15`, `production_speed_buildings_factor = -0.10`. Le joueur doit investir dans la branche économique pour ne pas prendre de retard industriel.

### Modifiers des esprits militaires — état de base

- **D-13:** Représentent une armée fonctionnelle mais sous-financée. Petits malus sur chaque branche (ordre de grandeur : `army_core_attack_factor = -0.05` pour la Terre, valeurs analogues pour Marine/Air). La dissuasion nucléaire (`FRA_dissuasion_nucleaire_spirit`) a des modifiers de deterrence légers en base. Les valeurs exactes sont à la discrétion du planner/implémenteur dans cette fourchette.

### Esprits politiques — modifiers finaux dès Phase 2

- **D-14:** `FRA_ve_republique_spirit` et `FRA_regime_dexception_spirit` codés avec leurs modifiers finaux en Phase 2 (pas des stubs). Phase 5 appelle simplement `add_ideas = FRA_ve_republique_spirit`. Tout le contenu ideas est centralisé dans `FRA_ideas.txt`.

### Localisation

- **D-15:** Nouveaux fichiers de localisation dédiés aux ideas créés en Phase 2 : `localisation/FRA_ideas_l_french.yml` et `localisation/FRA_ideas_l_english.yml`. UTF-8 avec BOM. Structure scalable pour un mod qui va grandir (convention Road to 56, Kaiserreich).
- **D-16:** Niveau de texte : **nom + une phrase narrative**. Fonctionnel, lisible, respecte LOC-03 (pas de PLACEHOLDER). Phase 6 enrichit le ton narratif si besoin.
- **D-17:** Les clés générales existantes (bookmark, etc.) restent dans `fh_l_french.yml` / `fh_l_english.yml`. Les nouveaux fichiers ne les remplacent pas — HoI4 charge tous les `.yml` du dossier `localisation/`.

</decisions>

<canonical_refs>
## Canonical References

**Les agents en aval DOIVENT lire ces fichiers avant de planifier ou d'implémenter.**

### Roadmap & Requirements
- `.planning/ROADMAP.md` §Phase 2 — Goal, Requirements (IDEA-01/02/03/04, PARL-01), Success Criteria
- `.planning/REQUIREMENTS.md` §Ideas / Esprits Nationaux, §Système Assemblée Nationale — specs exactes

### Design & Architecture
- `PLAN.md` §Assemblée Nationale — système parlementaire, composition de la coalition (577 sièges, tableau des partis), règle du malus PP permanent
- `PLAN.md` §Branche POLITIQUE — FRA_ve_republique_renforcee / FRA_regime_dexception et leurs effets narratifs
- `PLAN.md` §Branche MILITAIRE — sous-branches Terre/Mer/Aérien/Dissuasion et leurs effets de focus (pour comprendre quelle progression les esprits militaires devront supporter)
- `PLAN.md` §Branche ÉCONOMIE — FRA_crise_energetique, FRA_relance_nucleaire_civil, FRA_economie_de_guerre
- `CLAUDE.md` — Conventions de nommage (`FRA_` prefix), contraintes HoI4

### Fichiers à modifier/créer en Phase 2
- `common/ideas/FRA_ideas.txt` — squelette Phase 1 existant, à remplir avec les 11 esprits
- `interface/fh_ideas.gfx` — 11 sprites déjà déclarés (Phase 1), chaque idea doit référencer le bon `GFX_idea_FRA_*`
- `gfx/interface/ideas/` — 11 fichiers `.dds` présents (Phase 1 validée), pas de création nécessaire
- `common/on_actions/FRA_on_actions.txt` — note Phase 3 : les esprits D-01 à D-04 seront câblés ici via `on_startup`

### Contexte Phase 1 (décisions portées)
- `.planning/phases/01-foundation/01-CONTEXT.md` — D-10 (BOM UTF-8), D-04 (anciens fichiers supprimés), D-05 (squelettes créés)
- `.planning/phases/01-foundation/01-VERIFICATION.md` — confirme GFX-03 validé (11 sprites), DDS présents

</canonical_refs>

<code_context>
## Existing Code Insights

### Assets réutilisables
- `interface/fh_ideas.gfx` : 11 sprites `GFX_idea_FRA_*` déclarés, chaque idea doit utiliser `picture = GFX_idea_FRA_[nom]` correspondant
- `gfx/interface/ideas/` : 11 fichiers `.dds` présents — aucune création de DDS requise en Phase 2

### Patterns établis
- `common/ideas/FRA_ideas.txt` : squelette `ideas = { country = { } }` valide — remplir le bloc `country` avec les 11 ideas
- Chaque idea suit le pattern vanilla : `ID = { picture = ..., allowed = { original_tag = FRA }, modifier = { ... } }`
- Localisation `.yml` : UTF-8 avec BOM obligatoire, namespace `l_french:` ou `l_english:` sur la première ligne

### Points d'intégration
- `common/on_actions/FRA_on_actions.txt` : le placeholder `# PHASE 3 : FRA = { add_ideas = FRA_esprit_francais }` doit être mis à jour pour inclure tous les esprits D-01 à D-04 — ce travail est Phase 3
- Les focus militaires en Phase 5 utiliseront `remove_ideas` + `add_ideas` sur les esprits militaires pour implémenter la progression (D-09)

</code_context>

<specifics>
## Specific Ideas

- Le système Assemblée Nationale est **le mechanic central** de Phase 2 : `FRA_assemblee_nationale` doit être le premier esprit testé via `give_idea` en console (Success Criteria SC-1). Le malus PP doit être immédiatement visible dans l'UI.
- Les 4 esprits militaires au départ représentent la France de 2026 : une armée professionnelle, une marine, une force aérienne et une dissuasion nucléaire **fonctionnelles mais sous-financées**. Le joueur voit que ces capacités existent — elles ont juste besoin de réformes (focus).
- La crise énergétique est punitive pour créer une **urgence narrative immédiate** — le joueur doit agir vite sur la branche économique. C'est intentionnel, cohérent avec le scénario 2026.

</specifics>

<deferred>
## Deferred Ideas

- Migration des clés générales de `fh_l_french.yml`/`fh_l_english.yml` vers des fichiers plus granulaires — à envisager lors d'une phase de cleanup ou Phase 6 si la taille des fichiers devient un problème de navigation.
- Tuning fin des modifiers militaires de base (valeurs exactes des sous-branches) — à ajuster après les premiers playtests, hors scope Phase 2.

</deferred>

---

*Phase: 2-Ideas-Esprits-Nationaux*
*Context gathered: 2026-05-07*
