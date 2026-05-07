---
phase: 01-foundation
plan: 03
subsystem: gfx
tags: [gfx, dds, placeholders, sprites, interface]
dependency_graph:
  requires: []
  provides:
    - interface/fh_goals.gfx (GFX_focus_FRA_*)
    - interface/fh_ideas.gfx (GFX_idea_FRA_*)
    - interface/fh_events.gfx (GFX_event_FRA_*)
    - gfx/interface/goals/ (20 DDS placeholders)
    - gfx/interface/ideas/ (11 DDS placeholders)
    - gfx/event_pictures/ (3 DDS placeholders)
  affects:
    - Phase 2 (ideas/spirits FRA — sprites déjà déclarés)
    - Phase 3 (events FRA — sprites déjà déclarés)
    - Phase 5 (focus tree FRA — sprites déjà déclarés)
tech_stack:
  added:
    - ImageMagick convert 7.1.2-21 (génération DDS placeholder)
  patterns:
    - Clausewitz spriteType declaration (vanilla goals.gfx format)
    - DDS DXT5 compression pour focus et events
    - DDS ARGB8888 (no compression) pour ideas
key_files:
  created:
    - interface/fh_goals.gfx
    - interface/fh_ideas.gfx
    - interface/fh_events.gfx
    - gfx/interface/goals/FRA_darkest_hour_europe.dds
    - gfx/interface/goals/FRA_la_republique_tient.dds
    - gfx/interface/goals/FRA_leffondrement.dds
    - gfx/interface/goals/FRA_etat_urgence.dds
    - gfx/interface/goals/FRA_reforme_constitutionnelle.dds
    - gfx/interface/goals/FRA_balle_dentree.dds
    - gfx/interface/goals/FRA_rearmement_urgence.dds
    - gfx/interface/goals/FRA_revue_strategique_nationale.dds
    - gfx/interface/goals/FRA_dissuasion_nucleaire.dds
    - gfx/interface/goals/FRA_doctrine_ia_militaire.dds
    - gfx/interface/goals/FRA_crise_energetique.dds
    - gfx/interface/goals/FRA_inspection_nationale_territoire.dds
    - gfx/interface/goals/FRA_amenagements_regionaux.dds
    - gfx/interface/goals/FRA_relance_nucleaire_civil.dds
    - gfx/interface/goals/FRA_economie_de_guerre.dds
    - gfx/interface/goals/FRA_sauver_leurope.dds
    - gfx/interface/goals/FRA_financer_union_europeenne.dds
    - gfx/interface/goals/FRA_conference_de_paris.dds
    - gfx/interface/goals/FRA_federation_europeenne.dds
    - gfx/interface/goals/FRA_la_france_seule.dds
    - gfx/interface/ideas/FRA_esprit_francais.dds
    - gfx/interface/ideas/FRA_assemblee_nationale.dds
    - gfx/interface/ideas/FRA_ve_republique_spirit.dds
    - gfx/interface/ideas/FRA_regime_dexception_spirit.dds
    - gfx/interface/ideas/FRA_armee_de_terre_spirit.dds
    - gfx/interface/ideas/FRA_marine_spirit.dds
    - gfx/interface/ideas/FRA_armee_air_spirit.dds
    - gfx/interface/ideas/FRA_dissuasion_nucleaire_spirit.dds
    - gfx/interface/ideas/FRA_crise_energetique_spirit.dds
    - gfx/interface/ideas/FRA_relance_nucleaire_spirit.dds
    - gfx/interface/ideas/FRA_economie_guerre_spirit.dds
    - gfx/event_pictures/FRA_lheure_la_plus_sombre.dds
    - gfx/event_pictures/FRA_inspection.dds
    - gfx/event_pictures/FRA_conference_paris.dds
  modified: []
decisions:
  - "D-01 : Aucun asset DDS réel trouvé sur /home/yugos06 hors Steam — 100% placeholders générés"
  - "D-03 : Zéro pink square garanti — 34 fichiers DDS créés avant déclaration"
  - "GFX-01/02/03/04 : Tous les sprites déclarés en Phase 1, disponibles pour Phases 2-5"
metrics:
  duration: "~2 minutes"
  completed_date: "2026-05-07"
  tasks_completed: 3
  files_created: 37
---

# Phase 1 Plan 03 : GFX Sprite Catalog — Summary

Création du catalogue GFX complet pour la France : 3 fichiers de déclaration Clausewitz + 34 fichiers DDS placeholder (rouge sang / noir charbon / or) garantissant zéro pink square dès le premier chargement du mod.

