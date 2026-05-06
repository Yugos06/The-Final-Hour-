# Architecture: HoI4 Multi-Nation Mod Structure

**Project:** The Final Hour (Hearts of Iron IV mod)
**Researched:** 2026-05-06
**Confidence:** HIGH — based on direct inspection of all existing project files plus authoritative training knowledge of Clausewitz engine conventions

---

## 1. Component Map

The mod is composed of six discrete component types. Each type has a defined responsibility and a defined set of cross-references to other types. Understanding these boundaries is the prerequisite to scaling cleanly from France (v1) to Germany (v2) and beyond.

```
┌─────────────────────────────────────────────────────────────────┐
│                        PLAYER ACTION                            │
│                    (starts a focus / takes a decision)          │
└───────────────┬─────────────────────────────────────────────────┘
                │
                ▼
┌──────────────────────────┐     ┌──────────────────────────────┐
│   FOCUS TREE             │────▶│   EVENTS                     │
│   common/national_focus/ │     │   events/                    │
│                          │     │                              │
│  • References: ideas     │     │  • Reference: ideas (add)    │
│  • References: events    │     │  • Reference: flags (set)    │
│  • References: flags     │     │  • Reference: decisions      │
│  • Sets: country flags   │     │    (visibility gating)       │
│  • Reads: country flags  │     │                              │
│  • Reads: decisions done │     │  Is triggered only = yes     │
└──────────────┬───────────┘     └──────────────────────────────┘
               │
               │ add_ideas =
               ▼
┌──────────────────────────┐     ┌──────────────────────────────┐
│   IDEAS / SPIRITS        │     │   DECISIONS                  │
│   common/ideas/          │     │   common/decisions/          │
│                          │     │                              │
│  • Defines modifiers     │     │  • Reference: events         │
│  • allowed = { tag }     │     │  • Reference: flags          │
│  • No outgoing refs      │     │  • Reference: focus          │
│    (ideas are endpoints) │     │    completion (visible)      │
│                          │     │  • Reads: ideas (available)  │
└──────────────────────────┘     └──────────────────────────────┘

┌──────────────────────────┐     ┌──────────────────────────────┐
│   ON_ACTIONS             │     │   BOOKMARKS                  │
│   common/on_actions/     │     │   common/bookmarks/          │
│                          │     │                              │
│  • on_startup: seeds     │     │  • References: country tags  │
│    initial ideas + flags │     │  • Sets: default country     │
│  • on_daily: guards      │     │  • Sets: starting ideology   │
│  • on_country_annexed    │     │  • Reads: nothing            │
│  • on_ruling_party_change│     │                              │
└──────────────────────────┘     └──────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                    LOCALISATION                                  │
│                    localisation/                                 │
│                                                                  │
│  Every string key in every other component resolves here.        │
│  One file per language. Language header determines which game    │
│  language loads the file. All components read loc silently —     │
│  missing keys display the raw key string as a visible bug.       │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                    GFX / INTERFACE                               │
│                    interface/*.gfx  +  gfx/                      │
│                                                                  │
│  Every icon = GFX_... and picture = GFX_... resolves here.       │
│  Missing keys fall back silently to generic icons (no crash).    │
└─────────────────────────────────────────────────────────────────┘
```

### Reference Direction Summary

