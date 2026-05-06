# HoI4 Modding Pitfalls

**Project:** The Final Hour
**Researched:** 2026-05-06
**Scope:** National focus trees, events, localisation, GFX, Steam Workshop
**Confidence:** HIGH — sourced from direct code audit of the project plus established HoI4 modding mechanics

---

## CRITICAL PITFALLS

Mistakes that cause silent failures, crashes, or full rewrites.

---

### Pitfall 1: Missing UTF-8 BOM in Localisation Files

**What goes wrong:** HoI4 requires all `.yml` localisation files to be saved as UTF-8 **with BOM** (Byte Order Mark: `EF BB BF`). A file saved as plain UTF-8 without BOM will cause HoI4 to silently ignore the entire file. All localised strings display as their raw key (e.g. `FRA_focus_title` instead of the text). No error is logged.

**Why it happens:** Most editors (VS Code, Sublime, vim) save UTF-8 without BOM by default. BOM-less UTF-8 is correct everywhere except HoI4 and a few other Paradox games.

**Consequences:** Every single localisation key in the file shows as raw text in-game. Affects focus names, event titles, idea tooltips — the entire mod looks broken.

**Current project status:** GOOD. Both `localisation/fh_l_french.yml` and `localisation/fh_l_english.yml` are confirmed UTF-8 with BOM (`file` command output: "Unicode text, UTF-8 (with BOM) text"). This must be maintained on every new file created.

**Prevention:**
- VS Code: install the "Fix File Encoding" extension or configure `"files.encoding": "utf8bom"` in workspace settings for `*.yml` files specifically.
- When creating new localisation files, always verify with: `file localisation/yourfile.yml` — output must say "with BOM".
- Never copy-paste localisation file contents to a new file without re-adding the BOM.

**Warning signs:** Any localisation key appearing as its raw ID string in-game instead of readable text.

**Phase to address:** Phase 1 (infrastructure setup). Establish the encoding convention before writing a single key.

---

### Pitfall 2: `l_french:` / `l_english:` Header Must Be First Line (After BOM)

**What goes wrong:** The language tag (`l_french:` or `l_english:`) must appear on the very first line of the file (immediately after the invisible BOM bytes). A blank line before it, a space, or a UTF-8 BOM that the editor renders as a visible character will break parsing. HoI4 will not load the file.

**Why it happens:** The parser is extremely strict about this. A blank line #1 with `l_french:` on line #2 means zero keys are loaded.

**Current project status:** GOOD. Both files start directly with `l_french:` / `l_english:` on line 1.

**Prevention:** Never insert blank lines at the top of localisation files. If the file is regenerated or reformatted, verify line 1 is the language header.

**Warning signs:** All keys missing in-game even though the file has BOM and looks syntactically correct.

**Phase to address:** Phase 1 alongside BOM setup.

---

### Pitfall 3: `FRA_` Prefix Colliding with Vanilla Base Game Entries

**What goes wrong:** Vanilla HoI4 uses `FRA_` as the prefix for France's built-in focuses, ideas, and decisions (e.g. `FRA_the_maginot_line`, `FRA_recruit_the_foreign_legion`). If your mod defines a focus or idea whose ID exactly matches a vanilla entry, the game merges or overrides it unpredictably — sometimes silently winning, sometimes causing the vanilla tree to not render.

**Why it happens:** HoI4 loads all files in a folder and last-loaded wins for identical keys. Since your mod folder loads after vanilla, your IDs beat vanilla ones — but this means you may accidentally overwrite vanilla content for other countries or global ideas.

**Current project status:** MODERATE RISK. The project uses `fh_fra_` for most IDs (e.g. `fh_fra_darkest_hour_europe`, `fh_fra_industrial_recovery_spirit`) and `fh_france_` for decisions/categories. However, the focus tree ID is `fh_france_focus` and some idea IDs are generic (`fh_union_europeenne`, `fh_euro_spirit`, `fh_otan_spirit`) — these are not in vanilla's `FRA_` namespace, so collision risk is low but should be audited.

**The real risk for this project:** When rebuilding to use the `FRA_` prefix (as stated in PROJECT.md), any focus whose new ID matches a vanilla French focus ID will silently override vanilla content. For example, `FRA_european_leadership` would conflict if vanilla has such an entry.