## Résultat de la recherche d'assets réels

**Recherche effectuée :** `find /home/yugos06 -path '*/Steam/*' -prune -o -name "*.dds" -print` et PNG équivalent.

**Résultat : Aucun asset réel trouvé.** Aucun fichier `.dds` ou `.png` de mod trouvé en dehors du dossier Steam sur `/home/yugos06`. Les assets annoncés dans D-01 ("existent localement hors du repo") sont introuvables — cohérent avec la découverte de RESEARCH.md (Piège 4, Question ouverte 1).

## Tableau : Placeholders générés vs Assets réels

| Catégorie | Fichiers | Type | Statut |
|-----------|----------|------|--------|
| Focus icons (20 fichiers) | `FRA_darkest_hour_europe.dds`, ..., `FRA_la_france_seule.dds` | Placeholder 100x88 DXT5 rouge #8B0000 | Placeholder — à remplacer |
| Idea icons (11 fichiers) | `FRA_esprit_francais.dds`, ..., `FRA_economie_guerre_spirit.dds` | Placeholder 60x68 ARGB8888 noir #1A1A1A | Placeholder — à remplacer |
| Event pictures (3 fichiers) | `FRA_lheure_la_plus_sombre.dds`, `FRA_inspection.dds`, `FRA_conference_paris.dds` | Placeholder 397x153 DXT5 or #C9A961 | Placeholder — à remplacer |

**Total : 34 fichiers, tous placeholders.**

## Fichiers .gfx créés

| Fichier | Sprites déclarés | Utilisé par |
|---------|-----------------|-------------|
| `interface/fh_goals.gfx` | 20 × `GFX_focus_FRA_*` | Phase 5 (focus tree FRA) |
| `interface/fh_ideas.gfx` | 11 × `GFX_idea_FRA_*` | Phase 2 (national spirits FRA) |
| `interface/fh_events.gfx` | 3 × `GFX_event_FRA_*` | Phase 3 (events fra.001, fra.INSP, fra.CONF) |

**Total : 34 sprites déclarés (20 + 11 + 3).**

## Mapping sprite -> focus/idea/event

### Focus (Phase 5)

| Sprite GFX | Branche | Rôle |
|-----------|---------|------|
| `GFX_focus_FRA_darkest_hour_europe` | Racine | Focus de départ |
| `GFX_focus_FRA_la_republique_tient` | Politique | Mutex A — maintenir la Ve République |
| `GFX_focus_FRA_leffondrement` | Politique | Mutex B — l'effondrement institutionnel |
| `GFX_focus_FRA_etat_urgence` | Politique | État d'urgence |
| `GFX_focus_FRA_reforme_constitutionnelle` | Politique | Réforme constitutionnelle |
| `GFX_focus_FRA_balle_dentree` | Politique | Balle d'entrée |
| `GFX_focus_FRA_rearmement_urgence` | Militaire | Réarmement d'urgence |
| `GFX_focus_FRA_revue_strategique_nationale` | Militaire | Revue stratégique nationale |
| `GFX_focus_FRA_dissuasion_nucleaire` | Militaire | Dissuasion nucléaire |
| `GFX_focus_FRA_doctrine_ia_militaire` | Militaire | Doctrine IA militaire |
| `GFX_focus_FRA_crise_energetique` | Économie | Crise énergétique |
| `GFX_focus_FRA_inspection_nationale_territoire` | Économie | Inspection nationale du territoire |
| `GFX_focus_FRA_amenagements_regionaux` | Économie | Aménagements régionaux |
| `GFX_focus_FRA_relance_nucleaire_civil` | Économie | Relance nucléaire civil |
| `GFX_focus_FRA_economie_de_guerre` | Économie | Économie de guerre |
| `GFX_focus_FRA_sauver_leurope` | Diplomatie | Sauver l'Europe |
| `GFX_focus_FRA_financer_union_europeenne` | Diplomatie | Financer l'Union Européenne |
| `GFX_focus_FRA_conference_de_paris` | Diplomatie | Conférence de Paris |
| `GFX_focus_FRA_federation_europeenne` | Ending | Fédération Européenne |
| `GFX_focus_FRA_la_france_seule` | Ending | La France Seule |

### Ideas/Spirits (Phase 2)

