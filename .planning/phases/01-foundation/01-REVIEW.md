---
phase: 01
status: findings
files_reviewed: 13
findings:
  critical: 0
  warning: 1
  info: 1
  total: 2
---

## Code Review — Phase 01: Foundation

**Profondeur :** standard
**Fichiers examinés :** descriptor.mod, common/bookmarks/fh_2026.txt, common/national_focus/france.txt, common/national_focus/FRA_focus.txt, common/ideas/FRA_ideas.txt, common/on_actions/FRA_on_actions.txt, events/FRA_events.txt, localisation/fh_l_french.yml, localisation/fh_l_english.yml, .vscode/settings.json, interface/fh_goals.gfx, interface/fh_ideas.gfx, interface/fh_events.gfx

---

### WR-01 — Clé de localisation `FH_2026_GER_DESC` orpheline

**Sévérité :** Warning
**Fichiers :** `localisation/fh_l_french.yml:330`, `localisation/fh_l_english.yml:330`

La clé `FH_2026_GER_DESC` est définie dans les deux fichiers de localisation mais n'est pas référencée dans `common/bookmarks/fh_2026.txt` (qui utilise `FH_2026_GER_WIP_DESC` depuis la décision D-08). C'est un vestige de l'ancienne structure alpha que le plan 01-02 a partiellement nettoyé en créant la nouvelle clé mais sans supprimer l'ancienne.

La clé orpheline ne casse rien, mais représente de la dette technique et risque d'induire en erreur lors des phases suivantes.

**Correction :**
```bash
# Supprimer la ligne FH_2026_GER_DESC dans les deux fichiers .yml
# Attention : préserver le BOM (ef bb bf) — ne pas réécrire le fichier entièrement
```

---

### IN-01 — `CLAUDE.md` mentionne la version HoI4 `1.14.*` au lieu de `1.18.*`

**Sévérité :** Info
**Fichier :** `CLAUDE.md:7`

La ligne "Currently targets HoI4 version 1.14.*" dans CLAUDE.md est désynchronisée avec `descriptor.mod` qui déclare `supported_version="1.18.*"` (décision documentée dans RESEARCH.md — Piège 2). Ce n'est pas un bug dans le code du mod, mais une incohérence documentaire qui sera source de confusion pour les futurs contributeurs.

**Correction :** Mettre à jour CLAUDE.md ligne 7 : `1.14.*` → `1.18.*`

---

## Notes d'inspection (non-problèmes confirmés)

- **BOM localisation** : Confirmé présent (`ef bb bf`) dans les deux fichiers `.yml` — aucun problème.
- **Sprites DDS** : 20 focus + 11 ideas + 3 event pictures générés par plan 01-03 — tous présents sous `gfx/`.
- **Chemins texturefile** : Tous les chemins dans les `.gfx` commencent par `gfx/` — Piège 3 correctement évité.
- **Format clés localisation** : Le format `KEY: "value"` utilisé est cohérent avec les clés existantes du mod. HoI4 accepte les deux formats.
- **Bloc `on_startup` vide** : Placeholder intentionnel pour Phase 3, documenté dans le code.
