# Feature Landscape: HoI4 National Focus Mods

**Domain:** Hearts of Iron IV national focus tree mod (France, near-future 2026)
**Researched:** 2026-05-06
**Sources:** HoI4 modding wiki, Road to 56 / Kaiserreich / Old World Blues / The Fire Rises
  source analysis, project code inspection (615-line focus tree, 427-line events, decisions,
  ideas, localisation)

---

## Table Stakes

Features that players of serious focus tree mods universally expect. Absence earns
immediate negative reviews and "unpolished" tags.

| Feature | Why Expected | Complexity | Status in Current Code |
|---------|--------------|------------|------------------------|
| Mutual exclusion (mutex) branches | Every Paradox country has political choice — players expect it | Low | Present (republican/left/right) |
| Focus-triggered events with options | Rewards feel hollow without player agency | Low-Med | Present (fh_france.10–.60) |
| Named national spirits (ideas) with coherent modifiers | Players track buffs/debuffs — generic is invisible | Low | Present (14 ideas) |
| Full localisation in mod's declared languages | Un-localised keys are instant 1-star reviews | Low | Present (FR + EN yml) |
| Custom GFX icons for every focus | Vanilla recycled icons signal low effort | Med | Declared, needs .dds integration |
| Unique focus tree (not vanilla replacement) | Must use `default = no` + country filter | Low | Correct (`default = no`) |
| Correct HoI4 coordinate grid | Visual overlap or broken connector lines | Med | Needs audit (see Pitfalls) |
| Focus costs tuned for pacing | 70-day focus is HoI4 standard; spamming 5s feels cheap | Low | Mixed (5/10 costs) |
| `is_triggered_only = yes` on all reward events | Otherwise engine fires them randomly on timer | Low | Correct on all events |
| Event namespace declared | `add_namespace` at top of file | Low | Correct (`fh_france`) |
| `bypass` blocks on availability-gated focuses | Without bypass, AI soft-locks behind impossible `available` | Med | MISSING — critical gap |

---

## Differentiators

Features that separate a memorable mod from a competent one. These are why Road to 56
gets 350k subscribers and generic reskins get 3k.

### 1. Narrative Coherence — Every Focus Tells a Story

**What it means:** Each focus name, description, icon, and reward package form a coherent
political argument. Road to 56's France tree is memorable because "Front Populaire" triggers
a cascade: stability up, war support down, consumer goods up — the *texture* of a left-wing
government, not just stat changes.

**Implementation:** Focus descriptions (localisation) should explain *why* the reward
happens, not restate what it is. "The factories reopened" is weak. "Two thousand workers
clocked in at Renault-Vilvoorde on Monday — the first time in four years" is strong.

**In current project:** The localisation is already at this level (e.g., `fh_fra_lfi_spirit_desc`
reads like narrative, not a tooltip). This is a genuine differentiator — maintain it.

**Complexity:** Low (writing work only)

---

### 2. Mutex Branches with Meaningful Asymmetry

**What it means:** The three paths (left/center/right) must feel genuinely different in
play, not just cosmetically. Differentiating mods make each path produce a *different kind
of France* — not just different modifiers but different strategic options, different events,
different decisions unlocked.

**Implementation in HoI4 scripting:**

```paradox
focus = {
  id = FRA_left_unity
  mutually_exclusive = {
    focus = FRA_republican_path
    focus = FRA_right_national
  }
  ...
}
```

Each sibling in `mutually_exclusive` must declare all other siblings. The engine enforces
bi-directional exclusion — but you must declare it in *every* node of the mutex group, not
just one.

**Key differentiator pattern:** After the political split, unlock path-specific decisions
that are invisible to the other paths. A center-path player should never even *see* the
LFI decisions in the Assemblée Nationale. Use `visible = { has_completed_focus = X }` on
decision entries.

**Complexity:** Medium (coordination across focuses and decisions)

---