**Prevention:**
- Always use a mod-specific prefix that vanilla does not use. `FH_FRA_` or `fh_fra_` are safe. Pure `FRA_` with generic suffixes is dangerous.
- Before assigning any ID, check vanilla's `Hearts of Iron IV/common/national_focus/00_france.txt` to verify there is no collision.
- The `replace_path` mechanism in `descriptor.mod` is the correct way to suppress vanilla content (see Pitfall 9).

**Warning signs:** Vanilla France focus tree partially appears alongside your mod's tree, or a vanilla focus disappears for reasons you didn't intend.

**Phase to address:** Phase 1 (naming convention decision). Locked in before any focus IDs are written.

**Recommendation:** Do NOT use bare `FRA_` prefix. Use `FH_FRA_` or keep `fh_fra_` (lowercase). The PROJECT.md decision to use `FRA_` is a risk — clarify this means something like `FRA_FH_...` or better `fh_fra_...` which the current code already does correctly.

---

### Pitfall 4: Focus Tree x/y Coordinate System Misunderstanding

**What goes wrong:** HoI4 focus positions are specified in grid units, not pixels. Each unit is 96px wide and 130px tall. The coordinate system is relative — `x = 0` means the leftmost visible column, not an absolute screen position. Negative `y` values (like `y = -1` in the current trunk focus) place the focus above the first row and can render partially off-screen or be cut off depending on the interface scroll position.

**Current project status:** RISK IDENTIFIED. The trunk focus `fh_fra_darkest_hour_europe` uses `y = -1`. This is unusual — most trees start at `y = 0`. The `y = -1` focus sits above the first row and players may not see it unless they scroll up. In vanilla trees, `y = -1` is used specifically for "always visible" header-style focuses, but they still render oddly on some resolutions.

**The layout breadth problem:** The current tree spans from `x = -2` (economy branch) to `x = 12` (France Souveraine). That is 15 columns wide (14 units). A standard HoI4 tree displays approximately 13-15 columns comfortably. At this width, foci at the edges (x = -2 and x = 12) will require horizontal scrolling and may feel disconnected. This is a UX issue, not a crash.

**`relative_position_id` pattern:** When a subtree has many nodes, using `relative_position_id` + local offsets makes layout changes non-destructive. The current code uses absolute `x/y` everywhere — fine for now, but will become painful if the tree grows.

**Prevention:**
- Start all trees at `y = 0` unless `y = -1` is intentionally used for a decorative/always-visible header.
- Design column layout on paper first: left edge = 0, count right. Keep total width under 25 columns (the horizontal scroll range before it becomes uncomfortable).
- Test the layout at 1080p — it is the minimum common resolution among Steam players.

**Warning signs:** Focuses visually overlap, or the tree requires horizontal scrolling to see both the leftmost and rightmost branches simultaneously.

**Phase to address:** Phase 1 (focus tree architecture design) and Phase 2 (first in-game test pass).

---

### Pitfall 5: `mutually_exclusive` Asymmetry Breaks the Focus Tree

**What goes wrong:** Every `mutually_exclusive` relationship must be declared symmetrically. If Focus A declares `mutually_exclusive = { focus = B }` but Focus B does not declare `mutually_exclusive = { focus = A }`, the game may allow the player to complete both focuses — the lock only works one way.

**Current project status:** RISK PRESENT. Reviewing the current tree:

- `fh_fra_republican_path` declares mutual exclusivity with both `fh_fra_left_unity` and `fh_fra_right_national`. Both of those declare it back correctly. GOOD.
- `fh_fra_macron_path` declares `mutually_exclusive = { focus = fh_fra_le_pen_path }`. `fh_fra_le_pen_path` declares `mutually_exclusive = { focus = fh_fra_macron_path }`. GOOD.
- `fh_fra_socialist_front` declares `mutually_exclusive = { focus = fh_fra_france_insoumise }` and vice versa. GOOD.
- `fh_fra_france_souveraine` declares `mutually_exclusive = { focus = fh_fra_rn_path }`, but `fh_fra_rn_path` does NOT declare exclusivity with `fh_fra_france_souveraine`. **ASYMMETRY BUG.** A player could potentially complete `fh_fra_rn_path` and then `fh_fra_france_souveraine` depending on load order.

