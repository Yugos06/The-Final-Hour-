# Roadmap: The Final Hour (v1 — France)

## Overview

From a clean slate, the mod is built layer by layer following the Clausewitz dependency graph: infrastructure and GFX declarations first, then national spirit content, then event logic, then parliament decisions, then the focus tree that references everything, and finally a full localisation and QA pass that leaves zero errors in the game's log.

## Phases

**Phase Numbering:**
- Integer phases (1, 2, 3): Planned milestone work
- Decimal phases (2.1, 2.2): Urgent insertions (marked with INSERTED)

Decimal phases appear between their surrounding integers in numeric order.

- [x] **Phase 1: Foundation** - Shadow file, bookmark, VS Code config, and all GFX declarations wired to existing .dds assets (2026-05-07, human verification pending)
- [ ] **Phase 2: Ideas & Esprits Nationaux** - All national spirits coded with correct FRA_ namespace and modifiers
- [ ] **Phase 3: Events & on_actions** - Three narrative events and corrected on_actions trigger chain
- [ ] **Phase 4: Parliament & Decisions** - Assemblée Nationale idea, path-exclusive coalition decisions, paralysis resolution
- [ ] **Phase 5: Focus Tree** - Complete four-branch France focus tree referencing all prior layers
- [ ] **Phase 6: Localisation & QA** - Full FR/EN audit, modifier validation, error.log clean, chain test

## Phase Details

### Phase 1: Foundation
**Goal**: The mod loads cleanly in HoI4 with correct file structure, the 2026 bookmark is selectable, and all GFX sprites resolve to their .dds assets — no pink squares, no missing files.
**Depends on**: Nothing (first phase)
**Requirements**: FOUND-01, FOUND-02, FOUND-03, GFX-01, GFX-02, GFX-03, GFX-04
**Success Criteria** (what must be TRUE):
  1. HoI4 launches the mod without a missing-file error; `error.log` contains no sprite-not-found entries for `GFX_focus_FRA_*` or `GFX_idea_FRA_*`
  2. The "January 2026" bookmark appears in the main menu and France is listed as a selectable nation with the correct starting flags
  3. VS Code encodes every `.yml` file as UTF-8 with BOM (verified by `hexdump` showing `ef bb bf` as first three bytes)
  4. The vanilla France focus tree is fully shadowed — no `FRA_the_maginot_line` or other vanilla IDs appear when the mod is active
**Plans**: 3 plans
  - [x] 01-01-PLAN.md — Squelettes scripts FRA_, shadow file vanilla, descriptor 1.18.* (2026-05-07)
  - [x] 01-02-PLAN.md — Bookmark 2026 (GER WIP), localisation FH_2026_GER_WIP_DESC, config VS Code utf8bom (2026-05-07)
  - [x] 01-03-PLAN.md — Déclarations GFX (focus/ideas/events) + assets DDS (placeholders ou réels) (2026-05-07)

### Phase 2: Ideas & Esprits Nationaux
**Goal**: All France national spirits exist with correct FRA_ IDs, valid modifiers, and are assignable in-game — the political, military, economic, and EU spirits all apply their intended effects.
**Depends on**: Phase 1
**Requirements**: IDEA-01, IDEA-02, IDEA-03, IDEA-04, PARL-01
**Success Criteria** (what must be TRUE):
  1. `give_idea = FRA_assemblee_nationale` applied via console shows the PP malus representing the unstable coalition
  2. Military sub-branch spirits (`FRA_armee_de_terre_spirit`, `FRA_marine_spirit`, `FRA_armee_air_spirit`, `FRA_dissuasion_nucleaire_spirit`) each apply distinct modifiers without log errors
  3. Economic spirits (`FRA_crise_energetique_spirit`, `FRA_relance_nucleaire_spirit`, `FRA_economie_guerre_spirit`) are loadable and produce correct industry modifiers
  4. No idea in the file uses the old unprefixed ID pattern — every idea key begins with `FRA_`
**Plans**: TBD

