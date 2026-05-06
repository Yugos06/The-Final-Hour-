# The Final Hour

## What This Is

Mod Hearts of Iron IV se déroulant en janvier 2026 dans un futur proche alternatif. Le joueur incarne une nation en crise — instabilité politique, crise énergétique mondiale, guerre en Ukraine, montée de l'IA militaire — et doit façonner le destin de son pays avant l'effondrement. La v1 couvre la France avec un arbre de focus complet, des événements narratifs et des assets visuels intégrés.

## Core Value

Un mod HoI4 à la profondeur narrative de Road to 56 et au style cinématique de The Fire Rises — chaque focus raconte quelque chose, chaque choix a des conséquences réelles.

## Requirements

### Validated

- ✓ Contexte narratif mondial 2026 défini (Ukraine, Iran, crise énergétique, IA militaire) — existant
- ✓ Nations jouables identifiées : France (priorité) et Allemagne — existant
- ✓ Design complet des arbres de focus France et Allemagne documenté dans PLAN.md — existant
- ✓ Assets visuels (.dds) prêts à intégrer — existant

### Active

- [ ] Arbre de focus France complet (toutes branches : politique, militaire, économie, diplomatie)
- [ ] Système Assemblée Nationale (PP dynamique selon coalition)
- [ ] Événements France liés aux focus (fra.001, fra.CONF, fra.INSP)
- [ ] Localisation française (fh_l_french.yml) — textes focus + événements
- [ ] Localisation anglaise (fh_l_english.yml) — textes focus + événements
- [ ] Intégration assets GFX (icônes .dds des focus France)
- [ ] Ideas/esprits nationaux France (fh_france_ideas.txt)
- [ ] Decisions France si nécessaires
- [ ] Bookmark de départ 2026 configuré correctement

### Out of Scope (v1)

- Arbre de focus Allemagne — priorité v2, le design existe dans PLAN.md
- Autres nations jouables — v3+
- Nouvelles mécaniques de jeu IA — complexité trop haute pour v1
- Système économique énergie avancé — v2

## Context

Projet HoI4 mod en alpha 0.1 — le code existant (`fh_fra_` prefix) est à remplacer entièrement. Le design complet est documenté dans `PLAN.md` à la racine du repo. Les assets visuels existent mais ne sont pas encore intégrés. Le mod sera publié sur GitHub ET le Steam Workshop.

**Inspirations :**
- **The Fire Rises** — palette rouge sang / noir charbon / or, icônes cinématiques style propagande, DA sombre
- **Road to 56** — profondeur et ramification des arbres de focus, choix mutuellement exclusifs, branches parallèles

**Code existant :** Arbre France alpha (`common/national_focus/fh_france_focus.txt`, 615 lignes), events alpha (427 lignes), ideas (187 lignes), localisation FR/EN (330 lignes). Tout sera reconstruit proprement.

## Constraints

- **Convention nommage** : Préfixe `FRA_` pour tous les IDs France (focus, events, flags, GFX) — convention HoI4 standard, cohérent avec PLAN.md
- **Compatibilité HoI4** : Cibler la dernière version stable de HoI4 (pas de dépendances aux mods tiers)
- **Langue** : Français et anglais obligatoires en v1 pour maximiser l'audience Steam Workshop
- **GFX v1** : Les icônes .dds existantes doivent être intégrées dans la v1 France (pas de fallback générique)
- **Partis interdits** : KPD reconstitué et NPD/néo-nazis ne doivent JAMAIS être jouables (Article 21 §2 Grundgesetz — s'applique aussi au design GER quand il viendra)

## Key Decisions

| Décision | Rationale | Outcome |
|----------|-----------|---------|
| Reconstruction complète (pas de refacto) | Le code alpha utilise `fh_fra_` au lieu de `FRA_`, structure différente de PLAN.md — refacto coûterait plus cher que repartir | — Pending |
| v1 = France uniquement | Finir une nation complète vaut mieux qu'avoir deux nations à moitié faites | — Pending |
| Préfixe `FRA_` standard HoI4 | Cohérent avec PLAN.md, évite la confusion avec l'ancien code | — Pending |
| GFX inclus en v1 | Les assets sont prêts — pas de raison de les différer, ça fait partie de l'identité visuelle | — Pending |
| GitHub + Steam Workshop | Open source pour les contributions + distribution Steam pour les joueurs | — Pending |

## Evolution

Ce document évolue à chaque transition de phase et jalon.

**Après chaque transition de phase** (via `/gsd-transition`) :
1. Requirements invalidés ? → Déplacer dans Out of Scope avec raison
2. Requirements validés ? → Déplacer dans Validated avec référence de phase
3. Nouveaux requirements émergés ? → Ajouter dans Active
4. Décisions à journaliser ? → Ajouter dans Key Decisions
5. "What This Is" toujours exact ? → Mettre à jour si dérivé

**Après chaque jalon** (via `/gsd-complete-milestone`) :
1. Revue complète de toutes les sections
2. Vérification Core Value — toujours la bonne priorité ?
3. Audit Out of Scope — les raisons sont-elles toujours valides ?
4. Mettre à jour Context avec l'état actuel

---
*Dernière mise à jour : 2026-05-06 après initialisation*