**Prevention:**
- Every pair of mutually exclusive focuses must declare the exclusion in both directions.
- Create a symmetry audit: for each focus with `mutually_exclusive`, verify every listed focus reciprocates.
- Write a grep/search: find all `mutually_exclusive` blocks, extract the pairs, verify each pair appears in both focuses.

**Warning signs:** A player reports being able to complete two focuses that were supposed to be exclusive.

**Phase to address:** Phase 2 (focus tree implementation). Audit after writing all focuses, before first full playtest.

---

### Pitfall 6: Focus Tree Not Loaded — Wrong `country` Filter

**What goes wrong:** The `country = { factor = 0; modifier = { add = 10; tag = FRA } }` block controls which country gets this focus tree. If `factor = 0` and no modifier raises it above 0 for France, the game treats all countries equally and may assign the tree to unexpected nations. The pattern used (factor = 0, modifier adds 10 for FRA) is correct — it means only FRA gets this tree.

**Silent failure mode:** If `default = no` and the `country` block has a bug, HoI4 simply shows no focus tree for France. The game doesn't crash. The player sees an empty focus tree panel.

**Current project status:** GOOD. The current implementation is the canonical pattern.

**Prevention:** Always use `default = no` for nation-specific trees. Always pair it with a `country` filter that has `factor = 0` and a positive `modifier` for the target tag.

**Warning signs:** The focus tree panel is empty for France, or another country shows the French tree.

**Phase to address:** Phase 1 (focus tree file creation).

---

### Pitfall 7: `on_startup` Scope Errors Silently Fail

**What goes wrong:** The `on_actions` block's `on_startup` fires once at game start. Scripting inside it runs in a global scope by default. Any effect that requires country scope (like `add_ideas`) must be explicitly scoped to a country tag. Using `add_ideas` directly in `on_startup` without scoping will silently do nothing.

**Current project status:** GOOD. The `fh_on_actions.txt` correctly scopes all effects inside `FRA = { ... }` and `GER = { ... }` blocks.

**`on_daily` performance risk:** The current `on_daily` block fires every single day for every country. The `FRA = { if = { ... } }` pattern checks a flag and a missing idea every day. This is acceptable for one country with one idea, but if this pattern is copy-pasted for 5+ countries or 10+ ideas, it creates noticeable performance degradation by late game.

**Prevention:**
- Always scope country-level effects inside named country blocks or `every_country` with a limit.
- Use `on_startup` for initialization; avoid `on_daily` for anything that can be handled via `on_startup` + event triggers.
- If `on_daily` must be used, add aggressive `limit` conditions to minimize how often the inner effect runs.

**Warning signs:** Ideas set in `on_startup` don't appear on game start; `on_daily` effects applying to wrong countries.

**Phase to address:** Phase 1 (on_actions setup) and ongoing during all phases.

---

### Pitfall 8: Event Scoping — `country_event` vs `news_event`

**What goes wrong:** `country_event` fires an event in the scope of the country that triggered it — the event's effects run as that country. `news_event` fires an event visible to all countries (with a world-news style popup). Using `country_event` when you need a news popup will show a private event instead. Using `news_event` when effects should only apply to France means the effect block runs in a shared/global context and may fail silently.

**Current project status:** All events in `fh_france_events.txt` use `country_event`. This is correct — these are France-only political events. No `news_event` is used. The `is_triggered_only = yes` flag on every event is also correct — these events will not fire randomly, only when explicitly triggered by focus completion or decisions.

**The scoping depth trap:** Events triggered by `country_event` from a focus `completion_reward` run in France's scope. But if that event's options use `random_owned_controlled_state`, the state scope is France-owned states. This is correct. The trap occurs when modders use `every_country` or global scopes inside an event that was triggered as a country event, then wonder why effects apply to unexpected countries.

**Prevention:**
- Always mark political events `is_triggered_only = yes` unless you explicitly want random triggering.
- Use `country_event` for nation-specific political events; use `news_event` only for world-visible announcements.
- When effects inside an event option need to target another country, explicitly scope: `GER = { add_opinion_modifier = { ... } }`.