### Phase 3: Events & on_actions
**Goal**: The three narrative events fire correctly when triggered via console, display their correct images, and the on_actions file uses startup/ruling-party-change triggers instead of the performance-killing on_daily pattern.
**Depends on**: Phase 2
**Requirements**: EVENT-01, EVENT-02, EVENT-03, QA-01
**Success Criteria** (what must be TRUE):
  1. `event fra.001` in console displays the 2026 introduction event with correct image and all option text (no PLACEHOLDER keys)
  2. `event fra.INSP` displays the infrastructure inspection report with accurate stats (38 civil factories, 12 military, 67% rail, CRITIQUE energy)
  3. `event fra.CONF` displays the Paris Conference event with both success and failure branches correctly gated on the four Fédération conditions
  4. `grep -r "on_daily" common/on_actions/` returns no results — the file uses only `on_startup`, `on_ruling_party_change`, and `on_civil_war_end`
**Plans**: TBD

### Phase 4: Parliament & Decisions
**Goal**: The Assemblée Nationale spirit applies its PP malus from game start, coalition path decisions are visible only after the correct focus is completed, and the parliamentary paralysis can be resolved through the designated focus or event chain.
**Depends on**: Phase 3
**Requirements**: PARL-02, PARL-03
**Success Criteria** (what must be TRUE):
  1. At game start, France has `FRA_assemblee_nationale` active and political power gain is visibly reduced compared to vanilla
  2. Coalition decisions are invisible until their corresponding path focus is completed (`has_completed_focus = FRA_xxx`) — verified by completing the focus in console then observing the decision become available
  3. Completing the designated resolution focus or triggering the resolution event removes the `FRA_assemblee_nationale` spirit and restores normal PP gain
**Plans**: TBD

### Phase 5: Focus Tree
**Goal**: The complete four-branch France focus tree is playable end-to-end — the player can reach the Fédération Européenne ending or the France Seule ending through coherent, non-softlocking paths, with all mutex pairs declared symmetrically and all focus positions correct.
**Depends on**: Phase 4
**Requirements**: FOCUS-01, FOCUS-02, FOCUS-03, FOCUS-04, FOCUS-05, FOCUS-06, FOCUS-07, FOCUS-08, FOCUS-09, FOCUS-10
**Success Criteria** (what must be TRUE):
  1. `FRA_lheure_la_plus_sombre` (cost 0) is visible at `y = 0` and triggers `fra.001` on completion — it is the first focus taken when France loads
  2. The Politique, Militaire, Économie, and Diplomatie branches are all reachable and display correctly in the focus tree UI with no overlapping or missing nodes
  3. Every focus with an `available` block has a corresponding `bypass` block — the AI does not soft-lock on any focus during a 10-year AI-only playthrough
  4. Both mutex pairs (`FRA_la_republique_tient` / `FRA_leffondrement` and all branch-level exclusive choices) are declared symmetrically — completing one blocks the other in both directions
  5. The `FRA_federation_europeenne` ending unlocks only when all four conditions are met simultaneously, and `FRA_la_france_seule` is available as an alternative ending
**UI hint**: yes
**Plans**: TBD

### Phase 6: Localisation & QA
**Goal**: Every French and English localisation key is filled with narrative-quality text, `guarantee_cost` is validated against vanilla, `cancel_if_invalid` is applied to all appropriate focus nodes, and HoI4 launches with zero critical errors in `error.log`.
**Depends on**: Phase 5
**Requirements**: LOC-01, LOC-02, LOC-03, QA-02, QA-03, QA-04, QA-05
**Success Criteria** (what must be TRUE):
  1. `grep -ri "PLACEHOLDER" localisation/` returns no results in either `fh_l_french.yml` or `fh_l_english.yml`
  2. `error.log` after a cold mod launch contains zero lines matching `[Error]` for any `FRA_` prefixed key, sprite, or modifier
  3. `cancel_if_invalid = yes` is present on every focus that has either an `available` or a `mutually_exclusive` block — verified by audit script or manual grep
  4. The full Fédération Européenne chain completes successfully using `focus.nochecks` in console — `fra.CONF` fires, both success and failure branches work, and the `FRA_federation_europeenne` focus becomes available on success
**Plans**: TBD

## Progress

**Execution Order:**
Phases execute in numeric order: 1 → 2 → 3 → 4 → 5 → 6

| Phase | Plans Complete | Status | Completed |
|-------|----------------|--------|-----------|
| 1. Foundation | 0/3 | Not started | - |
| 2. Ideas & Esprits Nationaux | 0/TBD | Not started | - |
| 3. Events & on_actions | 0/TBD | Not started | - |
| 4. Parliament & Decisions | 0/TBD | Not started | - |
| 5. Focus Tree | 0/TBD | Not started | - |
| 6. Localisation & QA | 0/TBD | Not started | - |
