---
plan: 01-02
phase: 01-foundation
status: complete
decisions_implemented:
  - D-08
  - D-09
  - D-10
  - FOUND-02
  - FOUND-03
key-files:
  modified:
    - common/bookmarks/fh_2026.txt
    - localisation/fh_l_french.yml
    - localisation/fh_l_english.yml
  created:
    - .vscode/settings.json
---

## Plan 01-02 — Bookmark WIP GER + Localisation + Config VS Code

### Tâche 1 : Mise à jour du bookmark fh_2026.txt

**Modification unique (D-08) :** La valeur de `history` dans le bloc GER est passée de `"FH_2026_GER_DESC"` à `"FH_2026_GER_WIP_DESC"`.

Diff avant/après :
```
- history = "FH_2026_GER_DESC"
+ history = "FH_2026_GER_WIP_DESC"
```

Éléments préservés (D-09) : `picture = "GFX_select_date_1936"`, `default_country = "FRA"`, `date = 2026.1.1.12`, bloc FRA inchangé.

Encodage : UTF-8 sans BOM (fichier .txt Clausewitz Script).

### Tâche 2 : Ajout de la clé FH_2026_GER_WIP_DESC dans les deux fichiers de localisation

**Clé ajoutée dans `localisation/fh_l_french.yml` :**
```yaml
  FH_2026_GER_WIP_DESC:0 "L'Allemagne est une grande puissance en profonde mutation. Contenu en développement — disponible dans une prochaine mise à jour."
```

**Clé ajoutée dans `localisation/fh_l_english.yml` :**
```yaml
  FH_2026_GER_WIP_DESC:0 "Germany is a major power undergoing deep transformation. Content in development — available in a future update."
```

BOM vérifié et préservé :
- `localisation/fh_l_french.yml` : début `ef bb bf` ✓
- `localisation/fh_l_english.yml` : début `ef bb bf` ✓

### Tâche 3 : Création de .vscode/settings.json (FOUND-03, D-10)

```json
{
  "[yaml]": {
    "files.encoding": "utf8bom"
  },
  "files.associations": {
    "*.yml": "yaml"
  }
}
```

JSON syntaxiquement valide. Encodage UTF-8 sans BOM sur le fichier .json lui-même. Cette configuration garantit que VS Code écrit automatiquement le BOM sur tous les fichiers .yml à l'avenir, évitant les pertes silencieuses de localisation HoI4.

### Self-Check: PASSED

| Critère | Résultat |
|---------|---------|
| `FH_2026_GER_WIP_DESC` dans fh_2026.txt | ✓ (1 occurrence) |
| Ancienne clé `FH_2026_GER_DESC` absente | ✓ (0 occurrence) |
| `picture = GFX_select_date_1936` préservé | ✓ |
| BOM `ef bb bf` sur fh_l_french.yml | ✓ |
| BOM `ef bb bf` sur fh_l_english.yml | ✓ |
| Clé WIP présente dans fh_l_french.yml | ✓ |
| Clé WIP présente dans fh_l_english.yml | ✓ |
| .vscode/settings.json créé + JSON valide | ✓ |
| `files.encoding: utf8bom` configuré | ✓ |