**Warning signs:** An event fires for all countries when it should only fire for France; or event effects apply to the wrong country.

**Phase to address:** Phase 2 (event writing). Establish the pattern at event #1 and never deviate.

---

### Pitfall 9: Overriding Vanilla France Focus Tree vs Creating a New One

**What goes wrong:** There are two ways to replace vanilla's France focus tree:
1. Use `replace_path = "common/national_focus"` in `descriptor.mod` — this suppresses ALL vanilla focus tree files and requires your mod to provide replacements for every nation, not just France.
2. Give your focus tree `default = no` and a `country` filter that only matches France — vanilla's `00_france.txt` tree also targets France, so both load, and the game uses the one with the highest weight for France (usually the vanilla tree wins or conflicts arise).

The correct approach for a single-nation mod is a third option: use `replace_path` pointed at a specific file, or — the cleanest approach — use the focus tree's own `country` weighting system combined with `default = no` on your tree and relying on vanilla's tree also having `default = no`.

**Current project status:** The `descriptor.mod` uses `replace_path = "common/bookmarks"`. This correctly replaces vanilla bookmarks with the 2026 scenario. However, there is NO `replace_path` for `common/national_focus`, which means vanilla's `00_france.txt` will also load alongside `fh_france_focus.txt`. Both trees target FRA. Whether the game shows one or both depends on their country weights.

**The actual behavior:** HoI4 can display multiple focus trees for the same country — it picks the one with the highest score from the `country` block. Vanilla `00_france.txt` has `default = yes` historically (varies by patch). Your mod's tree has `default = no` with `factor = 0, modifier = add 10 for FRA`. This should win. But vanilla has its own modifiers. Test carefully.

**The clean solution:** Add a file to your mod at `common/national_focus/00_france.txt` that is empty (or contains just `focus_tree = {}`). This shadows the vanilla file and removes the vanilla French tree entirely. Your mod's tree then has no competition.

**Prevention:**
- Add an empty shadow file at `common/national_focus/00_france.txt` in your mod folder.
- Do NOT use `replace_path = "common/national_focus"` unless you intend to replace all nation trees globally.
- Test by starting France at 2026 and verifying only your focus tree appears.

**Warning signs:** Vanilla French focuses (like "The Maginot Line") appear alongside your mod's focuses; the French focus panel shows two trees or the wrong tree.

**Phase to address:** Phase 1 (file structure setup).

---

### Pitfall 10: GFX Declaration Mistakes

**What goes wrong:** Every custom icon referenced in focus files (e.g. `icon = GFX_goal_fh_political_crisis`) must have a corresponding `.gfx` sprite declaration in an `interface/*.gfx` file. The `.dds` file must exist at the declared path. If either is missing, HoI4 substitutes a generic "missing texture" pink/grey icon — no crash, no log entry.

**DDS format requirements:**
- Format: DXT1 (no alpha) or DXT5 (with alpha). HoI4 does not accept standard PNG or uncompressed BMP.
- Size for focus icons: 94x82 pixels (the standard goal icon size). Using the wrong size causes stretching.
- Mipmaps: should be included. A DDS without mipmaps may display correctly but can cause rendering artifacts at distance.

**GFX declaration syntax:**
```
spriteTypes = {
  spriteType = {
    name = "GFX_goal_fh_your_focus"
    texturefile = "gfx/interface/goals/fh_your_focus.dds"
  }
}
```
The `name` must match exactly what is referenced in the focus `icon =` field. Path is relative to the mod root.

**Current project status:** All current focus icons reference vanilla GFX names (e.g. `GFX_goal_generic_dangerous_deal`). No custom GFX is declared yet. The project states `.dds` assets are ready but not yet integrated. This means Phase 3 will need to create the full `interface/*.gfx` infrastructure from scratch.

**Prevention:**
- Create `interface/fh_goals.gfx` before adding any custom icon references.
- Validate DDS files with a tool like GIMP (with DDS plugin) or Paint.NET before integrating — check format is DXT1/DXT5 and dimensions are 94x82.
- Use vanilla icon names (current approach) as placeholders until custom icons are ready — this avoids missing texture errors during development.