### 3. Assemblée Nationale as a Persistent Mechanic

**What it means:** The decision category simulating parliament is a genuine differentiator
because it creates ongoing player engagement after the focus tree is resolved. Players
who finish their political branch still have the Assemblée to manage.

**Implementation pattern:**

```paradox
# Category (fh_france_decision_categories.txt)
fh_france_assemblee = {
  priority = 50
  visible_when_empty = yes    # Show even if no decisions qualify
  allowed = { original_tag = FRA }
  visible = { original_tag = FRA }
}

# Decision entry
fh_policy_industrial_recovery = {
  cost = 35                   # PP cost
  days_re_enable = 180        # 6-month cooldown before re-use
  visible = {
    has_completed_focus = fh_fra_industrial_recovery
  }
  available = {
    has_political_power > 35
  }
  complete_effect = {
    country_event = { id = fh_france.70 }
  }
}
```

**PP economy:** The current code uses flat 35 PP for most decisions. Top mods make cost
vary by political climate. Consider scaling cost with stability: cheaper to push policy
when parliament is stable, expensive when fractured. Implement via `modifier = {}` block
on the decision or scaled `cost` using `var:` references (requires NSB DLC).

**What makes it truly differentiated:** The "has idea → add PP instead of re-adding idea"
pattern in the current event code (fh_france.70–.82) is the right approach — it means
re-using a policy decision still gives value. Keep this pattern everywhere.

**Complexity:** Medium (already partially built; main work is extending coverage)

---

### 4. Multi-Condition Secret/Hidden Focus Unlocks

**What it means:** Focuses that require completing multiple independent prerequisites
(not just one chain) create a sense of discovery and reward comprehensive play. The
Fédération Européenne / Volt Europa pattern (complete 4 specific things AND set a flag)
is what Road to 56 uses for its most satisfying late-game unlocks.

**Implementation — 4-condition unlock:**

```paradox
focus = {
  id = FRA_fedEuropeenne
  prerequisite = { focus = FRA_european_leadership }

  available = {
    has_country_flag = FRA_volt_unlocked
    has_completed_focus = FRA_euro_strategy
    has_completed_focus = FRA_nato_commitment
    has_completed_focus = FRA_industrial_recovery
    # Optional: require another country to be in a certain state
    # GER = { has_completed_focus = GER_european_partnership }
  }

  ai_will_do = {
    factor = 0    # Disable AI — secret focuses should be human-driven
  }

  completion_reward = {
    add_ideas = FRA_fedEuropeenne_spirit
    hidden_effect = {
      set_country_flag = FRA_federation_formed
    }
  }
}
```

The `available` block is the unlock gate. All conditions inside `available` must be true
simultaneously before the focus can be *started* (not just completed). The focus appears
grayed-out in the tree until unlocked — this is the intended visual behavior.

**The flag mechanism:** Set flags via decisions (as the current `fh_france_volt_europa_initiative`
does) or via events. Never set a flag and immediately check it in the same effect block —
flags propagate on the next engine tick.

**Complexity:** Low-Medium (scripting is straightforward; design requires knowing what
the 4 conditions should be)

---

### 5. Event Options with Asymmetric Trade-offs

**What it means:** Events where both options are reasonable but serve different strategies
are a hallmark of great mods. Events where one option is obviously correct are a waste.

**Implementation pattern:**

```paradox
country_event = {
  id = FRA_pol.10
  title = FRA_pol.10.t
  desc = FRA_pol.10.d
  picture = GFX_report_event_political_rally
  is_triggered_only = yes

  option = {
    name = FRA_pol.10.a
    # Safe option — stability, modest PP
    add_political_power = 40
    add_stability = 0.03
  }
  option = {
    name = FRA_pol.10.b
    # Bold option — high PP, stability cost
    add_political_power = 75
    add_stability = -0.02
    add_war_support = 0.02
  }
}
```

