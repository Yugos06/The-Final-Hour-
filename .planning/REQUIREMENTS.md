# Requirements — The Final Hour (v1 : France)

## v1 Requirements

### Foundation & Infrastructure

- [ ] **FOUND-01** : Créer `common/national_focus/00_france.txt` (fichier vide) pour écraser l'arbre vanilla France et éviter les conflits avec le préfixe `FRA_`
- [ ] **FOUND-02** : Configurer le bookmark `fh_2026.txt` avec la date janvier 2026, description narrative, nations jouables (FRA en priorité), et drapeaux de départ corrects
- [ ] **FOUND-03** : Configurer VS Code workspace avec `"files.encoding": "utf8bom"` pour tous les `.yml` — vérification BOM sur chaque nouveau fichier de localisation

### Arbre de Focus France

- [ ] **FOCUS-01** : Focus de départ `FRA_lheure_la_plus_sombre` (cost: 0, effets narratifs, déclenche `fra.001`)
- [ ] **FOCUS-02** : Choix initial mutuellement exclusif — `FRA_la_republique_tient` vs `FRA_leffondrement`
- [ ] **FOCUS-03** : Branche Politique complète — `FRA_etat_urgence` → `FRA_reforme_constitutionnelle` → `FRA_balle_dentree` → choix `FRA_ecraser_gauche_radicale` / `FRA_neutraliser_extreme_droite` → `FRA_ve_republique_renforcee` / `FRA_regime_dexception`
- [ ] **FOCUS-04** : Branche Militaire complète — `FRA_rearmement_urgence` → `FRA_revue_strategique_nationale` → sous-branches Terre / Mer / Aérien en parallèle + `FRA_dissuasion_nucleaire` + `FRA_doctrine_ia_militaire`
- [ ] **FOCUS-05** : Branche Économie complète — `FRA_crise_energetique` → `FRA_inspection_nationale_territoire` (déclenche `fra.INSP`) → `FRA_amenagements_regionaux` + `FRA_relance_nucleaire_civil` + `FRA_economie_de_guerre`
- [ ] **FOCUS-06** : Branche Diplomatie complète — `FRA_sauver_leurope` → OTAN / Méditerranée / Ukraine / Nouvelle politique africaine + `FRA_financer_union_europeenne` → `FRA_conference_de_paris` (déclenche `fra.CONF`)
- [ ] **FOCUS-07** : Ending Fédération Européenne (`FRA_federation_europeenne`) avec ses 4 conditions obligatoires + ending alternatif `FRA_la_france_seule`
- [ ] **FOCUS-08** : Chaque focus avec bloc `available` doit avoir un bloc `bypass` correspondant (prévention soft-lock IA)
- [ ] **FOCUS-09** : Chaque paire `mutually_exclusive` déclarée symétriquement dans les deux nœuds
- [ ] **FOCUS-10** : Trunk focus positionné à `y = 0` (pas `y = -1`)

### Système Assemblée Nationale

- [ ] **PARL-01** : Idea `FRA_assemblee_nationale` appliquant un malus PP permanent représentant la coalition instable (577 sièges, aucun parti à la majorité)
- [ ] **PARL-02** : Décisions path-exclusives pour chaque coalition politique — visibles uniquement si le focus de chemin correspondant a été complété (`visible = { has_completed_focus = FRA_xxx }`)
- [ ] **PARL-03** : Résolution de la paralysie parlementaire via focus ou event qui retire le malus PP

### Événements

- [ ] **EVENT-01** : `fra.001` — événement d'introduction déclenché par `FRA_lheure_la_plus_sombre`, pose le contexte narratif 2026, `is_triggered_only = yes`
- [ ] **EVENT-02** : `fra.INSP` — rapport d'inspection du territoire (usines civiles: 38, militaires: 12, ferroviaire: 67%, énergie: CRITIQUE), déclenché par `FRA_inspection_nationale_territoire`, `is_triggered_only = yes`
- [ ] **EVENT-03** : `fra.CONF` — Conférence de Paris, vérifie les 4 conditions Fédération Européenne, deux branches (succès / échec), `is_triggered_only = yes`