**Warning signs:** Focus icons appear as pink/grey squares; game log (if verbose logging is on) may mention missing sprites.

**Phase to address:** Phase 3 (GFX integration). Do not mix custom GFX names in focus files until the interface file is ready.

---

### Pitfall 11: `descriptor.mod` Requirements for Steam Workshop

**What goes wrong:** The `descriptor.mod` file has specific required fields for Steam Workshop upload. Missing or malformed fields prevent the mod from appearing in the Workshop correctly or cause upload rejection.

**Required fields:**
- `name` — mod display name (string)
- `version` — mod version (string)
- `supported_version` — HoI4 version pattern (e.g. `"1.14.*"`)
- `tags` — array of Workshop category tags
- `path` — absolute path to the mod folder on disk (added automatically by the launcher, but must exist for local testing)
- `picture` — path to the mod thumbnail (`.jpg` or `.png`, max 1MB, recommended 900x504px)

**What is missing from the current `descriptor.mod`:**
- No `picture` field — Workshop upload will use a default grey image.
- No `path` field — this is added by the HoI4 launcher when the mod is loaded locally, but must exist in the copy submitted to Workshop.

**`replace_path` behavior:** The current `replace_path = "common/bookmarks"` is correct for suppressing vanilla bookmarks. Any path listed under `replace_path` completely removes vanilla files in that directory from the load order — only your mod's files in that directory are used. This is a global, nuclear option per-directory.

**Prevention:**
- Add a `thumbnail.jpg` (900x504, under 1MB) and reference it in `descriptor.mod` as `picture = "thumbnail.jpg"`.
- When uploading to Workshop, let the HoI4 launcher handle the `path` field — do not manually edit it.
- Keep `supported_version` updated every time a major HoI4 patch releases; an outdated version string triggers an "outdated mod" warning that suppresses Workshop visibility.

**Warning signs:** Mod appears without a thumbnail in the Workshop; "outdated mod" banner appears in the launcher after a HoI4 update.

**Phase to address:** Phase 4 (Steam Workshop release preparation). Set up `picture` field during Phase 1 to avoid forgetting it.

---

## MODERATE PITFALLS

---

### Pitfall 12: Silent Syntax Errors in Focus Tree `.txt` Files

**What goes wrong:** HoI4's Clausewitz parser is extremely permissive — most syntax errors do not crash the game or print readable errors. Common silent failures:

- **Missing closing brace `}`**: The parser may ignore everything after the unclosed block, silently dropping all focuses that follow.
- **Wrong keyword spelling**: `prerequisite` misspelled as `prerequesite` — the block is silently ignored, the focus becomes available from the start with no prerequisites.
- **`available` vs `allowed`**: Confusing these two blocks. `allowed` is checked once at tree load (static). `available` is checked every tick (dynamic). Using `allowed` for a condition that depends on game state (like `has_country_flag`) means it only evaluates at load time — the focus appears available or unavailable permanently regardless of the flag changing.
- **`completion_reward` vs `select_effect` vs `cancel_effect`**: Using `completion_reward` for effects that should only fire on selection (not completion) means the player must finish the focus before effects trigger.

**Current project status:** The focus tree syntax looks structurally sound. The `available = { has_country_flag = fh_volt_europa_unlocked }` usage on `fh_fra_volt_europa` is correct — `available` is the right block for a runtime flag check.

**Prevention:**
- Use an editor with HoI4 syntax highlighting (VS Code with "Paradox Language Support" extension).
- After writing a batch of focuses, run HoI4 with the mod enabled and check the error log at `Documents/Paradox Interactive/Hearts of Iron IV/logs/error.log`.
- Close all blocks at write time — never write an opening `{` without immediately writing the closing `}` and filling in between.

**Warning signs:** A focus has no prerequisites in-game when it should; a focus never becomes available even when conditions are met; focuses disappear from the tree partway down.

**Phase to address:** Every phase. Establish error.log checking as part of every development session.

---

### Pitfall 13: Ideas Not Showing Localisation

**What goes wrong:** An idea defined in `common/ideas/` requires localisation entries for its name and description. The key format is `idea_id` for the name and `idea_id_desc` for the tooltip. If the localisation file uses a different suffix convention or the idea ID doesn't match exactly (case-sensitive), the name shows as the raw key.