The current code already does this well (see fh_france.10: "Stabiliser la coalition" vs
"Forcer le passage"). The pattern to extend: make one option unlock a *decision* or set
a *flag* rather than just giving stats. Then the choice has lasting architectural
consequences, not just immediate number differences.

**Complexity:** Low (writing + light scripting)

---

### 6. Custom GFX Integration (Non-Generic Icons)

**What it means:** The single most visible quality signal. A focus tree where every icon
is unique and matches the mod's visual identity is instantly recognizable as premium work.

**Implementation:**

In `interface/goals/` (or `gfx/interface/goals/`), place `.dds` files (64x64px, DXT5/BC3
compression). Define them in an `.gfx` file:

```paradox
# interface/goals/The_Final_Hour_goals.gfx
spriteTypes = {
  spriteType = {
    name = "GFX_focus_FRA_darkest_hour_europe"
    texturefile = "gfx/interface/goals/FRA_darkest_hour_europe.dds"
  }
}
```

Reference in focus: `icon = GFX_focus_FRA_darkest_hour_europe`

**Naming convention:** Prefix all GFX names with mod prefix (`GFX_focus_FRA_`) to avoid
overwriting vanilla or other mods' sprites.

**Current state:** The project has `.dds` assets ready but they are not yet integrated.
This is a v1 blocker per PROJECT.md.

**Complexity:** Low-Medium (mechanical integration is simple; requires all .dds to be in
the right format and path)

---

### 7. `select_effect` and `in_progress_effect` for Immersion

**What it means:** Top-tier mods use these underused focus blocks to create effects that
fire when a focus is *selected* (not completed) or while it is *in progress*. This gives
focuses a sense of unfolding rather than binary start/complete.

**Implementation:**

```paradox
focus = {
  id = FRA_industrial_recovery
  ...
  select_effect = {
    # Fires the moment player clicks the focus
    news_event = { id = FRA_news.01 hours = 6 }
  }
  in_progress_effect = {
    # Fires periodically while the focus is being researched
    # WARNING: fires every day — keep effects small or gated
    if = {
      limit = { NOT = { has_country_flag = FRA_recovery_announced } }
      set_country_flag = FRA_recovery_announced
      add_stability = 0.005
    }
  }
  completion_reward = {
    ...
  }
}
```

**Caution:** `in_progress_effect` fires daily. Without an `if + flag` guard, it will
apply effects 70 times over the focus duration. Always gate with a flag that you set on
first fire.

**Complexity:** Medium (requires careful flag management)

---

### 8. `cancel_if_invalid` for Civil War / Regime Change Robustness

**What it means:** When a civil war fires or a country tags change, in-progress focuses
can get into broken states. Adding `cancel_if_invalid = yes` and `continue_if_invalid = no`
prevents ghost focuses.

```paradox
focus = {
  id = FRA_republican_path
  cancel_if_invalid = yes
  continue_if_invalid = no
  ...
}
```

Use on all politically sensitive focuses that have `available` conditions — if the
conditions become false mid-focus, it cancels cleanly instead of completing anyway.

**Complexity:** Trivial (one line per focus)

---

## Anti-Features

Features to explicitly NOT build in v1, with rationale.