| From | References | To |
|------|------------|----|
| Focus tree | `add_ideas =` | Ideas |
| Focus tree | `country_event =` | Events |
| Focus tree | `set_country_flag =` / `has_country_flag =` | Country flags (stateful) |
| Focus tree | `icon =` | GFX key → interface/*.gfx |
| Events | `add_ideas =` | Ideas |
| Events | `set_country_flag =` | Country flags |
| Decisions | `country_event =` | Events |
| Decisions | `has_completed_focus =` | Focus tree (one-way read) |
| Decisions | `icon =` | GFX key → interface/*.gfx |
| Ideas | `picture =` | GFX key → interface/*.gfx |
| Ideas | `allowed = { original_tag = }` | Country identity (no file) |
| On_actions | `add_ideas =` | Ideas |
| On_actions | `set_country_flag =` | Country flags |
| Bookmarks | country entries | Country tags (no file) |
| Localisation | nothing | (endpoint — no outgoing refs) |

**Key insight:** Ideas and Localisation are pure endpoints. They are referenced by everything but reference nothing themselves. This means they must be built before the components that depend on them, but they have zero coupling to those components — you can add a new idea without touching any focus tree.

---

## 2. Data Flow for a Typical Focus → Event → Idea Chain

This is the most common pattern in the mod, seen repeatedly in the existing code. Understanding it precisely prevents bugs when building new content.

```
1. PLAYER completes a focus
   e.g. FRA_industrial_recovery

2. FOCUS TREE fires in completion_reward:
   country_event = { id = fh_france.70 }
   — OR —
   add_ideas = FRA_industrial_recovery_spirit
   (direct add, no event intermediary)

3. If via event — EVENT fh_france.70 displays to player
   Player picks option A or B

4. OPTION EFFECT fires:
   add_ideas = FRA_industrial_recovery_spirit

5. IDEA is applied to country
   Modifiers go live immediately
   idea stays until removed by: remove_ideas, cancel = {}, or game end

6. DECISION becomes visible (optional further layer):
   fh_policy_industrial_recovery has:
     visible = { has_completed_focus = fh_fra_industrial_recovery }
   Player can re-trigger related event on cooldown
```

The existing code uses BOTH patterns: direct `add_ideas` in `completion_reward` (simpler), and event-mediated idea application (gives the player a narrative moment and a binary choice). The decision layer creates a repeatable mechanism for idea reinforcement.

**The three-layer architecture: Focus → Decision → Event** is what enables the Assemblée Nationale system in this mod. The focus UNLOCKS the decision category. The decision FIRES the event. The event APPLIES or REINFORCES the idea. This layering is intentional and correct.

---

## 3. Nation Separation: France vs Germany

### 3.1 The Correct Separation Model

Each nation gets its own file for every component type. Nothing is shared between nations at the file level. The engine merges all files in a directory at load time, so one nation's file never interferes with another's as long as IDs are properly prefixed.

**Target file structure for v1 (France) + v2 (Germany):**

```
common/
  national_focus/
    fh_france_focus.txt         ← France focus tree only
    fh_germany_focus.txt        ← Germany focus tree only (v2)
  ideas/
    fh_france_ideas.txt         ← France ideas only
    fh_germany_ideas.txt        ← Germany ideas only (v2)
  decisions/
    categories/
      fh_france_decision_categories.txt
      fh_germany_decision_categories.txt   ← (v2)
    fh_france_decisions.txt
    fh_germany_decisions.txt               ← (v2)
  on_actions/
    fh_on_actions.txt           ← SHARED — both nations' startup logic here
  bookmarks/
    fh_2026.txt                 ← SHARED — both FRA and GER entries here

events/
  fh_france_events.txt          ← France events only
  fh_germany_events.txt         ← Germany events only (v2)

localisation/
  fh_l_french.yml               ← ALL nations, ALL keys, French language
  fh_l_english.yml              ← ALL nations, ALL keys, English language

interface/
  fh_goals.gfx                  ← all nations' focus icons (or split by nation)
  fh_ideas.gfx                  ← all nations' idea icons
  fh_events.gfx                 ← all nations' event pictures
```

### 3.2 What Must Be Per-Nation (Never Shared)

- **Focus tree files** — each `focus_tree = { id = fh_X_focus ... }` block is one file. The `country = { factor = 0; modifier = { add = 10; tag = FRA } }` gating means the tree only activates for that nation. Never mix two nations' focuses in one file.
- **Ideas files** — enforced by `allowed = { original_tag = FRA }` in each idea block. Still, keep them in separate files for sanity. Finding all French ideas in a German ideas file is a maintenance nightmare.
- **Events files** — namespace scoping (`add_namespace = fh_france` vs `add_namespace = fh_germany`) is the isolation mechanism. Different namespaces, different files.
- **Decision files** — decision categories use `allowed = { original_tag = FRA }` and `visible = { original_tag = FRA }`. Keep them in separate files.

### 3.3 What Is Legitimately Shared

- **on_actions/fh_on_actions.txt** — This file handles startup logic for ALL modded nations. The existing code already shows this pattern: both FRA and GER blocks live in the same `on_actions = { on_startup = { effect = { FRA = { ... } GER = { ... } } } }` block. This is correct. Do not split on_actions by nation — the engine merges on_actions additively, so having two files both defining `on_startup` is fine, but one single-file approach is cleaner.
- **bookmarks/fh_2026.txt** — The bookmark describes the scenario start date and lists EVERY playable nation in one block. Both FRA and GER entries belong in this file.
- **localisation files** — One file per language contains ALL nations' keys. Do not create `fh_l_french_france.yml` and `fh_l_french_germany.yml` — this is unnecessary fragmentation. The engine loads every `.yml` file in `localisation/` that matches the active language header, so two files both starting with `l_french:` would both load. But keeping a single file per language is simpler to manage and avoids duplicate-key risks.

### 3.4 Naming Convention: The ID Firewall

The ID prefix is the nation-separation mechanism at the content level. Since all `.txt` files in a directory are loaded into the same engine namespace, the ONLY thing preventing `FRA_political_realign` from conflicting with `GER_political_realign` is the prefix. This is non-negotiable:

```
FRA_*    ← all France focus IDs, idea IDs, decision IDs, flag names
GER_*    ← all Germany focus IDs, idea IDs, decision IDs, flag names
fh_france.N   ← France event IDs (namespace scoped)
fh_germany.N  ← Germany event IDs (namespace scoped)
FH_*     ← mod-global identifiers (bookmark keys, cross-nation flags)
GFX_focus_FRA_*   ← France focus icons
GFX_focus_GER_*   ← Germany focus icons
GFX_idea_FRA_*    ← France idea icons
GFX_idea_GER_*    ← Germany idea icons
```

This convention is what makes it safe to add Germany without touching any France file.

---

## 4. How Focus Trees Reference Events and Ideas

### 4.1 Direct Idea Application (no event)

Use when the narrative moment does not require player interaction — the effect is automatic and unambiguous.

```
completion_reward = {
  add_ideas = FRA_industrial_recovery_spirit
  add_political_power = 15
}
```

The idea ID must be declared in `common/ideas/` before this fires. The engine resolves the ID at effect execution time (not at load time), so there is no strict file load order requirement — but if the idea is missing from the file, the effect silently fails.

### 4.2 Event-Mediated Application (player chooses)

Use when the focus completion creates a branching narrative moment. The event gives the player agency and provides a lore beat.

```
# In focus completion_reward:
completion_reward = {
  country_event = { id = fh_france.10 }
}

# In events/fh_france_events.txt:
country_event = {
  id = fh_france.10
  is_triggered_only = yes    ← MANDATORY for focus-triggered events

  option = {
    name = fh_france.10.a
    add_ideas = FRA_republican_spirit
  }
  option = {
    name = fh_france.10.b
    add_ideas = FRA_technocrat_spirit
    add_political_power = -20
  }
}
```

`is_triggered_only = yes` is mandatory. Without it, the engine will also roll the event as a random event against trigger conditions, potentially firing it at unexpected times.

### 4.3 The `select_effect` Field

Fires when the player QUEUES the focus (before completion). Use for narrative flavour or to set a flag that makes the focus visible to AI. Rare — most mods only use `completion_reward`.

```
focus = {
  id = FRA_diplomatic_offensive
  select_effect = {
    set_country_flag = FRA_diplomatic_push_started
  }
  completion_reward = {
    country_event = { id = fh_france.20 }
  }
}
```

### 4.4 The `available` Field

Gates whether a focus can be queued at all. Evaluated continuously while the focus is highlighted. If it becomes false after queueing, see `cancel_if_invalid`.

```
focus = {
  id = FRA_nato_commitment
  available = {
    is_in_faction = no
    stability > 0.4
  }
  completion_reward = { ... }
}
```

This is distinct from `prerequisite` (which requires a parent focus to be completed first). `available` checks game state conditions; `prerequisite` checks the focus tree graph.

---

## 5. How on_actions Works

### 5.1 Core Principle

`on_actions` is an additive merge file. Every mod that defines an `on_startup` block adds to it — none replace the others. The engine runs ALL on_startup effects when a new game starts. This is fundamentally different from most other file types where you add new named records.

The existing `fh_on_actions.txt` is structured correctly:

```
on_actions = {
  on_startup = {
    effect = {
      FRA = {
        add_ideas = FRA_esprit_francais
        set_country_flag = FRA_initialized
      }
      GER = {
        leave_faction = yes
      }
    }
  }
}
```

### 5.2 Use on_actions for:

- **Seeding initial ideas** — applying ideas that France/Germany should start with before any focus is taken
- **Setting initialization flags** — `set_country_flag = FRA_initialized` prevents re-seeding
- **Modifying starting state** — leaving factions, setting variables, adjusting ideology popularity at game start

### 5.3 Do NOT use on_actions for:

- **Content that belongs in history files** — if a nation should start with a specific unit template or technology, that belongs in `history/countries/FRA-France.txt` overrides. on_actions is for dynamic effects, not historical starting state.
- **Anything that should only happen once** — without a flag guard, on_startup fires every time a new game is loaded. The existing code correctly guards with `NOT = { has_idea = ... }` and `has_country_flag = FRA_initialized`.
- **Complex decision trees** — keep on_actions simple. Complex per-nation startup logic belongs in a triggered event (`is_triggered_only = yes`) that on_startup fires once.

### 5.4 Available on_actions hooks

The most useful hooks for this mod:

| Hook | When it fires | Use case |
|------|--------------|----------|
| `on_startup` | New game start | Seed initial ideas, flags |
| `on_daily` | Every in-game day | Guards, repeating checks |
| `on_ruling_party_change` | Ideology shifts | Swap ideas based on government type |
| `on_civil_war_end` | Civil war resolution | Restore ideas after split |
| `on_country_annexed` | Nation removed from map | Cleanup flags |

The existing code uses `on_startup` and `on_daily`. For this mod's scope, those two are sufficient for v1. `on_ruling_party_change` becomes valuable if political paths can result in ideology switches (e.g., France going fascist should potentially replace the starting `FRA_esprit_francais` spirit with a different one).

---

## 6. How Ideas / National Spirits Are Declared and Applied

### 6.1 Declaration Structure

All ideas live inside `ideas = { country = { ... } }` in files under `common/ideas/`. The `country` slot is for national spirits visible in the political tab. Other slots (`army`, `navy`, `air`, `designer`, etc.) are for advisors and companies — not relevant for national spirits.

```
ideas = {
  country = {
    FRA_esprit_francais = {
      picture = GFX_idea_FRA_esprit_francais   ← custom icon, or use a vanilla GFX_idea_generic_*
      allowed = {
        original_tag = FRA    ← ENGINE-enforced: this idea cannot be given to non-France
      }
      modifier = {
        stability_factor = 0.03
        research_speed_factor = 0.02
      }
      removal_cost = -1       ← -1 = cannot be manually removed through advisor UI
    }
  }
}
```

### 6.2 The `allowed` Field vs `available` Field

- `allowed = { original_tag = FRA }` — hard restriction. The idea simply cannot exist on a non-FRA country. Use for all nation-specific ideas.
- `available = { ... }` — soft condition for whether the idea shows in the advisor hire UI. Not relevant for spirits applied via effects (focus/event/on_actions bypass this).
- `cancel = { ... }` — condition that auto-removes the idea. Useful for ideas that should disappear if the country falls below a threshold (e.g., a war spirit that cancels when at peace).

### 6.3 Application Methods (all three routes exist in this mod)

1. **Focus `completion_reward`** — `add_ideas = FRA_idea_name` — fires on focus completion
2. **Event option effect** — `add_ideas = FRA_idea_name` — fires on player option selection
3. **on_actions** — `add_ideas = FRA_esprit_francais` in on_startup — seeds starting ideas

Removal: `remove_ideas = FRA_idea_name` in any effect block (focus, event, decision, on_actions).

### 6.4 Swapping Ideas (mutually exclusive spirits)

When one idea should replace another (e.g., picking a political path replaces the base spirit):

```
completion_reward = {
  remove_ideas = FRA_base_spirit
  add_ideas = FRA_left_coalition_spirit
}
```

This is cleaner than using `cancel` conditions, which can create timing issues if both ideas exist simultaneously for one tick.

---

## 7. Decisions vs Focus Tree Rewards

### 7.1 When to Use a Focus Completion Reward

Use focus rewards for one-time, permanent, narrative-significant effects:
- Applying a major national spirit for the first time
- Building construction grants
- Research slot additions
- Forming alliances or annexing territory
- Firing a major narrative event

Focus rewards fire exactly once, cannot be repeated, and are gated by the prerequisite tree. They are the backbone of the national narrative.

### 7.2 When to Use Decisions

Use decisions for repeatable, costly, player-controlled reinforcements:
- Re-applying or strengthening an idea that was first unlocked by a focus
- Minor political power expenditures that offer flavour events
- Time-gated recurring choices (the `days_re_enable` field)
- Actions the player can choose NOT to take (unlike focus completions which are automatic rewards)

The existing architecture is correct: focuses UNLOCK decisions (via `has_completed_focus = X` in the decision's `visible` block), then decisions give the player ongoing agency within the path they chose.

### 7.3 The `days_re_enable` Field

```
fh_policy_industrial_recovery = {
  days_re_enable = 180    ← decision becomes available again 180 days after completion
  cost = 35
  ...
}
```

This creates a soft repeating cycle: player pays 35 PP every 180 days to reinforce the industrial spirit. It is NOT a trigger-based event — it requires active player choice. This is appropriate for policy decisions. For AI play, add `ai_will_do = { factor = X }` with a non-zero weight to make the AI participate.

### 7.4 Decision Categories

The `common/decisions/categories/` file defines the tab that groups decisions in the UI. A decision that references a category that does not exist will crash the game (unlike missing GFX, which degrades gracefully). The category must exist before the decision file is loaded.

```
# categories/fh_france_decision_categories.txt
fh_france_assemblee = {
  icon = generic_political_actions
  priority = 50
  allowed = { original_tag = FRA }
  visible = { original_tag = FRA }
}

# decisions/fh_france_decisions.txt
fh_france_assemblee = {        ← category name must match
  fh_policy_industrial_recovery = { ... }
}
```

---

## 8. Bookmarks: How They Reference Starting Conditions

### 8.1 What a Bookmark Does

The bookmark defines the scenario that appears on the main menu's "Select a Scenario" screen. It does NOT change a country's history — that belongs in `history/countries/` files. What the bookmark controls:

- The start date (`date = 2026.1.1.12`)
- Which country is highlighted by default (`default_country = "FRA"`)
- The ideology shown for each playable country in the scenario description UI
- The flavour text ("history" field) shown per country in the selection screen

```
bookmarks = {
  bookmark = {
    name = "FH_2026_NAME"          ← localisation key for scenario name
    desc = "FH_2026_DESC"          ← localisation key for scenario description
    date = 2026.1.1.12
    picture = "GFX_select_date_1936"   ← scenario screen background image
    default_country = "FRA"
    default = yes

    FRA = {
      history = "FH_2026_FRA_DESC"     ← localisation key: France's blurb
      ideology = democratic
    }

    GER = {
      history = "FH_2026_GER_DESC"     ← localisation key: Germany's blurb
      ideology = democratic
    }
  }
}
```

### 8.2 Bookmark vs History File

The bookmark only controls the UI display. The actual starting state of France (which technologies they have, which leaders are in power, which ideas they start with) is controlled by:

1. `history/countries/FRA-France.txt` — vanilla file. To override, create this path in the mod and modify it. This is where initial leaders, political parties, and technologies are set.
2. `on_actions on_startup` — where this mod seeds its starting ideas (the existing approach).

The existing mod avoids modifying history files, using on_startup instead to apply initial ideas. This is the correct approach for a mod that wants to be compatible with existing save games and not break vanilla game starts for non-modded nations.

### 8.3 Adding Nations to the Bookmark

Adding Germany to v1 as a selectable (but non-focus-modded) nation requires ONLY adding the GER block to the bookmark. The GER entry in the existing bookmark already does this. No additional files are needed to make Germany "appear" as a selectable country — it already exists in vanilla with its vanilla focus tree.

For v2 (actual Germany focus tree), add `fh_germany_focus.txt` and the GER blocks in ideas, events, and decisions. The bookmark already has the GER entry — no bookmark changes needed for v2.

---

## 9. Build Order: What Must Exist Before What

This is the dependency graph for the v1 France implementation. Items earlier in the list must be created before items later in the list, because later items reference earlier ones.

### 9.1 Foundation Layer (no dependencies)

These files have no outgoing references and can be written in any order:

1. **`common/defines/00_fh_defines.lua`** — Already done. Sets START_DATE and END_DATE. No references to other mod files.

2. **`common/ideas/fh_france_ideas.txt`** — All idea IDs. Ideas reference GFX keys but missing GFX does not crash. Write ideas before focuses reference them (though engine resolves at runtime, not load time — the risk is silent failure if an idea ID is misspelled).

3. **`interface/fh_goals.gfx`** — Sprite declarations for all focus icons. No incoming references from other script files.

4. **`interface/fh_ideas.gfx`** — Sprite declarations for idea icons.

5. **`interface/fh_events.gfx`** — Sprite declarations for event pictures.

6. **`gfx/interface/goals/*.dds`** — Physical texture files. The `.gfx` declarations reference these paths.

### 9.2 Logic Layer (depends on Foundation)

7. **`events/fh_france_events.txt`** — Events reference idea IDs (from step 2) and GFX keys (from steps 3-5). Events are referenced BY the focus tree, but event files do not reference focus files.

8. **`common/decisions/categories/fh_france_decision_categories.txt`** — Category definitions. Must exist before the decision file that uses these category names.

9. **`common/on_actions/fh_on_actions.txt`** — References idea IDs (from step 2). Must be written after ideas are defined (or at least simultaneously, since engine resolves at runtime).

### 9.3 Top Layer (depends on both layers)

10. **`common/decisions/fh_france_decisions.txt`** — References: decision categories (step 8), events (step 7), focus completion (step 11, but only `has_completed_focus` which is a read-only check). EXCEPTION: the `visible = { has_completed_focus = X }` condition references focus IDs that are declared in the focus tree — write focus IDs in decisions after you have determined the focus ID namespace, even if the focus tree file is written after.

11. **`common/national_focus/fh_france_focus.txt`** — References everything: idea IDs (step 2), event IDs (step 7). The focus tree file is typically the last major file written because it pulls together all the content.

12. **`common/bookmarks/fh_2026.txt`** — References no mod files (country tags and localisation keys only). Can be written at any time, but depends on having the localisation keys defined.

### 9.4 Localisation (parallel track)

13. **`localisation/fh_l_french.yml`** and **`localisation/fh_l_english.yml`** — These can be written at any stage. Write localisation keys at the same time as you write the content they describe. Do not batch all localisation to the end — it becomes error-prone.

**Recommended practice:** Create the key skeleton (ID: "PLACEHOLDER") when you write each content file, then fill in the real text when the content design is finalized.

### 9.5 Dependency Summary Table

| File | Depends on |
|------|-----------|
| `defines/00_fh_defines.lua` | Nothing |
| `ideas/fh_france_ideas.txt` | GFX keys (soft — no crash if missing) |
| `interface/fh_goals.gfx` | `.dds` files in `gfx/` |
| `interface/fh_ideas.gfx` | `.dds` files in `gfx/` |
| `interface/fh_events.gfx` | `.dds` files in `gfx/` |
| `events/fh_france_events.txt` | Idea IDs, GFX keys |
| `decisions/categories/*.txt` | Nothing |
| `on_actions/fh_on_actions.txt` | Idea IDs |
| `decisions/fh_france_decisions.txt` | Event IDs, category IDs, focus IDs (read-only) |
| `national_focus/fh_france_focus.txt` | Idea IDs, event IDs |
| `bookmarks/fh_2026.txt` | Localisation keys, country tags |
| `localisation/*.yml` | Nothing (endpoint) |

---

## 10. Localisation Structure for Multiple Nations and Multiple Languages

### 10.1 File Organization

Use one file per language, containing all nations' keys. Do NOT split by nation.

```
localisation/
  fh_l_french.yml      ← l_french: header — all French-language strings
  fh_l_english.yml     ← l_english: header — all English-language strings
```

Adding Germany in v2 means appending GER_* keys to both existing files. No new files needed.

### 10.2 Internal Organization Convention

Within each localisation file, use comment blocks to separate nations. The engine ignores `#` comments.

```yaml
l_french:
 # ============================================================
 # FRANCE
 # ============================================================

 # Focus tree
 FRA_darkest_hour_europe: "L'heure la plus sombre d'Europe"
 FRA_darkest_hour_europe_desc: "..."

 # Ideas
 FRA_esprit_francais: "L'Esprit français"
 FRA_esprit_francais_desc: "..."

 # Events
 fh_france.10.t: "Congrès républicain"
 fh_france.10.d: "..."
 fh_france.10.a: "Stabiliser la coalition"
 fh_france.10.b: "Forcer le passage"

 # Bookmark
 FH_2026_NAME: "L'Heure Finale"
 FH_2026_DESC: "..."
 FH_2026_FRA_DESC: "La Ve République vacille..."
 FH_2026_GER_DESC: "La République fédérale fait face..."

 # ============================================================
 # GERMANY (v2)
 # ============================================================

 GER_grundgesetz_crisis: "..."
```

### 10.3 Key Naming Convention for Localisation

| Type | Key pattern | Example |
|------|------------|---------|
| Focus name | `NATION_focus_id` | `FRA_darkest_hour_europe` |
| Focus description | `NATION_focus_id_desc` | `FRA_darkest_hour_europe_desc` |
| Idea name | `NATION_idea_id` | `FRA_esprit_francais` |
| Idea description | `NATION_idea_id_desc` | `FRA_esprit_francais_desc` |
| Event title | `namespace.N.t` | `fh_france.10.t` |
| Event description | `namespace.N.d` | `fh_france.10.d` |
| Event option | `namespace.N.a/b/c` | `fh_france.10.a` |
| Decision name | `NATION_decision_id` | `FRA_assemblee_budget_deal` |
| Decision description | `NATION_decision_id_desc` | `FRA_assemblee_budget_deal_desc` |
| Bookmark name | `FH_YEAR_NAME` | `FH_2026_NAME` |
| Bookmark country blurb | `FH_YEAR_NATION_DESC` | `FH_2026_FRA_DESC` |

### 10.4 Critical Encoding Requirement

Both `.yml` files MUST begin with the UTF-8 BOM (`EF BB BF`). Without it, the entire file is silently ignored. Verify:

```bash
hexdump -C localisation/fh_l_french.yml | head -1
# Must show: ef bb bf 6c 5f 66 72 65 6e 63 68 3a 0a  (BOM + "l_french:")
```

The existing files may be missing the BOM if created without explicit BOM encoding. This is the single most common silent bug in HoI4 modding.

### 10.5 Current English File Issue

The existing `localisation/fh_l_english.yml` contains identical text to `fh_l_french.yml` — French text under an English language header. This is a known alpha-state placeholder. For v1, all English keys must be translated to English. The architecture supports this — it is purely a content gap, not a structural problem.

---

## 11. Scalability: Adding Nations Without Breaking France

### 11.1 The Non-Interference Guarantee

The file-per-nation structure plus the ID prefix convention means adding Germany requires ZERO changes to France files. Specifically:

- `fh_france_focus.txt` — not touched
- `fh_france_ideas.txt` — not touched
- `fh_france_events.txt` — not touched
- `fh_france_decisions.txt` — not touched

Only two files must be modified when adding Germany:
- `localisation/fh_l_french.yml` — append GER_* keys
- `localisation/fh_l_english.yml` — append GER_* keys
- `common/on_actions/fh_on_actions.txt` — append GER startup logic (already has GER block)

Everything else for Germany is new files, not modifications to existing ones.

### 11.2 The on_actions Expansion Pattern

When v2 ships, expand the on_actions GER block (it currently only removes GER from factions on startup). Add idea seeding:

```
GER = {
  if = {
    limit = { NOT = { has_idea = GER_grundgesetz_spirit } }
    add_ideas = GER_grundgesetz_spirit
  }
  if = {
    limit = { is_in_faction = yes }
    leave_faction = yes
  }
  set_country_flag = GER_initialized
}
```

### 11.3 The Bookmark Expansion Pattern

The existing bookmark already has both FRA and GER entries. For v2, no change to the bookmark file is needed unless the GER description text needs updating (localisation key change only).

### 11.4 Adding a Third Nation (v3+)

For any additional nation (e.g., USA, RUS), the pattern is:
1. Add new files: `fh_usa_focus.txt`, `fh_usa_ideas.txt`, `fh_usa_events.txt`, `fh_usa_decisions.txt`, `fh_usa_decision_categories.txt`
2. Expand: `fh_on_actions.txt` (USA startup block), `fh_l_french.yml` (USA keys), `fh_l_english.yml` (USA keys), `fh_2026.txt` (USA bookmark entry)
3. Add GFX files or expand existing `.gfx` files

The only files that scale with number of nations are the shared ones (on_actions, localisation, bookmark). All other content is fully isolated per nation.

---

## 12. Critical Architecture Decisions for Reconstruction

### 12.1 Problem: Mixed Prefixes in Existing Code

The current code mixes `fh_fra_` (focus/idea IDs), `fh_france_` (decision/category IDs), and `fh_` (some idea IDs like `fh_union_europeenne`, `fh_euro_spirit`, `fh_otan_spirit`). This creates three separate prefix namespaces for what is semantically one nation.

**The three-way inconsistency causes real problems:**
- `fh_union_europeenne` is an idea that seems mod-global (no nation prefix) but is `allowed = { original_tag = FRA }`. When Germany gets a European Union idea, it will need a different ID but a visually confusing similar name.
- `fh_france.N` event namespace is fine (it is already scoped).
- The reconstruction must enforce: all France-specific IDs use `FRA_` prefix, all Germany-specific IDs use `GER_` prefix, all genuinely mod-global IDs use `FH_` prefix.

### 12.2 Recommendation: Dedicated GFX Files Per Nation

For v1, a single `fh_goals.gfx` can hold all France focus icons. For v2+, split into:

```
interface/
  fh_goals_fra.gfx    ← France focus icons
  fh_goals_ger.gfx    ← Germany focus icons (v2)
  fh_ideas_fra.gfx    ← France idea icons
  fh_ideas_ger.gfx    ← Germany idea icons (v2)
  fh_events.gfx       ← all event pictures (shared, fewer, less need to split)
```

This prevents a single large `.gfx` file from becoming unmaintainable.

### 12.3 Recommendation: on_actions Flag Guard Pattern

Every nation block in on_startup must use an initialized flag guard. The existing France block does this correctly. The Germany block in the existing code does NOT set an initialization flag — it only calls `leave_faction`. When Germany gets full content in v2, the flag guard pattern must be added:

```
GER = {
  if = {
    limit = { NOT = { has_country_flag = GER_initialized } }
    add_ideas = GER_starting_spirit
    set_country_flag = GER_initialized
  }
}
```

Without the flag guard, on_startup effects re-apply every time a new game is started (not just once per campaign). With is_retriggerable effects like `leave_faction`, double-execution is harmless. With `add_ideas`, double-execution creates duplicate ideas in the national spirit slot, which breaks modifier stacking.

### 12.4 Recommendation: History File vs on_actions for Starting Ideas

The existing approach (seeding France's starting idea via on_startup) is the right call for a mod that does not want to ship `history/countries/FRA-France.txt` overrides. However, if the mod later needs to override France's starting leader, technologies, or political parties (likely for a 2026 scenario), `history/countries/` files become necessary. When that happens, move idea seeding INTO the history file and remove it from on_actions to avoid double-application.

This is not a v1 concern but should be designed for: keep the `FRA_initialized` flag system so that moving the seeding to a history file means simply removing the on_startup block without changing any other file.

---

## Sources

- Direct inspection of all project files: `common/national_focus/fh_france_focus.txt`, `common/ideas/fh_france_ideas.txt`, `events/fh_france_events.txt`, `common/decisions/fh_france_decisions.txt`, `common/decisions/categories/fh_france_decision_categories.txt`, `common/on_actions/fh_on_actions.txt`, `common/bookmarks/fh_2026.txt`, `common/defines/00_fh_defines.lua`, `localisation/fh_l_french.yml`, `localisation/fh_l_english.yml`
- Authoritative training knowledge of Paradox Clausewitz engine architecture (HoI4 stable since v1.9.x for focus tree syntax; HIGH confidence)
- Community-established patterns from Road to 56, Kaiserreich, The New Order (all publicly available)
- Note: paradoxwikis.com and WebSearch were inaccessible during this session (tool permission restrictions). All findings are grounded in direct project file inspection and well-established modding conventions for this domain.