### Ideas / Esprits Nationaux

- [ ] **IDEA-01** : Esprits politiques — `FRA_assemblee_nationale` (malus PP coalition), `FRA_ve_republique_spirit` (bonus stabilité), `FRA_regime_dexception_spirit` (bonus efficacité / malus dip.)
- [ ] **IDEA-02** : Esprits militaires — un esprit par sous-branche débloquée (Terre, Mer, Aérien) + `FRA_dissuasion_nucleaire_spirit`
- [ ] **IDEA-03** : Esprits économiques — `FRA_crise_energetique_spirit` (malus industrie), `FRA_relance_nucleaire_spirit` (bonus prod.), `FRA_economie_guerre_spirit`
- [ ] **IDEA-04** : Renommer les ideas sans préfixe nation (`fh_union_europeenne` → `FRA_union_europeenne`, etc.) — cohérence namespace

### GFX / Assets Visuels

- [ ] **GFX-01** : Fichier `interface/fh_goals.gfx` déclarant tous les sprites `GFX_focus_FRA_*` avec chemins vers les fichiers `.dds` correspondants
- [ ] **GFX-02** : Intégration de toutes les icônes focus `.dds` (74×74 px, DXT5) pour l'arbre France
- [ ] **GFX-03** : Fichier `interface/fh_ideas.gfx` + icônes ideas `.dds` (94×93 px) pour les esprits nationaux
- [ ] **GFX-04** : Images d'événements `.dds` (600×240 px) pour `fra.001`, `fra.INSP`, `fra.CONF`

### Localisation

- [ ] **LOC-01** : `localisation/fh_l_french.yml` (UTF-8 avec BOM) — clés pour tous les focus, events, ideas, décisions France
- [ ] **LOC-02** : `localisation/fh_l_english.yml` (UTF-8 avec BOM) — traduction anglaise complète, écrite simultanément avec le français (jamais différée)
- [ ] **LOC-03** : Chaque clé localisation écrite au moment de la création du contenu — aucune clé `PLACEHOLDER` en production

### Qualité & Correctness

- [ ] **QA-01** : Supprimer le bloc `on_daily` dans `fh_on_actions.txt`, remplacer par `on_startup` + `on_ruling_party_change` + `on_civil_war_end`
- [ ] **QA-02** : Vérifier la validité du modifier `guarantee_cost` dans l'idea `fh_union_europeenne` contre les fichiers vanilla
- [ ] **QA-03** : Audit `cancel_if_invalid = yes` sur tous les focus avec `available` ou `mutually_exclusive`
- [ ] **QA-04** : Zéro erreur critique dans `error.log` HoI4 au lancement
- [ ] **QA-05** : Test de la chaîne complète Fédération Européenne (4 conditions) via `focus.nochecks` + console

---

## v2 Requirements (Différés)

- Arbre de focus Allemagne (GER_) — design complet disponible dans PLAN.md
- Events Allemagne (ger.001, ger.INSP, ger.VERFASSUNGSGERICHT)
- Esprits diplomatiques France (FRA_union_europeenne, FRA_otan_spirit) — refonte avec nouvelles conventions
- Events politiques majeurs supplémentaires (manifestations, crises de coalition)
- `ai_will_do` factor tuning — requiert playtests

---

## Out of Scope

- Autres nations jouables (v3+) — trop tôt, la profondeur France prime
- Nouvelles mécaniques IA militaire — complexité système, v3+
- Système économique énergie avancé (custom resource) — v2 avec Allemagne
- Multijoueur / compétitif — hors scope du mod narratif

---

## Traceability

*(Rempli par le roadmapper)*

| REQ-ID | Phase | Plan |
|--------|-------|------|
| ... | ... | ... |