| Sprite GFX | Spirit | Contexte |
|-----------|--------|----------|
| `GFX_idea_FRA_esprit_francais` | `FRA_esprit_francais` | Spirit de base, force-applied au startup |
| `GFX_idea_FRA_assemblee_nationale` | `FRA_assemblee_nationale` | PARL-01 — Système parlementaire |
| `GFX_idea_FRA_ve_republique_spirit` | Politique Ve République | |
| `GFX_idea_FRA_regime_dexception_spirit` | Politique régime d'exception | |
| `GFX_idea_FRA_armee_de_terre_spirit` | Militaire armée de terre | |
| `GFX_idea_FRA_marine_spirit` | Militaire marine nationale | |
| `GFX_idea_FRA_armee_air_spirit` | Militaire armée de l'air | |
| `GFX_idea_FRA_dissuasion_nucleaire_spirit` | Militaire dissuasion | |
| `GFX_idea_FRA_crise_energetique_spirit` | Économique crise énergie | |
| `GFX_idea_FRA_relance_nucleaire_spirit` | Économique relance nucléaire | |
| `GFX_idea_FRA_economie_guerre_spirit` | Économique économie de guerre | |

### Event Pictures (Phase 3)

| Sprite GFX | Event | Description |
|-----------|-------|-------------|
| `GFX_event_FRA_lheure_la_plus_sombre` | `fh_france.001` | L'heure la plus sombre (event de départ FRA) |
| `GFX_event_FRA_inspection` | `fra.INSP` | Inspection nationale |
| `GFX_event_FRA_conference_paris` | `fra.CONF` | Conférence de Paris |

## Note pour l'utilisateur : Remplacement des placeholders

Les vrais assets DDS peuvent remplacer les placeholders **SANS modifier les fichiers .gfx**. Les noms de fichiers et les chemins sont fixés :

- Focus : `gfx/interface/goals/FRA_<nom>.dds` (100x88 px recommandé, DXT5)
- Ideas : `gfx/interface/ideas/FRA_<nom>.dds` (60x68 px recommandé, ARGB8888)
- Events : `gfx/event_pictures/FRA_<nom>.dds` (397x153 px recommandé, DXT5)

Le moteur HoI4 accepte également des PNG dans ces dossiers. Pour convertir un PNG en DDS :
```bash
convert input.png -define dds:compression=dxt5 output.dds
```

## Décisions implémentées

- **D-01** : Recherche des assets réels effectuée — aucun trouvé, placeholders générés
- **D-03** : Zéro pink square au premier chargement — 34 DDS créés avant déclaration
- **GFX-01** : `interface/fh_goals.gfx` créé avec 20 sprites `GFX_focus_FRA_*`
- **GFX-02** : 20 fichiers DDS focus créés dans `gfx/interface/goals/`
- **GFX-03** : `interface/fh_ideas.gfx` créé avec 11 sprites `GFX_idea_FRA_*`
- **GFX-04** : `interface/fh_events.gfx` créé avec 3 sprites `GFX_event_FRA_*`

## Commits

| Tâche | Commit | Description |
|-------|--------|-------------|
| 1 | `5298986` | 34 fichiers DDS placeholders |
| 2 | `980e834` | interface/fh_goals.gfx (20 sprites) |
| 3 | `b44e1cc` | interface/fh_ideas.gfx + fh_events.gfx (14 sprites) |

## Deviations from Plan

Aucune deviation — plan exécuté exactement tel qu'écrit.

La commande `convert` affiche des avertissements de dépréciation IMv7 ("use magick instead of convert") mais fonctionne correctement et produit des fichiers DDS valides. Ce n'est pas une erreur.

## Known Stubs

| Stub | Fichier | Raison |
|------|---------|--------|
| Tous les fichiers .dds sont des placeholders couleur unie | `gfx/interface/goals/*.dds`, `gfx/interface/ideas/*.dds`, `gfx/event_pictures/*.dds` | Assets réels absents du système — placeholders intentionnels (D-01/D-03). Remplacement prévu lorsque l'utilisateur fournira les vrais assets. |

## Self-Check: PASSED

Fichiers vérifiés :
- `interface/fh_goals.gfx` : FOUND
- `interface/fh_ideas.gfx` : FOUND
- `interface/fh_events.gfx` : FOUND
- `gfx/interface/goals/` : 20 fichiers DDS — FOUND
- `gfx/interface/ideas/` : 11 fichiers DDS — FOUND
- `gfx/event_pictures/` : 3 fichiers DDS — FOUND

Commits vérifiés :
- `5298986` : FOUND (chore(01-03): 34 DDS placeholders)
- `980e834` : FOUND (feat(01-03): fh_goals.gfx)
- `b44e1cc` : FOUND (feat(01-03): fh_ideas.gfx + fh_events.gfx)
