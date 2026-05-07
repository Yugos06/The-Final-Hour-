---
phase: 1
slug: foundation
status: draft
nyquist_compliant: false
wave_0_complete: false
created: 2026-05-07
---

# Phase 1 — Validation Strategy

> Contrat de validation par phase pour le sampling de feedback pendant l'exécution.
> Domaine HoI4 mod : validation principalement manuelle via lancement du jeu et inspection de `error.log`.

---

## Test Infrastructure

| Propriété | Valeur |
|-----------|-------|
| **Framework** | Test manuel HoI4 + bash scripts (aucun framework automatisé — Clausewitz Script non testable hors moteur) |
| **Config file** | Aucun — validation via `error.log` HoI4 |
| **Commande rapide** | `grep -E "GFX_focus_FRA_|GFX_idea_FRA_|GFX_event_FRA_" ~/.local/share/Paradox\ Interactive/Hearts\ of\ Iron\ IV/logs/error.log` |
| **Suite complète** | Lancer HoI4 avec le mod actif → démarrer partie FRA en 2026 → vérifier bookmark, focus tree, absence pink squares |
| **Durée estimée** | ~5 min (lancement HoI4 + test manuel) |

---

## Taux de sampling

- **Après chaque commit de tâche GFX :** `grep -E "sprite|GFX_" ~/.local/share/Paradox\ Interactive/Hearts\ of\ Iron\ IV/logs/error.log | grep -v "TFR\|ugc_"`
- **Après chaque wave :** Lancer HoI4 et vérifier que le mod charge sans erreur critique
- **Avant `/gsd-verify-work` :** Suite complète verte (error.log sans entrées `GFX_focus_FRA_*` ni `GFX_idea_FRA_*`)
- **Latence maximale de feedback :** ~10 min (inclut temps de lancement HoI4)

---

## Carte de vérification par tâche

| Task ID | Plan | Wave | Requirement | Comportement sécurisé | Type | Commande automatisée | Fichier | Statut |
|---------|------|------|-------------|----------------------|------|---------------------|---------|--------|
| 1-01-01 | 01 | 1 | FOUND-01 | N/A | smoke | `grep "FRA_the_maginot_line\|french_focus" common/national_focus/france.txt` → doit être vide | ❌ W0 | ⬜ pending |
| 1-01-02 | 01 | 1 | FOUND-02 | N/A | smoke | Lancement HoI4 → menu principal → bookmark 2026 visible avec FRA + GER | — (manuel) | ⬜ pending |
| 1-01-03 | 01 | 1 | FOUND-03 | N/A | unit | `hexdump -C localisation/fh_l_french.yml \| head -1 \| grep "ef bb bf"` → exit 0 | ❌ W0 | ⬜ pending |
| 1-02-01 | 02 | 1 | GFX-01 | N/A | smoke | `grep "GFX_focus_FRA_" ~/.local/.../error.log` → aucun résultat | — (manuel) | ⬜ pending |
| 1-02-02 | 02 | 1 | GFX-02 | N/A | smoke | Écran focus FRA en jeu → aucun pink square | — (manuel) | ⬜ pending |
| 1-02-03 | 02 | 1 | GFX-03 | N/A | smoke | `grep "GFX_idea_FRA_" ~/.local/.../error.log` → aucun résultat | — (manuel) | ⬜ pending |
| 1-02-04 | 02 | 1 | GFX-04 | N/A | smoke | `grep "GFX_event_FRA_" ~/.local/.../error.log` → aucun résultat | — (manuel) | ⬜ pending |

*Statut : ⬜ pending · ✅ green · ❌ red · ⚠️ flaky*

---

## Exigences Wave 0

- [ ] Script bash de vérification BOM : `hexdump -C localisation/fh_l_french.yml | head -1 | grep "ef bb bf"` — couvre FOUND-03
- [ ] Vérification présence shadow file : `test -f common/national_focus/france.txt && echo "OK"` — couvre FOUND-01
- [ ] Commande grep error.log post-lancement — couvre GFX-01/02/03/04

*Note : Aucun framework de test installable — le domaine HoI4 mod se valide uniquement par lancement du moteur de jeu.*

---

## Vérifications manuelles uniquement

| Comportement | Requirement | Pourquoi manuel | Instructions |
|--------------|-------------|-----------------|--------------|
| Bookmark 2026 visible dans le launcher | FOUND-02 | Nécessite interface graphique HoI4 | Lancer HoI4 → Launcher → "Play" → Menu principal → sélectionner scénario → vérifier "L'Heure Finale - 2026.01.01" avec FRA et GER |
| Aucun pink square dans l'arbre de focus | GFX-02 | Nécessite rendu graphique HoI4 | Démarrer partie FRA → ouvrir écran de focus → inspecter visuellement toutes les icônes |
| Arbre vanilla France absent | FOUND-01 | Nécessite interface graphique HoI4 | Démarrer partie FRA → vérifier que "Ligne Maginot" et autres focuses vanilla n'apparaissent PAS |

---

## Validation Sign-Off

- [ ] Toutes les tâches ont une vérification `<automated>` ou une dépendance Wave 0
- [ ] Continuité du sampling : pas 3 tâches consécutives sans vérification automatisée
- [ ] Wave 0 couvre toutes les références MISSING
- [ ] Aucun flag watch-mode
- [ ] Latence de feedback < 10 min
- [ ] `nyquist_compliant: true` défini dans le frontmatter

**Approbation :** pending
