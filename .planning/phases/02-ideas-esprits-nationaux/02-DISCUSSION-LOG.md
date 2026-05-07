# Phase 2: Ideas & Esprits Nationaux - Discussion Log

> **Audit trail only.** Do not use as input to planning, research, or execution agents.
> Decisions are captured in CONTEXT.md — this log preserves the alternatives considered.

**Date:** 2026-05-07
**Phase:** 02-ideas-esprits-nationaux
**Areas discussed:** Esprits au démarrage, Malus PP coalition, Intensité éco & mili, Localisation Phase 2

---

## Zone 1 — Esprits au démarrage

### Q1 : Quels esprits actifs dès le 1er janvier 2026 ?

| Option | Description | Selected |
|--------|-------------|----------|
| FRA_assemblee_nationale | Coalition instable dès jan 2026 — malus PP immédiat | ✓ |
| FRA_esprit_francais | Esprit de base permanent de la nation | ✓ |
| FRA_crise_energetique_spirit | Contexte 2026 — pression immédiate sur le joueur | ✓ |
| Aucun autre (seulement Assemblée) | Seul FRA_assemblee_nationale automatique | |

**User's choice:** FRA_assemblee_nationale + FRA_esprit_francais + FRA_crise_energetique_spirit

---

### Q2 : Esprits militaires — départ ou via focus ?

| Option | Description | Selected |
|--------|-------------|----------|
| Uniquement via focus | Représentent des réformes, débloqués par investissement | |
| Tous actifs au départ | France a une armée, marine, air, dissuasion en 2026 | |

**User's choice (libre) :** "les esprits seront directement là — mais le joueur doit faire les focus de chaque branche pour accumuler leur buff pour avoir la pleine puissance"

**Notes :** Les 4 esprits militaires sont actifs dès le départ en état faible. Les focus militaires (Phase 5) renforcent progressivement les modifiers.

---

### Q3 : Mécanique de progression militaire

| Option | Description | Selected |
|--------|-------------|----------|
| Remplacement | remove_ideas + add_ideas avec tiers distincts | |
| Esprit unique évolutif | Un seul esprit, modifiers cumulés via focus effects | ✓ |
| Base + bonus spirit | Esprit de base permanent + bonus spirits qui s'accumulent | |

**User's choice :** Esprit unique évolutif

---

### Q4 : Confirmation bilan démarrage

7 esprits actifs au départ :
- FRA_assemblee_nationale (coalition)
- FRA_esprit_francais (base nation)
- FRA_crise_energetique_spirit (économie)
- FRA_armee_de_terre_spirit, FRA_marine_spirit, FRA_armee_air_spirit, FRA_dissuasion_nucleaire_spirit (militaires, faibles)

4 esprits via focus uniquement :
- FRA_ve_republique_spirit, FRA_regime_dexception_spirit (politique, mutuellement exclusifs)
- FRA_relance_nucleaire_spirit, FRA_economie_guerre_spirit (économique)

**User's choice :** Oui, c'est correct

---

## Zone 2 — Malus PP coalition (FRA_assemblee_nationale)

### Q1 : Type de malus PP

| Option | Description | Selected |
|--------|-------------|----------|
| political_power_gain flat | Prévisible, lisible dans UI HoI4 | ✓ |
| political_power_factor % | Proportionnel mais moins prévisible | |
| Les deux combinés | Plus complexe à équilibrer | |

**User's choice :** political_power_gain flat

---

### Q2 : Valeur du malus PP flat

| Option | Description | Selected |
|--------|-------------|----------|
| -1.0 PP/jour | ~50% de réduction, urgence narrative | ✓ |
| -0.5 PP/jour | -25%, gêne modérée | |
| -1.5 PP/jour | -75%, quasi-paralysie, risque frustration | |

**User's choice :** -1.0 PP/jour

---

### Q3 : Modifiers supplémentaires sur FRA_assemblee_nationale

| Option | Description | Selected |
|--------|-------------|----------|
| PP only | Focalisé sur son rôle mécanique clé | |
| PP + stabilité malus | political_power_gain = -1.0 + stability_factor = -0.05 | ✓ |
| PP + war support malus | political_power_gain = -1.0 + war_support = -0.03 | |

**User's choice :** PP + stabilité malus (stability_factor = -0.05)

---

## Zone 3 — Intensité éco & mili

### Q1 : FRA_crise_energetique_spirit — intensité

| Option | Description | Selected |
|--------|-------------|----------|
| Punitif — force une action rapide | industrial_capacity_factory = -0.15, production_speed_buildings_factor = -0.10 | ✓ |
| Modéré — gêne sans paralyser | -0.08 / -0.05 | |
| Léger — flavor uniquement | -0.05 symbolique | |

**User's choice :** Punitif

---

### Q2 : État de base des esprits militaires

| Option | Description | Selected |
|--------|-------------|----------|
| Armée fonctionnelle mais sous-financée | Petits malus, armée existe mais pas optimisée | ✓ |
| Armée en crise | Malus modérés -0.10 à -0.15 | |
| Bonus neutres (flavor) | Modifiers très légers, juste pour tenir un emplacement UI | |

**User's choice :** Armée fonctionnelle mais sous-financée (petits malus, ~-0.05)

---

### Q3 : Esprits politiques — modifiers finaux dès Phase 2

| Option | Description | Selected |
|--------|-------------|----------|
| Oui, modifiers finaux dès Phase 2 | Phase 5 appelle juste add_ideas, contenu centralisé | ✓ |
| Stubs maintenant, remplis en Phase 5 | Plus de contexte en Phase 5 mais re-travail à prévoir | |

**User's choice :** Oui, modifiers finaux dès Phase 2

---

## Zone 4 — Localisation Phase 2

### Q1 : Niveau de texte

| Option | Description | Selected |
|--------|-------------|----------|
| Nom + description narrative | Nom + une phrase, respecte LOC-03, enrichi en Phase 6 | ✓ |
| Nom seulement | Risque de clé vide dans l'UI HoI4 | |
| Texte narratif complet | Idéal mais long × 11 esprits × 2 langues | |

**Notes :** L'utilisateur a demandé la recommandation de Claude → recommandation validée.

---

### Q2 : Fichiers de localisation

| Option | Description | Selected |
|--------|-------------|----------|
| Séparer par type dès maintenant | FRA_ideas_l_french.yml + FRA_ideas_l_english.yml | ✓ |
| Garder tout dans fh_l_*.yml | Migration plus tard | |

**Notes :** Discussion sur la scalabilité HoI4 — l'utilisateur s'inquiétait de la taille des fichiers. Confirmation que HoI4 charge tous les .yml sans limite technique. Convention des grands mods adoptée.

---

## Claude's Discretion

- Valeurs exactes des modifiers militaires de base (fourchette ~-0.05 sur les stats de combat) — l'utilisateur a validé la fourchette, les valeurs précises laissées au planner/implémenteur.
- Wording des descriptions narratives des esprits (nom + une phrase) — contenu éditorial laissé à l'implémentation.

## Deferred Ideas

- Migration des clés générales de fh_l_*.yml vers des fichiers plus granulaires — si la navigation dans les grands fichiers devient un problème.
- Tuning fin des modifiers militaires après playtests.
