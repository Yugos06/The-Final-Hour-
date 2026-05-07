# Phase 1: Foundation - Discussion Log

> **Audit trail only.** Do not use as input to planning, research, or execution agents.
> Decisions are captured in CONTEXT.md — this log preserves the alternatives considered.

**Date:** 2026-05-07
**Phase:** 1-Foundation
**Areas discussed:** Assets GFX (.dds), Code alpha hérité, Shadow file France, Bookmark GER

---

## Assets GFX (.dds)

| Option | Description | Selected |
|--------|-------------|----------|
| Dossier local hors repo | Assets existent sur le disque, pas encore commités | ✓ |
| Encore à créer | Placeholders uniquement en Phase 1 | |
| Dans le repo, autre chemin | Quelque part dans le projet | |

**User's choice:** Dossier local hors repo
**Notes:** Les assets visuels (.dds) existent localement mais n'ont jamais été intégrés au repo.

---

| Option | Description | Selected |
|--------|-------------|----------|
| interface/goals/ (Recommandé) | Standard HoI4 : goals/, ideas/, eventpictures/ | ✓ |
| gfx/interface/ | Chemin alternatif moins standard | |
| Je décide du chemin | Structure personnalisée | |

**User's choice:** interface/goals/ (Recommandé)
**Notes:** Structure HoI4 standard adoptée.

---

| Option | Description | Selected |
|--------|-------------|----------|
| Tout déclarer maintenant (Recommandé) | GFX-01 à GFX-04 tous en Phase 1 | ✓ |
| Seulement les focus icons | GFX-03/04 différés | |

**User's choice:** Tout déclarer maintenant
**Notes:** Évite les pink squares dès le premier chargement. Tous les sprites déclarés en Phase 1.

---

## Code alpha hérité

| Option | Description | Selected |
|--------|-------------|----------|
| Tout supprimer en Phase 1 (Recommandé) | Table rase du code fh_fra_ | ✓ |
| Garder jusqu'à la phase qui remplace | Coexistence temporaire | |
| Archiver dans _legacy | Référence sans impact jeu | |

**User's choice:** Tout supprimer en Phase 1
**Notes:** Décision claire — reconstruction complète, pas de refacto.

---

| Option | Description | Selected |
|--------|-------------|----------|
| Oui — fichiers squelettes (Recommandé) | FRA_focus.txt, FRA_ideas.txt, etc. avec headers minimaux | ✓ |
| Non — juste supprimer | Chaque phase crée son fichier | |

**User's choice:** Fichiers squelettes immédiats
**Notes:** Le mod doit charger sans erreurs de syntaxe dès Phase 1.

---

## Shadow file France

| Option | Description | Selected |
|--------|-------------|----------|
| Factor prioritaire par nation (Recommandé) | country = { tag = FRA factor = 10 } | ✓ |
| replace_path maintenant + copie vanilla | Contrôle total immédiat, plus de maintenance | |
| replace_path différé | Transition quand 5+ nations couverts | |

**User's choice:** Factor prioritaire par nation (via free-text "c'est nickel la 1")
**Notes:** L'utilisateur pense à un grand mod multi-nations. La stratégie évolutive par nation permet d'ajouter GER, etc. quand prêt. Les nations non travaillées gardent leur arbre vanilla. Option replace_path envisageable à terme.

---

## Bookmark GER

| Option | Description | Selected |
|--------|-------------|----------|
| Garder GER avec mention WIP (Recommandé) | Description "En développement" | ✓ |
| Retirer GER du bookmark | FRA uniquement jusqu'au contenu GER | |
| Garder GER tel quel | Sans modification | |

**User's choice:** Garder GER avec mention WIP

---

| Option | Description | Selected |
|--------|-------------|----------|
| Image vanilla temporaire (Recommandé) | GFX_select_date_1936 en Phase 1 | ✓ |
| Custom si l'asset .dds existe | Intégrer immédiatement si disponible | |

**User's choice:** Image vanilla temporaire
**Notes:** Remplacée par custom quand les assets correspondants seront identifiés.

---

## Claude's Discretion

Aucune — toutes les décisions ont été tranchées par l'utilisateur.

## Deferred Ideas

- Arbres de focus "transitoires" pour nations non travaillées (GER, etc.)
- Image de fond custom pour le bookmark 2026
- Migration vers `replace_path = "common/national_focus"` quand 5+ nations couvertes