**Current project status:** RISK PRESENT. The French localisation file uses keys like `fh_fra_industrial_recovery_spirit` (name) and `fh_fra_industrial_recovery_spirit_desc` (description). The idea in `fh_france_ideas.txt` is named `fh_fra_industrial_recovery_spirit`. This matches. However, the english file is a direct copy of the French text — English players will see French text for all ideas and focuses. This is a v1 known limitation noted in PROJECT.md.

**Note on idea GFX:** All ideas currently reference vanilla `GFX_idea_*` pictures. When custom icons are integrated, the `picture = GFX_idea_fh_...` references must have corresponding declarations in `interface/*.gfx` files with the correct `spriteType` format for ideas (which differs slightly from focus goal sprites).

**Prevention:**
- Keep localisation key naming consistent: `idea_id` + `_desc` suffix, no exceptions.
- When adding a new idea, add both the `name` and `_desc` entries to both localisation files simultaneously.

**Phase to address:** Phase 2 (ideas integration) and Phase 3 (localisation completion).

---

### Pitfall 14: `has_completed_focus` in `visible` Conditions

**What goes wrong:** Using `has_completed_focus = fh_fra_industrial_recovery` in a decision's `visible` block means the decision only appears after that focus is completed. This is correct and intentional in the current code. The trap is using `has_completed_focus` in `available` when you actually mean `visible` — then the decision appears but cannot be taken until the focus is done, which is confusing UX.

**Current project status:** GOOD. All decisions use `visible` for focus-gate and `available` for PP-gate. This is the correct pattern.

**The `days_re_enable` trap:** All decisions use `days_re_enable = 180`. This means after taking a decision, it becomes available again after 180 days. If a decision's `complete_effect` triggers an event that gives a national spirit, and the spirit is already present, the fallback `if/limit` in the event handles it correctly (gives PP instead). This pattern works. The trap would be if `days_re_enable` is too short and the player spams the decision to stack effects — verify the `limit = { NOT = { has_idea = ... } }` guard is robust.

**Phase to address:** Phase 2 (decisions). Audit when all decisions are written.

---

### Pitfall 15: Console Commands for Testing (Avoiding Full Restarts)

**What the workflow should be:** Restarting HoI4 every time you change a focus tree file wastes 2-3 minutes per iteration. The correct workflow is:

1. Change focus tree `.txt` file.
2. In-game, open the console (` ~ ` key).
3. Type `reload focus` — reloads all focus tree files without restarting.
4. Type `reload localisation` — reloads all localisation files.
5. Type `reload ideas` — reloads national ideas.
6. Type `event fh_france.10` — fires a specific event directly for testing.
7. Type `focus.nochecks` — removes all prerequisites and conditions, letting you complete any focus instantly.
8. Type `tag FRA` — switches your controlled country to France.
9. Type `pp 999` — gives 999 political power for testing decisions.

**For GFX testing:** `gfxreload` or `reload gfx` can reload interface files, but DDS changes often require a full restart.

**Caution:** Console commands in ironman mode are disabled. Testing always requires a non-ironman save.

**The `error.log` loop:** After every reload, check `Documents/Paradox Interactive/Hearts of Iron IV/logs/error.log`. This file accumulates errors from this session. Search for your mod's prefix (`fh_`) to filter relevant errors.

**Phase to address:** Phase 1 (establish the development workflow before any content is written). Document these commands in a dev-notes file.

---

### Pitfall 16: `add_resource` State ID Hardcoding

**What goes wrong:** The focus `fh_fra_strategic_reserves` uses `add_resource = { type = steel; amount = 8; state = 16 }`. State `16` is hardcoded as a vanilla state ID (Nord-Pas-de-Calais in vanilla, a French steel region). This is a known HoI4 convention — state IDs are stable and do not change between vanilla patches. But if a future HoI4 update renumbers states (rare but has happened with DLC), this hardcoded ID breaks silently by adding resources to the wrong state.

**Prevention:** Document every hardcoded state ID with a comment: `state = 16 # Nord-Pas-de-Calais (vanilla ID, stable)`. Prefer `random_owned_controlled_state` when the specific state doesn't matter for gameplay.

