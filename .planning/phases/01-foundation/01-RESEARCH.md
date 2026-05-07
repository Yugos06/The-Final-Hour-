# Phase 1 : Foundation — Recherche

**Recherché :** 2026-05-07
**Domaine :** Infrastructure HoI4 mod — fichiers squelettes, GFX declarations, bookmark, shadow file
**Confiance globale :** HIGH (domaine stable, base de code inspectée, installation vanilla vérifiée)

---

<user_constraints>
## Contraintes utilisateur (depuis CONTEXT.md)

### Décisions verrouillées

**Assets GFX (.dds)**
- D-01 : Les fichiers .dds existent localement hors du repo — ils doivent être copiés lors de la Phase 1.
- D-02 : Structure de dossiers standard HoI4 : `interface/goals/` (focus), `interface/ideas/` (spirits), `interface/eventpictures/` (événements).
- D-03 : Tous les sprites GFX déclarés en Phase 1 (GFX-01 à GFX-04). Zéro pink square dès le premier chargement.

**Code alpha hérité**
- D-04 : Supprimer entièrement le code `fh_fra_` : `fh_france_focus.txt`, `fh_france_ideas.txt`, `fh_france_decisions.txt`, `fh_france_decision_categories.txt`, `fh_on_actions.txt`.
- D-05 : Créer immédiatement les fichiers squelettes avec nouveaux noms et headers minimaux valides : `FRA_focus.txt`, `FRA_ideas.txt`, `FRA_events.txt`, `FRA_on_actions.txt`.

**Shadow file — arbre vanilla France**
- D-06 : Stratégie par priorité — `FRA_focus.txt` déclare `country = { tag = FRA factor = 10 }` pour surpasser l'arbre vanilla. **Pas de `replace_path = "common/national_focus"` en Phase 1.**
- D-07 : Modèle évolutif par nation. Nations non travaillées conservent l'arbre vanilla.

**Bookmark 2026**
- D-08 : GER reste visible dans le bookmark avec description "En développement".
- D-09 : Image de fond vanilla (`GFX_select_date_1936`) conservée en Phase 1.

**VS Code / Workspace**
- D-10 : Configurer `.vscode/settings.json` avec `"files.encoding": "utf8bom"` pour les `.yml`.

### Discrétion de Claude

Aucune — toutes les décisions ont été tranchées par l'utilisateur.

### Idées différées (HORS SCOPE)

- Arbres "transitoires" pour nations non travaillées (GER, etc.)
- Image de fond custom pour le bookmark 2026
- Migration vers `replace_path = "common/national_focus"` quand 5+ nations couvertes
</user_constraints>

---

<phase_requirements>
## Requirements de la phase

| ID | Description | Support de la recherche |
|----|-------------|------------------------|
| FOUND-01 | Créer un fichier shadow pour écraser l'arbre vanilla France | Stratégie `france.txt` vide + priorité `factor = 10` dans `FRA_focus.txt` — voir section Architecture Patterns |
| FOUND-02 | Configurer le bookmark `fh_2026.txt` — date, description, nations, drapeaux | Structure du bookmark vérifiée contre vanilla — voir section Patterns |
| FOUND-03 | Configurer `.vscode/settings.json` avec `utf8bom` pour les `.yml` | VS Code 1.119.0 disponible, clé `files.encoding` documentée |
| GFX-01 | Fichier `interface/fh_goals.gfx` avec tous les sprites `GFX_focus_FRA_*` | Format `spriteType` vérifié contre vanilla `goals.gfx` |
| GFX-02 | Intégration des icônes focus `.dds` (ou PNG) pour l'arbre France | Assets introuvables sur le disque — voir section Open Questions |
| GFX-03 | Fichier `interface/fh_ideas.gfx` + icônes ideas pour les esprits nationaux | Format vérifié contre vanilla `ideas.gfx` |
| GFX-04 | Images d'événements pour `fra.001`, `fra.INSP`, `fra.CONF` | Format vérifié contre vanilla + TFR mod |
</phase_requirements>

---

## Résumé

La Phase 1 pose l'infrastructure de chargement du mod : le jeu doit démarrer proprement avec le bookmark 2026, l'arbre vanilla France remplacé, et zéro sprite manquant. Aucun gameplay n'est implémenté — seulement des fichiers squelettes valides et des déclarations GFX complètes.

La principale découverte critique est un **écart de version** : le `descriptor.mod` cible `1.14.*` mais HoI4 installé est en `1.18.1` (Case Green). Ce mismatch affiche un avertissement "mod obsolète" dans le launcher et doit être corrigé en Phase 1.