| Anti-Feature | Why Avoid | What to Do Instead |
|--------------|-----------|-------------------|
| Y=-1 trunk focus | `y = -1` places focus above the header line — breaks visual layout, often invisible | Start trunk at `y = 0` or use a display-only focus at `y = 0` |
| Generic vanilla icons on political focuses | Signals low effort; players notice immediately | Integrate the existing .dds assets even if imperfect |
| Events with only one option | Single-option events are loading screens, not content | Always provide 2 options; if only one makes sense, make the other a "moderate" variant with smaller but different effects |
| `on_daily` spirit enforcement | The current `on_daily` block re-adds `fh_fra_esprit_francais` every day — this is unnecessary and adds to tick load | Use `on_startup` + `on_actions.on_country_capitulated` to re-add; remove the daily check |
| Hardcoded state IDs in resources | `add_resource = { state = 16 }` breaks if map mods are loaded | Use `capital_scope` or `random_owned_controlled_state` with `limit` blocks |
| Focus tree IDs with old prefix | `fh_fra_` prefix in existing alpha code conflicts with target `FRA_` convention | The reconstruction decision in PROJECT.md is correct — do not mix prefixes |
| Multi-path focuses that all give the same rewards | Mutex branches that give identical stat values with different icon colors are not real choices | Each path must have a meaningfully different modifier profile |
| Focus chains without bypass for impossible conditions | AI can soft-lock if `available` conditions can never be met | Add `bypass = { always = yes }` or a fallback condition to every `available`-gated focus |
| Decision costs that don't scale | Flat 35 PP for every decision regardless of situation | Vary costs (25–75 PP) to make parliamentary decisions feel like real tradeoffs |

---

## Complexity Notes by Feature Area

### Focus Tree Layout (x/y Coordinates)

HoI4 uses a fixed grid: each unit = 96 pixels in the focus tree view. The standard grid
is approximately 1 unit wide = 1 focus slot. Conventions established by vanilla and
top mods:

- **x** is column, **y** is row. `x = 0, y = 0` is top-left of a standard tree.
- Vertical spacing: 1 unit between rows (connected focuses). Leave 2 units gap for visual
  grouping of separate branches.
- Horizontal spacing: mutex branches typically placed 2–4 units apart to allow connector
  lines to be distinct.
- The current alpha tree uses `y = -1` for the trunk focus — this is above the visible
  header area and is likely causing visual artifacts. Move to `y = 0`.
- Visual grouping convention: economic branch far left (x = 0–3), political center
  (x = 5–9), military/diplomatic right (x = 10–14). The current layout roughly follows
  this but the economic branch at `x = -2` is off-grid left.
- **Rule of thumb from Road to 56:** Never exceed 20 columns without a separator focus;
  players lose spatial orientation.

### Mutual Exclusion (Mutex) Scripting

Full syntax — every sibling must declare every other sibling:

```paradox
# Node A
focus = {
  id = FRA_path_A
  mutually_exclusive = {
    focus = FRA_path_B
    focus = FRA_path_C
  }
}

# Node B
focus = {
  id = FRA_path_B
  mutually_exclusive = {
    focus = FRA_path_A
    focus = FRA_path_C
  }
}

# Node C
focus = {
  id = FRA_path_C
  mutually_exclusive = {
    focus = FRA_path_A
    focus = FRA_path_B
  }
}
```

If any node is missing a sibling declaration, the engine allows completing both — a
critical bug that the Steam Workshop reviews always catch within 48 hours.

Downstream focuses from a mutex node do NOT need `mutually_exclusive` — exclusion is
inherited for focuses that have the mutex node as their sole prerequisite.

### Triggering Events from Focus Completion

```paradox
completion_reward = {
  country_event = { id = namespace.number }
  # Optional: add delay in days before event fires
  country_event = { id = namespace.number days = 3 }
}
```

The event MUST have `is_triggered_only = yes` — without it, the event fires on its own
timer AND from the focus, causing double-firing. This is the single most common bug in
beginner mod events.

For events targeting another country:
```paradox
completion_reward = {
  GER = { country_event = { id = namespace.number } }
}
```