**Phase to address:** Phase 2. When hardcoding state IDs, add comments immediately.

---

## MINOR PITFALLS

---

### Pitfall 17: `guarantee_cost = -1.0` Modifier in Ideas

**What goes wrong:** The `fh_union_europeenne` idea uses `guarantee_cost = -1.0` as a modifier. This is not a standard HoI4 modifier key — the actual key is `guarantee_cost` and it modifies the political power cost of guaranteeing another country's independence. A value of `-1.0` would reduce the cost by 100%, making guarantees free. This may be intentional, but verify the modifier key exists in the game's modifier documentation. Using a non-existent modifier key is silently ignored.

**Phase to address:** Phase 2 (ideas audit). Verify all modifier keys against vanilla idea definitions.

---

### Pitfall 18: Bookmark `picture` Using Vanilla GFX

**What goes wrong:** The bookmark in `fh_2026.txt` references `picture = "GFX_select_date_1936"` — this is the vanilla 1936 scenario thumbnail. This means the 2026 scenario uses the 1936 artwork in the scenario selection screen. While it won't crash the game, it is visually incorrect for a 2026-set mod.

**Prevention:** Create a custom `GFX_select_date_fh_2026` sprite in `interface/fh_bookmarks.gfx` pointing to a custom background image. This is part of Phase 3 GFX integration.

**Phase to address:** Phase 3 (GFX).

---

### Pitfall 19: Focus Tree Not Resetting on Civil War

**What goes wrong:** The focus tree has `reset_on_civilwar = yes`. This means if France enters a civil war, the focus tree resets. Given the mod's political branches include paths that increase fascism/communism popularity significantly, a civil war mid-playthrough is plausible. The reset wipes all completed focuses.

Whether `reset_on_civilwar = yes` is intentional design (yes, it is in Road to 56 style) or a bug depends on intent. If the player has invested 20 focuses into the Macron path and a civil war resets everything, this could feel punishing.

**Prevention:** Decide deliberately whether civil war should reset the tree. If yes, keep it. If no (the player's completed focuses represent permanent narrative decisions), change to `reset_on_civilwar = no`.

**Phase to address:** Phase 1 (design decision, not a bug).

---

### Pitfall 20: English Localisation is Identical to French

**Current project status:** Both `fh_l_french.yml` and `fh_l_english.yml` contain identical French text. English-speaking players will see French in all focus names, event titles, and tooltips. This is acknowledged as a v1 limitation in PROJECT.md (both languages are listed as required). However, this means the English file must be fully written before the Steam Workshop release.

**Risk:** If the English file is left as a copy of French until the last moment, the translation workload becomes a blocker. Better to write English text alongside French as content is created, not at the end.

**Prevention:** Write English and French localisation entries simultaneously for each new key. Never ship to Workshop with English keys containing French text.

**Phase to address:** Every phase of content writing. Not deferred to a "translation phase."

---

## PHASE-SPECIFIC WARNINGS

| Phase Topic | Likely Pitfall | Mitigation |
|-------------|---------------|------------|
| Infrastructure setup | BOM encoding lost on new files | Set VS Code workspace encoding for `*.yml` to `utf8bom` |
| Focus tree IDs | Collision with vanilla `FRA_` namespace | Use `fh_fra_` prefix; shadow `00_france.txt` |
| Focus tree layout | `y = -1` trunk invisible; width overflow | Audit at 1080p before declaring tree done |
| Mutual exclusion | Asymmetric pairs (rn_path / france_souveraine) | Run symmetry audit script after all focuses are written |
| Ideas writing | Non-existent modifier keys silently ignored | Cross-check every modifier key against vanilla idea files |
| Events writing | Scope confusion, missing `is_triggered_only` | Establish pattern at event #1; never deviate |
| GFX integration | Wrong DDS format or size; missing `.gfx` declaration | Validate DDS before referencing; create `.gfx` file first |
| Localisation (EN) | French text in English file at ship time | Write both languages simultaneously per key |
| Steam Workshop | Missing thumbnail, outdated `supported_version` | Add `picture` field in Phase 1; update version string before release |
| Testing workflow | Full restart every change | Use `reload focus`, `reload localisation` console commands |