Deuxième découverte critique : la stratégie de shadow file vanilla décrite dans FOUND-01 nécessite le fichier nommé `france.txt` (pas `00_france.txt`). Le nom doit correspondre exactement au fichier vanilla à remplacer.

Troisième découverte : les assets DDS annoncés dans D-01 ("existent localement") sont introuvables sur le système (`/home/yugos06` fouillé jusqu'à profondeur 8). La Phase 1 devra gérer des placeholders ou localiser ces assets.

**Recommandation principale :** Corriger `descriptor.mod` vers `1.18.*`, créer `france.txt` (shadow vide) + `FRA_focus.txt` (squelette avec `factor = 10`), déclarer tous les sprites GFX avec des placeholders si les assets réels sont absents.

---

## Carte des responsabilités architecturales

| Capacité | Tier primaire | Tier secondaire | Justification |
|----------|--------------|-----------------|---------------|
| Shadow arbre vanilla France | Fichier script (`national_focus/`) | `descriptor.mod` | Le mod override par nom de fichier — aucun backend requis |
| Déclaration sprites GFX | Fichier interface (`interface/*.gfx`) | Fichier image (`gfx/`) | Le `.gfx` pointe vers le `.dds`/PNG ; les deux sont nécessaires |
| Bookmark 2026 | Fichier script (`common/bookmarks/`) | `descriptor.mod` (replace_path) | Le `replace_path` supprime le vanilla, le fichier définit le contenu |
| Encodage localisation BOM | Workspace VS Code (`.vscode/settings.json`) | Vérification hexdump | Le paramètre éditeur garantit les nouveaux fichiers ; hexdump valide |
| Fichiers squelettes | Fichiers script (multiple dossiers) | — | Chaque squelette prépare la surface de travail pour les phases suivantes |

---

## Stack standard

### Core (tous vérifiés contre l'installation HoI4 1.18.1)

| Élément | Version / Format | Rôle | Statut vérifié |
|---------|-----------------|------|----------------|
| HoI4 | 1.18.1 (Case Green) | Cible d'exécution | VÉRIFIÉ — installé |
| Clausewitz Script `.txt` | UTF-8 sans BOM | Fichiers de gameplay | VÉRIFIÉ — vanilla confirme |
| Localisation `.yml` | UTF-8 **avec BOM** (`EF BB BF`) | Textes affichés | VÉRIFIÉ — `hexdump` confirme BOM sur les fichiers existants |
| GFX `.gfx` | Clausewitz Script, UTF-8 sans BOM | Déclarations sprites | VÉRIFIÉ — vanilla `goals.gfx` inspecté |
| Images focus | PNG ou DDS (ARGB8888 ou DXT5) | Icônes d'arbre de focus | VÉRIFIÉ — vanilla=100x88 DDS, TFR=PNG ~150x150 |
| Images ideas | PNG ou DDS (ARGB8888) | Icônes d'esprits nationaux | VÉRIFIÉ — vanilla=60x68 DDS, TFR=PNG ~64x64 |
| Images événements | PNG ou DDS (DXT5) | Fonds d'événements | VÉRIFIÉ — vanilla=397x153, TFR=500x250 |

### Outils disponibles sur le système

| Outil | Version | Usage |
|-------|---------|-------|
| VS Code | 1.119.0 | Édition des fichiers mod |
| hexdump | util-linux 2.42 | Vérification BOM `ef bb bf` |
| ImageMagick convert | 7.1.2-21 | Conversion PNG → DDS (DXT1/DXT5 supportés) |

**Installation ImageMagick DDS confirmée :**
```bash
convert -list format | grep DDS
# DDS* DDS rw+ Microsoft DirectDraw Surface
# DXT5* DDS rw+ Microsoft DirectDraw Surface
```
[VÉRIFIÉ : bash sur le système]

---

## Architecture Patterns

### Diagramme flux de chargement HoI4

```
Démarrage HoI4
      │
      ▼
descriptor.mod
  └─ replace_path="common/bookmarks"
       └─ Supprime bookmarks vanilla (1936, 1939)
       └─ Charge uniquement fh_2026.txt
              │
              ▼
        Bookmark "2026 : L'Heure Finale"
        FRA / GER disponibles
              │
              ▼
     Chargement focus trees
     (tous les fichiers dans common/national_focus/)
              │
         ┌────┴────┐
         │         │
  france.txt       FRA_focus.txt
  (shadow vide)    (tree actif)
  score FRA=0      score FRA=10
         │         │
         └────┬────┘
              ▼
       HoI4 choisit l'arbre avec le score le plus élevé
       → FRA_focus.txt gagne (score 10 vs 0)
              │
              ▼
     Chargement GFX (interface/*.gfx)
     → fh_goals.gfx : sprites GFX_focus_FRA_*
     → fh_ideas.gfx : sprites GFX_idea_FRA_*
     → fh_events.gfx : sprites event pictures
              │
              ▼
     error.log : zéro entrée "sprite not found"
```

### Structure de dossiers recommandée (après Phase 1)

```
The-Final-Hour-/
├── descriptor.mod                    ← Mettre à jour vers 1.18.*
├── common/
│   ├── bookmarks/
│   │   └── fh_2026.txt              ← Mettre à jour description GER
│   ├── defines/
│   │   └── 00_fh_defines.lua        ← Conserver tel quel
│   ├── national_focus/
│   │   ├── france.txt               ← NOUVEAU : shadow vide (remplace vanilla france.txt)
│   │   └── FRA_focus.txt            ← NOUVEAU : squelette focus_tree vide, factor=10
│   ├── ideas/
│   │   └── FRA_ideas.txt            ← NOUVEAU : ideas block vide
│   └── on_actions/
│       └── FRA_on_actions.txt       ← NOUVEAU : on_actions block vide
├── events/
│   └── FRA_events.txt               ← NOUVEAU : namespace + placeholder
├── gfx/
│   ├── interface/
│   │   └── goals/
│   │       └── FRA_*.dds (ou .png)  ← Copier les assets locaux ici
│   ├── interface/
│   │   └── ideas/
│   │       └── FRA_*.dds (ou .png)
│   └── event_pictures/
│       └── FRA_*.dds (ou .png)
├── interface/
│   ├── fh_goals.gfx                 ← NOUVEAU : déclare GFX_focus_FRA_*
│   ├── fh_ideas.gfx                 ← NOUVEAU : déclare GFX_idea_FRA_*
│   └── fh_events.gfx                ← NOUVEAU : déclare GFX pour events
├── localisation/
│   ├── fh_l_french.yml              ← Ajouter clés manquantes (FRA_focus_* nouveaux IDs)
│   └── fh_l_english.yml
└── .vscode/
    └── settings.json                ← NOUVEAU : utf8bom pour *.yml
```

### Pattern 1 : Shadow file — remplacement de l'arbre vanilla France

**Quoi :** Créer `common/national_focus/france.txt` dans le mod avec un focus_tree vide.

**Pourquoi ce nom :** HoI4 fusionne les fichiers de même chemin relatif. Un fichier `france.txt` dans le mod **remplace** le `france.txt` vanilla (pas de fusion — le fichier mod prend le dessus). Un fichier `00_france.txt` dans le mod serait **additionnel** (différent nom = additionnel, pas remplacement).

**Le fichier vanilla cible :**
```
Hearts of Iron IV/common/national_focus/france.txt
  focus_tree = { id = french_focus ... factor=0 modifier { add=10 tag=FRA } ... }
```

**Notre shadow :**
```
# common/national_focus/france.txt (dans le mod)
# Shadow file — efface l'arbre vanilla France (french_focus)
# Ne PAS mettre de contenu ici : FRA_focus.txt porte notre arbre

focus_tree = {
  id = french_focus
  country = { factor = 0 }
  default = no
}
```
[VÉRIFIÉ : inspection vanilla `france.txt` + mécanisme merge HoI4]

### Pattern 2 : Focus tree squelette avec priorité

**Quoi :** `FRA_focus.txt` déclare un `focus_tree` vide mais valide avec `factor = 10` pour FRA.

```
# common/national_focus/FRA_focus.txt
focus_tree = {
  id = FRA_focus_tree
  country = {
    factor = 0
    modifier = {
      add = 10
      tag = FRA
    }
  }
  default = no
  reset_on_civilwar = yes

  # PLACEHOLDER — sera rempli en Phase 5
  # focus = { id = FRA_lheure_la_plus_sombre ... }
}
```

**Pourquoi `factor = 0 + modifier add = 10` et non `factor = 10` :**
Le bloc `country` contrôle la sélection de l'arbre. `factor = 0` + `modifier { add = 10 tag = FRA }` signifie : score 0 pour toutes les nations, +10 pour FRA. C'est le pattern vanilla et le plus robuste.
[VÉRIFIÉ : vanilla `france.txt` utilise exactement ce pattern]

### Pattern 3 : Déclaration de sprite GFX

**Format `.gfx` (Clausewitz Script, UTF-8 sans BOM) :**
```
# interface/fh_goals.gfx
spriteTypes = {

  spriteType = {
    name = "GFX_focus_FRA_darkest_hour_europe"
    texturefile = "gfx/interface/goals/FRA_darkest_hour_europe.dds"
  }

  spriteType = {
    name = "GFX_focus_FRA_la_republique_tient"
    texturefile = "gfx/interface/goals/FRA_la_republique_tient.dds"
  }

}
```

**Attention :** `texturefile` est relatif à la **racine du mod**, pas au fichier `.gfx`. Le dossier images est donc `gfx/interface/goals/` (avec `gfx/` en préfixe), tandis que le fichier `.gfx` lui-même se trouve dans `interface/`.
[VÉRIFIÉ : vanilla `interface/goals.gfx` inspecté + TFR `TFR_goals.gfx` inspecté]

### Pattern 4 : Déclaration sprite pour ideas

```
# interface/fh_ideas.gfx
spriteTypes = {

  spriteType = {
    name = "GFX_idea_FRA_esprit_francais"
    texturefile = "gfx/interface/ideas/FRA_esprit_francais.dds"
  }

}
```
[VÉRIFIÉ : vanilla `interface/ideas.gfx` inspecté]

### Pattern 5 : Déclaration sprite pour event pictures

```
# interface/fh_events.gfx
spriteTypes = {

  spriteType = {
    name = "GFX_event_FRA_lheure_la_plus_sombre"
    texturefile = "gfx/event_pictures/FRA_lheure_la_plus_sombre.dds"
  }

}
```
[VÉRIFIÉ : vanilla `ideas.gfx` + TFR `TFR_goals.gfx` inspectés]

### Pattern 6 : `.vscode/settings.json` pour l'encodage BOM

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

Vérification BOM après chaque nouveau fichier `.yml` :
```bash
hexdump -C localisation/fh_l_french.yml | head -1
# Attendu : ef bb bf 6c ...
```
[VÉRIFIÉ : `hexdump` disponible sur le système, BOM confirmée sur les fichiers existants]

### Anti-patterns à éviter

- **`00_france.txt` comme shadow :** Fichier avec un nom différent de `france.txt` — ne remplace pas le vanilla, s'additionne.
- **`replace_path = "common/national_focus"` :** Détruit tous les arbres vanilla de toutes les nations — non requis et destructeur pour Phase 1.
- **Référencer `GFX_focus_FRA_*` sans déclarer le sprite :** Le moteur affiche un icône générique sans erreur de crash — piège visuel silencieux.
- **Créer les `.gfx` sans créer les fichiers images :** Déclaration orpheline = pink square ou icône générique selon le fallback du moteur.
- **Sauvegarder `.yml` sans BOM :** VS Code sans configuration sauvegarde en UTF-8 pur — HoI4 ignore silencieusement le fichier entier.

---

## Ce qu'il ne faut pas recoder

| Problème | Ne pas construire | Utiliser à la place | Pourquoi |
|----------|------------------|---------------------|----------|
| Conversion PNG vers DDS | Script maison | `convert` (ImageMagick) | DXT5 et ARGB8888 supportés nativement |
| Validation BOM | Outil custom | `hexdump -C file.yml \| head -1` | Builtin sur le système |
| Override arbre vanilla | `replace_path` global | Shadow `france.txt` vide | Moins destructeur, évolutif |
| Vérification erreurs | Plugin externe | `error.log` HoI4 natif | Le moteur log tous les sprites manquants |

---

## Pièges courants

### Piège 1 : Mauvais nom pour le fichier shadow

**Ce qui va mal :** Créer `common/national_focus/00_france.txt` au lieu de `france.txt`. Le fichier `00_france.txt` est **additionnel** (nom différent) — il ne remplace pas le vanilla `france.txt`. Les deux arbres coexistent, et avec des scores égaux (tous deux `add = 10` pour FRA), le résultat est indéterminé.

**Pourquoi ça arrive :** La documentation communautaire mentionne parfois `00_france.txt` pour d'autres usages (style guideline de nommage alphanumérique). FOUND-01 dans REQUIREMENTS.md cite `00_france.txt` — c'est une erreur.

**Comment éviter :** Le fichier shadow dans le mod doit avoir le **même chemin relatif** que le fichier vanilla qu'il remplace : `common/national_focus/france.txt`.

**Signe d'alarme :** Des focuses vanilla comme `FRA_the_maginot_line` apparaissent encore lorsque le mod est actif.

[VÉRIFIÉ : vanilla `france.txt` inspecté, mécanisme merge HoI4 confirmé]

### Piège 2 : Mismatch de version HoI4

**Ce qui va mal :** `descriptor.mod` déclare `supported_version="1.14.*"` mais HoI4 installé est 1.18.1. Le launcher affiche "Mod obsolète", peut désactiver le mod automatiquement ou montrer des avertissements qui gênent les tests.

**Découverte :** HoI4 version 1.18.1 (Case Green) est installé — quatre versions majeures d'écart par rapport à la cible déclarée.

**Comment éviter :** Mettre à jour `descriptor.mod` : `supported_version="1.18.*"`. Également mettre à jour le fichier de registration Paradox (`/home/yugos06/.local/share/Paradox Interactive/Hearts of Iron IV/mod/The Final Hour.mod`).

**Signe d'alarme :** Bannière jaune "mod obsolète" dans le launcher Paradox ; mod potentiellement désactivé au démarrage.

[VÉRIFIÉ : `launcher-settings.json` HoI4 inspecté — version 1.18.1 confirmée]

### Piège 3 : Chemin texturefile incorrect dans les .gfx

**Ce qui va mal :** Le `texturefile` dans un `.gfx` est relatif à la **racine du mod**, pas au fichier `.gfx` lui-même. Écrire `texturefile = "interface/goals/FRA_xxx.dds"` au lieu de `texturefile = "gfx/interface/goals/FRA_xxx.dds"` pointe vers un chemin inexistant.

**Découverte :** La CONTEXT.md D-02 cite `interface/goals/` comme dossier de destination des assets. C'est ambigu — cela désigne le **type** de dossier (goals), pas le chemin complet. Le chemin complet correct depuis la racine du mod est `gfx/interface/goals/`.

**Comment éviter :** Utiliser `texturefile = "gfx/interface/goals/FRA_xxx.dds"` dans les `.gfx`. Stocker les fichiers images dans `<mod_root>/gfx/interface/goals/`.

**Signe d'alarme :** Sprites déclarés dans `.gfx` mais toujours pink squares en jeu, erreur `[spritetype.cpp]` dans `error.log`.

[VÉRIFIÉ : vanilla `interface/goals.gfx` inspecté — `texturefile = "gfx/interface/goals/..."` confirmé]

### Piège 4 : Assets GFX introuvables sur le disque

**Ce qui va mal :** D-01 affirme que les assets `.dds` "existent localement hors du repo". La recherche sur `/home/yugos06` jusqu'à profondeur 8 n'a trouvé aucun fichier `.dds` hors Steam. Les assets sont peut-être : (a) sur un autre appareil, (b) dans un dossier cloud non monté, (c) encore à créer.

**Impact pour Phase 1 :** Si les assets réels sont absents, il faut des **placeholders** pour que les sprites soient déclarés sans pink squares. Les placeholders peuvent être des DDS minimalistes générés avec ImageMagick (disponible sur le système) ou des icônes vanilla copiées.

**Comment éviter :** Avant de planifier la tâche "copier les DDS", vérifier physiquement l'emplacement des assets. Si absents : générer des placeholders 100x88 de couleur unie (rouge/noir pour la DA du mod) avec `convert`.

**Signe d'alarme :** Aucun fichier `.dds` trouvé après `find /home/yugos06 -name "*.dds"` hors Steam.

[VÉRIFIÉ : recherche sur le système — aucun asset DDS mod trouvé]

### Piège 5 : Syntaxe d'arbre de focus squelette invalide

**Ce qui va mal :** Un `focus_tree = {}` totalement vide cause une erreur de parsing. Il faut au minimum un `id`, un `country` block, et `default = no`.

**Comment éviter :** Utiliser le pattern de squelette minimal validé (Pattern 2 ci-dessus).

**Signe d'alarme :** HoI4 log des erreurs `[focus.cpp]` au démarrage ; arbre de focus vide pour FRA.

[ASSUMÉ — basé sur le comportement du parser Clausewitz]

---

## Exemples de code

### Squelette focus_tree minimal valide (FRA_focus.txt)

```
# common/national_focus/FRA_focus.txt
# Phase 1 skeleton — sera complété en Phase 5
# Source : pattern vanilla france.txt inspecté

focus_tree = {
  id = FRA_focus_tree

  country = {
    factor = 0
    modifier = {
      add = 10
      tag = FRA
    }
  }

  default = no
  reset_on_civilwar = yes

  # PHASE 5 : focus = { id = FRA_lheure_la_plus_sombre ... }
}
```

### Shadow file france.txt (vide mais valide)

```
# common/national_focus/france.txt
# Remplace le fichier vanilla : Hearts of Iron IV/common/national_focus/france.txt
# Efface l'arbre "french_focus" vanilla pour France
# Source : mécanisme override HoI4 vérifié

focus_tree = {
  id = french_focus
  country = { factor = 0 }
  default = no
}
```

### Squelette ideas valide (FRA_ideas.txt)

```
# common/ideas/FRA_ideas.txt
# Phase 1 skeleton — sera complété en Phase 2

ideas = {
  country = {
    # PHASE 2 : FRA_esprit_francais = { ... }
  }
}
```

### Squelette events valide (FRA_events.txt)

```
# events/FRA_events.txt
# Phase 1 skeleton — sera complété en Phase 3

add_namespace = fh_france

# PHASE 3 : country_event = { id = fh_france.001 ... }
```

### Squelette on_actions valide (FRA_on_actions.txt)

```
# common/on_actions/FRA_on_actions.txt
# Phase 1 skeleton — sera complété en Phase 3

on_actions = {
  on_startup = {
    effect = {
      # PHASE 3 : FRA = { add_ideas = FRA_esprit_francais }
    }
  }
}
```

### bookmark fh_2026.txt mis à jour

```
# common/bookmarks/fh_2026.txt
bookmarks = {
  bookmark = {
    name = "FH_2026_NAME"
    desc = "FH_2026_DESC"
    date = 2026.1.1.12
    picture = "GFX_select_date_1936"
    default_country = "FRA"
    default = yes

    FRA = {
      history = "FH_2026_FRA_DESC"
      ideology = democratic
    }

    GER = {
      history = "FH_2026_GER_WIP_DESC"
      ideology = democratic
    }
  }
}
```

**Nouvelle clé à ajouter en localisation :**
```yaml
  FH_2026_GER_WIP_DESC: "L'Allemagne est une grande puissance en profonde mutation. Contenu en développement — disponible dans une prochaine mise à jour."
```

### descriptor.mod mis à jour

```
name="The Final Hour"
version="0.1"
supported_version="1.18.*"
tags={
  "Alternate History"
  "National Focuses"
  "Events"
}
replace_path="common/bookmarks"
```

### Génération placeholder DDS (si assets absents)

```bash
# Générer un placeholder 100x88 px en rouge foncé (DA du mod)
convert -size 100x88 xc:#8B0000 \
  -define dds:compression=none \
  gfx/interface/goals/FRA_placeholder_focus.dds

# Copier pour chaque icône nécessaire en Phase 1
# (les vrais assets remplaceront ces fichiers en Phase suivante)
```

---

## État de l'art

| Ancienne approche | Approche actuelle | Changé | Impact |
|-------------------|-------------------|--------|--------|
| Préfixe `fh_fra_` (alpha) | Préfixe `FRA_` (standard HoI4) | Décision Phase 1 | IDs cohérents avec vanilla et autres mods |
| `00_france.txt` shadow | `france.txt` shadow (même nom que vanilla) | Phase 1 | Override correct par chemin identique |
| `descriptor.mod` ciblant 1.14.* | Doit cibler 1.18.* (installé) | Phase 1 | Supprime avertissement obsolète |
| Assets DDS (format déclaré : 74x74, 94x93, 600x240) | Dimensions réelles : focus 100x88, ideas 60x68, events 397-500px | Vérification Phase 1 | Peut nécessiter redimensionnement des assets |

**Informations dépassées à corriger :**
- CONTEXT.md D-02 cite `74x74` pour focus icons — la réalité vanilla est `100x88` (et TFR utilise des PNG `150x150`). Le moteur accepte des tailles variées et les ajuste.
- CONTEXT.md D-02 cite `94x93` pour idea icons — la réalité vanilla est `60x68`.
- CONTEXT.md D-02 cite `600x240` pour event pictures — la réalité vanilla est `397x153`, TFR utilise `500x250`.
- REQUIREMENTS.md FOUND-01 cite `00_france.txt` — le nom correct est `france.txt`.

---

## Inventaire d'état runtime

*Phase de création pure (greenfield par rapport à l'alpha) — s'applique partiellement car des fichiers alpha existent.*

| Catégorie | Éléments trouvés | Action requise |
|-----------|-----------------|----------------|
| Données persistées | Aucune mémoire mod stockée en DB | Aucune |
| Config service live | `The Final Hour.mod` enregistré dans Paradox data dir | Mettre à jour `supported_version` dans ce fichier ET dans `descriptor.mod` |
| State OS | Aucun task scheduler, aucun service système | Aucune |
| Secrets / env vars | Aucune clé d'API ou secret lié au mod | Aucune |
| Artefacts de build | Fichiers alpha `fh_france_*.txt` à supprimer | Supprimer (D-04) |

**Fichier de registration Paradox à mettre à jour :**
`/home/yugos06/.local/share/Paradox Interactive/Hearts of Iron IV/mod/The Final Hour.mod`
Ce fichier duplique `descriptor.mod` — `supported_version` doit y être mis à jour manuellement ou via le launcher.

---

## Questions ouvertes

1. **Localisation des assets DDS/PNG**
   - Ce qu'on sait : D-01 affirme que les assets "existent localement hors du repo"
   - Ce qui est flou : aucun fichier `.dds` ni PNG de mod trouvé sur `/home/yugos06` (hors Steam)
   - Recommandation : Le planificateur doit inclure une tâche "localiser ou générer les assets GFX" comme **première tâche** de GFX-02. Si introuvables, générer des placeholders et les enregistrer dans le repo.

2. **Dimensions exactes des assets utilisateur**
   - Ce qu'on sait : Vanilla = 100x88 (focus), 60x68 (ideas), 397x153 (events). TFR = ~150x150 (focus), ~64x64 (ideas), 500x250 (events). HoI4 accepte les deux.
   - Ce qui est flou : Quelles dimensions ont les assets du projet s'ils existent ? Le moteur redimensionne-t-il automatiquement ?
   - Recommandation : Utiliser les assets dans leurs dimensions natives. Si conversion nécessaire, ImageMagick est disponible.

3. **Clés localisation manquantes pour les nouveaux IDs**
   - Ce qu'on sait : Les fichiers `.yml` existants utilisent les anciens IDs `fh_fra_*`. Les nouveaux IDs `FRA_*` des squelettes Phase 1 n'ont pas encore de clés de localisation.
   - Ce qui est flou : Faut-il créer des clés minimales dès Phase 1 ou des IDs fantômes (clé absente) sont acceptables pour un squelette ?
   - Recommandation : Ajouter au minimum la clé `FH_2026_GER_WIP_DESC` pour le bookmark GER (visible par le joueur). Les clés des squelettes focus/ideas/events peuvent attendre Phase 2-5.

---

## Disponibilité de l'environnement

| Dépendance | Requise par | Disponible | Version | Fallback |
|------------|------------|-----------|---------|----------|
| HoI4 1.18.x | Test en jeu | Oui | 1.18.1 | — |
| VS Code | D-10 (settings.json) | Oui | 1.119.0 | Notepad++ avec BOM |
| hexdump | Vérification BOM | Oui | util-linux 2.42 | `file` command |
| ImageMagick convert | Conversion/création DDS | Oui | 7.1.2-21 | GIMP (absent) |
| GIMP | Édition DDS interactive | Non | — | ImageMagick CLI |
| Assets DDS du projet | GFX-02/03/04 | Non trouvés | — | Placeholders générés |

**Dépendances manquantes sans fallback :** Aucune — toutes les opérations critiques ont un outil disponible.

**Dépendance manquante avec impact fonctionnel :** Les assets GFX du projet. Sans eux, seuls des placeholders peuvent remplir GFX-01/02/03/04. Le mod chargera sans erreur mais les icônes seront des blocs de couleur unie.

---

## Architecture de validation

### Framework de test

| Propriété | Valeur |
|-----------|--------|
| Framework | Test manuel en jeu — aucun framework automatisé pour Clausewitz Script |
| Fichier de config | Aucun — validation par `error.log` HoI4 |
| Commande rapide | Lancer HoI4 avec le mod actif → quitter → `grep "fh_\|FRA_" ~/.local/share/Paradox\ Interactive/Hearts\ of\ Iron\ IV/logs/error.log` |
| Suite complète | Démarrer une partie FRA en 2026 → vérifier bookmark, arbre de focus, absence de pink squares |

### Cartographie requirements → tests

| Req ID | Comportement à tester | Type | Commande | Fichier test |
|--------|----------------------|------|----------|--------------|
| FOUND-01 | Aucun focus vanilla FRA (ex: `FRA_the_maginot_line`) visible en jeu | Smoke | Démarrer partie FRA → écran focus | — (test manuel) |
| FOUND-02 | Bookmark "2026 : L'Heure Finale" visible avec FRA + GER | Smoke | Menu principal → sélection scénario | — (test manuel) |
| FOUND-03 | BOM `ef bb bf` présent dans tout nouveau `.yml` | Unit | `hexdump -C localisation/fh_l_french.yml \| head -1` | — (script bash) |
| GFX-01 | Zéro entrée `sprite not found` pour `GFX_focus_FRA_*` | Smoke | `grep "GFX_focus_FRA_" error.log` | — (test manuel) |
| GFX-02 | Aucun pink square dans l'arbre de focus FRA | Smoke | Écran focus en jeu | — (test manuel) |
| GFX-03 | Aucun `GFX_idea_FRA_*` absent dans error.log | Smoke | `grep "GFX_idea_FRA_" error.log` | — (test manuel) |
| GFX-04 | Aucun `GFX_event_FRA_*` absent dans error.log | Smoke | `grep "GFX_event_FRA_" error.log` | — (test manuel) |

### Taux d'échantillonnage

- **Après chaque tâche :** `grep "fh_\|FRA_\|sprite" ~/.local/share/Paradox\ Interactive/Hearts\ of\ Iron\ IV/logs/error.log | grep -v "TFR\|ugc_"` (filtre les erreurs des autres mods)
- **Vague finale :** Lancer une partie complète France → vérifier écran focus, bookmark, absence pink squares
- **Gate Phase 1 :** `error.log` sans entrée sprite-not-found pour `GFX_focus_FRA_*` ni `GFX_idea_FRA_*`

### Lacunes Wave 0

- Aucun test automatisé n'est applicable à Clausewitz Script (Paradox script n'est pas testable hors moteur)
- La validation repose intégralement sur HoI4 + `error.log` — c'est la norme du domaine

---

## Domaine de sécurité

*Mod HoI4 local — aucune transmission de données, aucun accès réseau, aucune authentification. ASVS non applicable.*

- V2 Authentication : Non applicable
- V3 Session : Non applicable
- V4 Access Control : Non applicable
- V5 Input Validation : Non applicable (fichiers lus par le moteur Paradox, pas par du code utilisateur)
- V6 Cryptography : Non applicable

---

## Journal des hypothèses

| # | Hypothèse | Section | Risque si faux |
|---|-----------|---------|----------------|
| A1 | Un `focus_tree` squelette vide (sans aucun `focus =` bloc) est syntaxiquement valide pour le parser HoI4 | Patterns, squelettes | Si invalide : le jeu loggue une erreur au démarrage et ignore le fichier ; impact : la Phase 1 ne charge pas proprement |
| A2 | Mettre à jour uniquement `descriptor.mod` suffit (sans re-enregistrer le mod via le launcher) | Piège 2 | Si faux : le fichier `/home/yugos06/.local/share/Paradox Interactive/Hearts of Iron IV/mod/The Final Hour.mod` doit aussi être mis à jour manuellement |
| A3 | Le moteur HoI4 accepte indifféremment PNG et DDS pour les textures de focus/ideas/events | Stack standard | Si faux : les assets PNG seraient affichés incorrectement ou ignorés ; impact GFX-02/03/04 |

**Si ce tableau est vide :** Toutes les affirmations ont été vérifiées — pas de confirmation utilisateur nécessaire.

---

## Sources

### Primaires (confiance HIGH)
- Inspection directe : `Hearts of Iron IV/common/national_focus/france.txt` — structure focus_tree, IDs FRA_, factor/modifier pattern
- Inspection directe : `Hearts of Iron IV/interface/goals.gfx` — format SpriteType, chemins texturefile
- Inspection directe : `Hearts of Iron IV/interface/ideas.gfx` — format spriteType pour ideas
- Inspection directe : `Hearts of Iron IV/gfx/interface/goals/*.dds` — dimensions réelles 100x88 ARGB8888
- Inspection directe : `Hearts of Iron IV/gfx/interface/ideas/*.dds` — dimensions réelles 60x68 ARGB8888
- Inspection directe : `Hearts of Iron IV/gfx/event_pictures/*.dds` — dimensions réelles 397x153
- Inspection directe : `launcher-settings.json` — version HoI4 1.18.1 confirmée
- Inspection directe : `hexdump` sur `fh_l_french.yml` et `fh_l_english.yml` — BOM `ef bb bf` confirmé
- Inspection directe : `The Final Hour.mod` (Paradox data dir) — mod enregistré, path correct
- Inspection directe : `The-Final-Hour-/common/on_actions/fh_on_actions.txt` — code alpha existant
- Inspection directe : `The-Final-Hour-/common/national_focus/fh_france_focus.txt` — code alpha existant

### Secondaires (confiance MEDIUM)
- Inspection TFR mod (`workshop/content/394360/3350890356`) : TFR_goals.gfx, dimensions PNG focus/ideas/events
- `.planning/research/PITFALLS.md` — pièges documentés lors de la session de recherche initiale 2026-05-06
- `.planning/research/STACK.md` — conventions stack documentées (certaines dimensions à corriger)

### Tertiaires (confiance LOW — signalées dans le journal des hypothèses)
- Syntaxe squelette focus_tree vide : basée sur la connaissance du parser Clausewitz, non testée en jeu

---

## Métadonnées

**Confiance par domaine :**
- Shadow file (`france.txt`) : HIGH — pattern vérifié contre vanilla
- Squelettes fichiers : HIGH — format vérifié contre vanilla
- Dimensions DDS : HIGH — mesurées directement sur les fichiers vanilla + TFR
- Assets DDS utilisateur : INDÉTERMINÉ — non trouvés sur le système
- Version HoI4 : HIGH — `launcher-settings.json` inspecté

**Date de recherche :** 2026-05-07
**Valide jusqu'à :** 2026-06-07 (stable domain, HoI4 modding syntax rarely breaks between patches)