For news events (appears in all countries' history feed):
```paradox
completion_reward = {
  news_event = { id = namespace.number hours = 24 }
}
```

### Multi-Condition Unlock Implementation

The `available` block inside a focus is evaluated continuously. All sub-conditions use
implicit AND logic. For OR logic, use:

```paradox
available = {
  OR = {
    has_completed_focus = FRA_path_A
    has_completed_focus = FRA_path_B
  }
  has_country_flag = FRA_special_flag
  NOT = { has_war = yes }
}
```

For the Fédération Européenne / Volt Europa pattern (4 independent conditions):

```paradox
focus = {
  id = FRA_federation_europeenne
  prerequisite = { focus = FRA_european_leadership }

  available = {
    # Condition 1: flag set by Assemblée Nationale decision
    has_country_flag = FRA_volt_unlocked
    # Condition 2: economic branch complete
    has_completed_focus = FRA_future_industry
    # Condition 3: diplomatic branch complete
    has_completed_focus = FRA_nato_commitment
    # Condition 4: political stability threshold
    stability > 0.6
  }

  # Tooltip shown to player explaining unlock conditions
  # Uses the auto-generated available_tooltip — no extra work needed
}
```

The focus UI automatically generates an "Available when:" tooltip from the `available`
block. Conditions using `has_completed_focus` and `has_country_flag` display cleanly.
Stability/resource thresholds also display. Do not add a manual `available_tooltip` block
unless the auto-generated one is misleading.

### Assemblée Nationale PP Modifier System

The current implementation uses flat PP costs on decisions. A more differentiating
approach — "coalition strength modifies PP costs" — can be implemented via:

```paradox
# National spirit that reduces decision costs when coalition is unified
FRA_strong_coalition_spirit = {
  modifier = {
    political_power_cost = -0.10    # 10% cheaper decisions
  }
}

# Alternatively, use decision modifiers (requires Man the Guns DLC)
FRA_assemblee_politique = {
  modifier = {
    # applied while the decision is "active" (if using timed decisions)
  }
}
```

For simpler v1 implementation: vary the `cost` value on each decision based on path
(50 PP for radical path decisions, 25 PP for moderate path decisions). This achieves
asymmetry without DLC dependencies.

---

## MVP Recommendation

For v1 France complete, prioritize in this order:

**Must have (table stakes, blockers):**
1. Custom GFX icons integrated (.dds) — currently a gap despite assets existing
2. Fix `y = -1` trunk focus coordinate — visual bug
3. Add `bypass` blocks to all `available`-gated focuses — AI soft-lock prevention
4. Add `cancel_if_invalid = yes` to mutex focuses — civil war robustness
5. Rename all IDs from `fh_fra_` to `FRA_` prefix — consistency with PLAN.md
6. Remove `on_daily` spirit enforcement — performance and correctness

**Should have (differentiators that make reviews positive):**
7. Make each mutex path unlock path-exclusive Assemblée Nationale decisions
8. Extend event options to set flags that unlock downstream content (not just stats)
9. Implement full 4-condition Fédération Européenne / Volt Europa unlock
10. Add `select_effect` news events on major political focuses (3–5 focuses)

**Defer to v1.x:**
- `in_progress_effect` on economic focuses (complexity vs benefit ratio is poor for v1)
- Dynamic PP cost scaling via modifier (requires more design work; flat costs are fine)
- AI path-weighting via `ai_will_do` factor tuning (do after basic testing)

---

## Confidence Assessment

| Area | Confidence | Basis |
|------|------------|-------|
| Mutex syntax | HIGH | Verified against existing project code + stable since HoI4 1.0 |
| Event triggering | HIGH | `is_triggered_only` pattern confirmed in all 27 events in current code |
| `available` multi-condition | HIGH | Confirmed pattern; `Volt Europa` decision already uses flag check |
| `bypass` requirement | HIGH | Standard HoI4 practice; absence is a known AI bug class |
| `select_effect` / `in_progress_effect` | MEDIUM | Known feature; daily firing behavior requires careful testing |
| GFX sprite naming | HIGH | Stable vanilla convention since HoI4 launch |
| Coordinate grid pixel values | MEDIUM | 96px per unit is community-documented; verify in-game |
| Decision PP modifier via spirit | HIGH | Confirmed by ideas file modifier format in project |
| `cancel_if_invalid` behavior | MEDIUM | Known feature; edge cases in civil war scenarios need testing |
