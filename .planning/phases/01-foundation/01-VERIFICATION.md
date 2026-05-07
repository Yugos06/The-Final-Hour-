---
status: passed
phase: 01-foundation
verified_at: 2026-05-07
requirements_verified:
  - FOUND-01
  - FOUND-02
  - FOUND-03
  - GFX-01
  - GFX-02
  - GFX-03
  - GFX-04
---

## Rapport de vérification — Phase 01 : Foundation

### Vérifications automatisées — TOUTES PASSÉES

| Exigence | Vérification | Résultat |
|----------|-------------|---------|
| FOUND-01 | `common/national_focus/france.txt` contient `id = french_focus` | ✓ PASS |
| FOUND-01 | `common/national_focus/FRA_focus.txt` : `FRA_focus_tree` + `tag = FRA` + `add = 10` | ✓ PASS |
| FOUND-02 | `common/bookmarks/fh_2026.txt` référence `FH_2026_GER_WIP_DESC` | ✓ PASS |
| FOUND-03 | BOM `ef bb bf` présent sur `fh_l_french.yml` | ✓ PASS |
| FOUND-03 | BOM `ef bb bf` présent sur `fh_l_english.yml` | ✓ PASS |
| FOUND-03 | `.vscode/settings.json` : `files.encoding = utf8bom` pour `[yaml]` | ✓ PASS |
| GFX-01 | `interface/fh_goals.gfx` : 20 sprites `GFX_focus_FRA_*` | ✓ PASS |
| GFX-02 | `gfx/interface/goals/` : ≥20 fichiers `.dds` présents | ✓ PASS |
| GFX-03 | `interface/fh_ideas.gfx` : 11 sprites `GFX_idea_FRA_*` | ✓ PASS |
| GFX-04 | `interface/fh_events.gfx` : 3 sprites `GFX_event_FRA_*` | ✓ PASS |
| D-04 | Fichiers alpha supprimés (`fh_france_focus.txt`, `fh_france_ideas.txt`, etc.) | ✓ PASS |
| D-05 | Squelettes `FRA_ideas.txt`, `FRA_on_actions.txt`, `FRA_events.txt` présents | ✓ PASS |
| descriptor | `supported_version="1.18.*"` dans `descriptor.mod` | ✓ PASS |

### Vérifications manuelles requises (human_needed)

Ces items nécessitent le lancement de HoI4 avec le mod activé :

1. **Chargement sans erreur** — Vérifier `~/.local/share/Paradox Interactive/Hearts of Iron IV/logs/error.log` après lancement : aucune ligne `sprite not found` pour `GFX_focus_FRA_*`, `GFX_idea_FRA_*`, `GFX_event_FRA_*`.

2. **Bookmark 2026 visible** — Dans le menu principal, le bookmark "Janvier 2026" apparaît et France est sélectionnable comme nation de départ.

3. **Description GER WIP** — Sélectionner Allemagne dans le bookmark 2026 : la description affichée correspond au contenu de `FH_2026_GER_WIP_DESC` (texte "En développement").

4. **Arbre vanilla masqué** — Lancer une partie France, vérifier dans l'arbre des focuses qu'aucun focus vanilla (`FRA_the_maginot_line`, etc.) n'apparaît.

5. **Fichier Paradox externe** — Relancer le launcher Paradox pour régénérer `/home/yugos06/.local/share/Paradox Interactive/Hearts of Iron IV/mod/The Final Hour.mod` avec `supported_version=1.18.*`.

### Points d'attention (review code)

- **WR-01** : Clé `FH_2026_GER_DESC` orpheline dans les deux fichiers `.yml` (non critique, à nettoyer).
- **IN-01** : `CLAUDE.md` mentionne encore `1.14.*` — mettre à jour vers `1.18.*`.
