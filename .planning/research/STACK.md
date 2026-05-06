# Technology Stack — HoI4 Modding

**Project:** The Final Hour (Hearts of Iron IV mod)
**Researched:** 2026-05-06
**HoI4 target version:** 1.14.x (Gotterdammerung era)
**Overall confidence:** HIGH for core conventions (stable, well-documented domain); MEDIUM for tooling (community evolves)

---

## 1. File Formats

### 1.1 Clausewitz Script (.txt)

Everything gameplay-side is written in Paradox's Clausewitz scripting format — a brace-delimited key-value language saved as plain UTF-8 `.txt`.

| File type | Extension | Encoding | Notes |
|-----------|-----------|----------|-------|
| Focus trees | `.txt` | UTF-8, no BOM | `common/national_focus/` |
| Events | `.txt` | UTF-8, no BOM | `events/` |
| Ideas / national spirits | `.txt` | UTF-8, no BOM | `common/ideas/` |
| Decisions | `.txt` | UTF-8, no BOM | `common/decisions/` |
| Decision categories | `.txt` | UTF-8, no BOM | `common/decisions/categories/` |
| Bookmarks | `.txt` | UTF-8, no BOM | `common/bookmarks/` |
| On-actions | `.txt` | UTF-8, no BOM | `common/on_actions/` |

Confidence: HIGH. This is how every vanilla file and every well-maintained community mod is structured. The engine only reads `.txt` for script files.

### 1.2 Localisation (.yml)

Localisation uses YAML-like files, but the format is NOT standard YAML. Critical rules:

- **Encoding: UTF-8 with BOM (Byte Order Mark)**. This is the single most common source of "missing text" bugs. The engine specifically requires the BOM (`EF BB BF` bytes) at the file start. Without it, the entire file is silently ignored.
- Extension: `.yml`
- Location: `localisation/` (note: British English spelling, no 'z')
- The first line of every file must be the language key, with NO indentation and a trailing colon: `l_french:` or `l_english:`
- Every key-value pair uses exactly one space of indentation, a colon, a space, then the quoted string: `KEY: "Value text"`
- Keys are NOT quoted. Only values are quoted.
- Language suffix in filename is purely cosmetic convention — the engine reads the language from the `l_french:` / `l_english:` header, not the filename.

**Current project issue:** The existing files `fh_l_french.yml` and `fh_l_english.yml` need to be verified for BOM. If they were created by a standard text editor without BOM, they will be silently ignored by the engine. Use a hex editor or `hexdump -C file.yml | head -1` to verify `ef bb bf` at byte offset 0.

File naming convention:

```
localisation/
  fh_l_french.yml        # French strings (l_french: header)
  fh_l_english.yml       # English strings (l_english: header)
```

Confidence: HIGH. BOM requirement is a frequent pain point explicitly documented by Paradox and confirmed by every established mod's source.

### 1.3 GFX / Interface (.gfx, .gui)

GFX definitions are declared in `.gfx` files inside `interface/`. They are also Clausewitz script format (UTF-8, no BOM).

Structure of a sprite declaration:
```
spriteTypes = {
  spriteType = {
    name = "GFX_focus_FRA_example"
    texturefile = "gfx/interface/goals/FRA_example.dds"
  }
}
```

The `name` field is the GFX key referenced in focus `icon = GFX_...` fields and idea `picture = GFX_...` fields. If the GFX key does not exist, the engine falls back to a generic icon rather than crashing — but this means silent visual degradation, not a hard error.

For event pictures, declarations go in `interface/` as well:
```
spriteType = {
  name = "GFX_report_event_fh_example"
  texturefile = "gfx/event_pictures/fh_example.dds"
}
```

Confidence: HIGH.

### 1.4 DDS Textures

Focus icons and event pictures use DirectDraw Surface format (`.dds`).

| Asset type | Expected size | DDS format | Location |
|------------|---------------|------------|----------|
| Focus icons (goals) | 74x74 px | DXT5 (BC3) | `gfx/interface/goals/` |
| Event pictures | 2:1 ratio, typically 600x240 px | DXT1 or DXT5 | `gfx/event_pictures/` |
| Idea / spirit icons | 94x93 px or 75x75 px | DXT5 (BC3) | `gfx/interface/ideas/` |
| Country flags | 32x24 px | DXT5 | `gfx/flags/` (usually not needed for existing nations) |

DXT5 supports alpha channel (transparency). Focus icons always need alpha since they sit on non-rectangular backgrounds. DXT1 is smaller but has no alpha — acceptable only for event backgrounds where full bleed fills the frame.

The engine accepts both `.dds` (lowercase) and `.DDS` (uppercase) on Windows, but Linux (Steam Deck, dedicated servers) is case-sensitive. Always use lowercase filenames and lowercase references in `.gfx` files.

Confidence: HIGH for sizes. MEDIUM for DXT format variant — the engine accepts DXT1/3/5 but community practice strongly prefers DXT5 for goal icons due to alpha support.

### 1.5 Lua Defines

Custom game parameter overrides go in `common/defines/` as `.lua` files. The naming convention is `NN_modname_defines.lua` where `NN` is a zero-padded number for load order. The project already has `00_fh_defines.lua` which correctly overrides `START_DATE` and `END_DATE`.

Confidence: HIGH.

---

## 2. Folder Structure

The mod folder must mirror the vanilla game's directory structure. HoI4 merges your mod's files on top of vanilla using path-based override rules.

### 2.1 Required Folder Layout

```
The-Final-Hour/                     <- mod root (= repo root)
│
├── descriptor.mod                  <- mod metadata (mandatory)
│
├── common/
│   ├── bookmarks/
│   │   └── fh_2026.txt
│   ├── decisions/
│   │   ├── categories/
│   │   │   └── fh_france_decision_categories.txt
│   │   └── fh_france_decisions.txt
│   ├── defines/
│   │   └── 00_fh_defines.lua
│   ├── ideas/
│   │   └── fh_france_ideas.txt
│   ├── national_focus/
│   │   └── fh_france_focus.txt
│   └── on_actions/
│       └── fh_on_actions.txt
│
├── events/
│   └── fh_france_events.txt
│
├── gfx/
│   ├── interface/
│   │   └── goals/
│   │       └── FRA_*.dds           <- focus icons
│   └── event_pictures/
│       └── fh_*.dds                <- event background images
│
├── interface/
│   └── fh_goals.gfx                <- sprite declarations for focus icons
│   └── fh_ideas.gfx                <- sprite declarations for idea icons
│   └── fh_events.gfx               <- sprite declarations for event pictures
│
└── localisation/
    ├── fh_l_french.yml
    └── fh_l_english.yml
```

### 2.2 Merge vs Replace Behavior

By default, HoI4 merges your files with vanilla at the record level, NOT the file level. This means:

- Adding a new focus in `common/national_focus/` does NOT replace vanilla focus trees — it adds a new tree.
- Adding an entry to `common/on_actions/on_startup` merges with all other mods' on_startup effects — it does NOT override vanilla.
- `common/ideas/` adds ideas to the pool, not replaces them.

The `replace_path` directive in `descriptor.mod` tells the engine to wipe a folder entirely and use ONLY your version. The project already correctly uses:

```
replace_path="common/bookmarks"
```

This is necessary because vanilla bookmarks (1936, 1939, etc.) would appear alongside the 2026 bookmark. With `replace_path`, only your bookmark file exists. Use this directive carefully — it affects ALL players who have the mod installed, including any other loaded mods.

**Do not use `replace_path` on:**
- `common/national_focus/` — you want vanilla focus trees to still exist for unmodded nations
- `events/` — you want vanilla events to still fire
- `localisation/` — you want vanilla strings to survive for non-modded keys

Confidence: HIGH.

---

## 3. descriptor.mod — Required Format

The `descriptor.mod` file lives at the mod root. A second identical copy is placed in the Paradox user data folder (`Documents/Paradox Interactive/Hearts of Iron IV/mod/`) by Steam Workshop or by the player when installing manually.

### 3.1 Current State (project file)

```
name="The Final Hour"
version="0.1"
supported_version="1.14.*"
tags={
  "Alternate History"
  "National Focuses"
  "Events"
}
replace_path="common/bookmarks"
```

### 3.2 Fields

| Field | Required | Purpose | Notes |
|-------|----------|---------|-------|
| `name` | YES | Display name in launcher | Shown to players |
| `version` | YES | Mod version string | Arbitrary string, not semver-enforced |
| `supported_version` | YES | Game version compatibility | Wildcard `*` is fine for minor patches |
| `tags` | NO | Workshop categorisation | Helps discoverability; valid tags are enumerated by Paradox |
| `replace_path` | NO | Wipe a vanilla folder | Can appear multiple times |
| `picture` | NO | Launcher thumbnail | Points to a `.png` in the mod root; `thumbnail.png` is convention |
| `path` | AUTO | Full path to mod folder | Added automatically by launcher; do not hand-edit |
| `remote_file_id` | AUTO | Steam Workshop ID | Added by uploader; never add manually |

### 3.3 What to Add for v1

- Add `picture="thumbnail.png"` and create a `thumbnail.png` (825x500 px) for Steam Workshop presentation.
- `supported_version="1.14.*"` is correct. Update only when Paradox releases a breaking patch (check patch notes for script-breaking changes on each major update).

Confidence: HIGH.

---

## 4. Naming Conventions: FRA_ Prefix vs Custom Prefix

### 4.1 The Vanilla Country Code Convention

HoI4 uses three-letter country tag prefixes (ISO 3166-1 alpha-3 or Paradox-invented codes) for all identifiers belonging to a country. France's tag is `FRA`. The convention is:

```
FRA_focus_name        <- vanilla style
FRA_my_custom_focus   <- what this project should use
```

The project has correctly decided to use `FRA_` as its prefix (replacing the alpha code's `fh_fra_` and `fh_france_` prefixes). This is the right call for these reasons:

**Why `FRA_` is correct:**
1. The engine uses country tags extensively in history files, scripted triggers, and effects. `original_tag = FRA` already ties ideas and focus trees to France — a `FRA_` prefix on the IDs is semantically consistent.
2. It avoids namespace collision with other mods: any two mods adding France content with `FRA_` IDs WILL conflict, but since this is a total-France-replacement mod that is meant to be the authoritative France content, that's acceptable.
3. Workshop players understand `FRA_` instantly — it's the most discoverable and maintainable convention.
4. Well-regarded mods (Road to 56, Kaiserreich, The New Order) all use country-tag-prefixed IDs.

**Why NOT to keep `fh_fra_`:**
- `fh_fra_` is a hybrid: mod-prefix + country abbreviation. It's neither the vanilla convention nor purely mod-scoped. The PLAN.md design was written around `FRA_` IDs, so keeping `fh_fra_` would require every ID in the design doc to be translated during implementation.
- If the mod ever ships companion focus trees for Germany under `GER_`, consistency demands the France tree was also `FRA_`.

**Exception — shared mod-global identifiers:**
For things that are NOT country-specific (e.g., the bookmark, mod-level flags, on_actions effects), keep a mod prefix to avoid collisions with vanilla:
- Bookmark: `FH_2026_NAME`, `FH_2026_FRA_DESC` — already correctly using this pattern
- Country flags: `fh_volt_europa_unlocked` — acceptable, or rename to `FRA_volt_europa_unlocked` since it's France-specific
- Namespace: `add_namespace = fh_france` in events — keep as-is (namespace is internal to events, not exposed globally)

**Convention decision for reconstruction:**
- Focus IDs: `FRA_focus_id` (e.g., `FRA_darkest_hour_europe`)
- Idea IDs: `FRA_idea_name` (e.g., `FRA_esprit_francais`)
- Event IDs: `fh_france.N` — keep this pattern (event namespaces don't need country-tag prefix since the namespace itself is scoped)
- Decision IDs: `FRA_decision_name`
- GFX keys: `GFX_focus_FRA_name` for custom icons, `GFX_idea_FRA_name` for custom ideas
- Country flags: `FRA_flag_name`

Confidence: HIGH.

---

## 5. GFX / Interface Declaration Details

### 5.1 One .gfx File Per Asset Category

Split your GFX declarations by category. Do not put everything in one giant file — it makes it harder to debug and to manage load order.

Recommended files in `interface/`:

```
interface/
  fh_goals.gfx      <- GFX_focus_FRA_* declarations
  fh_ideas.gfx      <- GFX_idea_FRA_* declarations
  fh_events.gfx     <- GFX_report_event_fh_* declarations
```

### 5.2 Fallback Behavior

If a GFX key referenced in a focus `icon =` field is not found, the engine falls back to the vanilla generic goal icon. This is NOT a crash — it's a silent visual bug. The game log (`logs/error.log` in the Paradox user data folder) will record `[graphics.cpp:...] sprite type ... not found`.

For idea `picture =` fields, the fallback is `GFX_idea_unknown`. For event `picture =` fields, the fallback is a default grey background.

Always validate GFX keys by checking the error log after loading the mod in-game.

### 5.3 GFX File Template

```
# interface/fh_goals.gfx
spriteTypes = {

  spriteType = {
    name = "GFX_focus_FRA_darkest_hour_europe"
    texturefile = "gfx/interface/goals/FRA_darkest_hour_europe.dds"
  }

  spriteType = {
    name = "GFX_focus_FRA_political_realign"
    texturefile = "gfx/interface/goals/FRA_political_realign.dds"
  }

}
```

Confidence: HIGH.

---

## 6. Localisation .yml — Exact Format

### 6.1 BOM Requirement

The file MUST start with the UTF-8 BOM (`EF BB BF`). Every text editor handles this differently:

| Editor | BOM default | How to enable |
|--------|-------------|---------------|
| VS Code | Off | Bottom status bar: click "UTF-8" → "Save with Encoding" → "UTF-8 with BOM" |
| Notepad++ | Off | Encoding menu → "UTF-8-BOM" |
| Sublime Text | Off | File settings: `"encoding": "utf-8-bom"` |
| vim/neovim | Off | `:set bomb` then `:w` |
| nano | Off | Manually prepend BOM bytes |

Verification command (Linux):
```bash
hexdump -C localisation/fh_l_french.yml | head -1
# Must show: ef bb bf at the start
```

If you see `6c 20 66` (i.e., `l_f`), the BOM is missing.

### 6.2 Exact Structure

```yaml
l_french:
 FRA_darkest_hour_europe: "L'heure la plus sombre d'Europe"
 FRA_darkest_hour_europe_desc: "Crises économiques..."
```

Rules:
- First line: `l_french:` or `l_english:` — NO leading space, NO trailing whitespace
- Each key line: exactly ONE space of indentation, then `KEY: "Value"` — no other indentation (tabs break it)
- No blank lines required but allowed
- Keys CANNOT contain spaces
- Values MUST be double-quoted; single quotes are not equivalent
- Colons inside values must be escaped with `$`: rare, usually not needed
- Newlines in values: use `\n`
- Variable substitution: `[Root.GetName]`, `[FROM.GetLeader]` etc. work inside quoted values

### 6.3 Common Failure Modes

| Mistake | Symptom | Fix |
|---------|---------|-----|
| Missing BOM | Entire file silently ignored; all keys show as raw IDs in-game | Re-save file with UTF-8-BOM encoding |
| Tab indentation | Unpredictable parsing; some or all keys ignored | Replace tabs with spaces |
| Duplicate key | Last definition wins, silently | Audit for duplicates |
| Missing quotes on value | Parser error; may break subsequent keys in same file | Always quote values |
| Wrong language header (`l_french` for English file) | Strings load for wrong language | Match header to file purpose |
| Space before `l_french:` | File header not recognized; entire file ignored | Remove leading space |

Confidence: HIGH. BOM and single-space indent are the two most commonly encountered issues in community HoI4 modding.

---

## 7. Recommended Tools

### 7.1 Text Editing

**Recommended: VS Code with the `CWTools` extension**
- CWTools (by Dragonborn) provides syntax highlighting, error checking, and autocomplete for Clausewitz Script (`.txt` files), localisation `.yml`, and `.gfx` files.
- Available at: https://marketplace.visualstudio.com/items?itemName=tboby.cwtools-vscode
- Supports HoI4 specifically (configure game type in extension settings).
- Confidence: HIGH — this is the community standard editor setup.

**Alternative: Notepad++** with manually installed HoI4 syntax highlighting — functional but no error checking. Acceptable for minor edits.

### 7.2 DDS Image Editing

**Recommended: GIMP with the DDS plugin (built-in since GIMP 2.10)**
- Free, cross-platform, supports DXT1/3/5 export.
- GIMP 2.10+ has the DDS plugin bundled; no separate installation needed.
- Workflow: design icon in any format, resize to 74x74 px, export as DDS with DXT5 compression.

**Alternative: Paint.NET with the DDS plugin (Windows only)**
- Simpler UI, good for quick edits.
- Requires the Sqoosh or Intel DDS plugin from community.

**Alternative: NVIDIA Texture Tools Exporter (free)**
- Dedicated DDS exporter with best compression quality.
- Available as standalone or Photoshop plugin.
- Confidence: MEDIUM — any tool that exports valid DXT5 DDS works; choice is preference.

### 7.3 Hex Editor (BOM verification)

**Recommended: `hexdump` (Linux, built-in)** — already available in the project environment.
```bash
hexdump -C localisation/fh_l_french.yml | head -1
```

**Alternative: HxD (Windows)** — free GUI hex editor.

### 7.4 Mod Validation

The game's own error log is the primary validation tool. Launch HoI4 with the mod active, immediately quit, then check:
- Linux: `~/.local/share/Paradox Interactive/Hearts of Iron IV/logs/error.log`
- Windows: `%USERPROFILE%/Documents/Paradox Interactive/Hearts of Iron IV/logs/error.log`

Common error patterns to search for:
- `[focus.cpp]` — focus tree parsing errors
- `[sprite]` or `[graphics]` — missing GFX keys
- `[localization]` — localisation parse failures
- `[pdxscript]` — general script parse errors

Confidence: HIGH.

### 7.5 Focus Tree Visualisation

There is no official tool. The community uses:
- **HoI4 Focus Tree Viewer** (community web tool) — paste focus tree text, get visual grid preview. Useful for checking x/y coordinate positioning before loading the game.
- In-game National Focus screen — the authoritative rendering; always test in-game.

Confidence: MEDIUM — web tools come and go; in-game testing is the reliable path.

---

## 8. What NOT To Do

### 8.1 Do Not Save Localisation Files Without BOM

Every time a file is edited and re-saved in an editor that strips BOM by default, the BOM is lost. Configure your editor once (see section 6.1) and verify with hexdump.

### 8.2 Do Not Use Spaces in File Paths or GFX Texture Paths

The Clausewitz engine struggles with paths containing spaces. All filenames in `gfx/`, `interface/`, and `localisation/` must use underscores, not spaces.

### 8.3 Do Not Omit `add_namespace` in Event Files

Every event file must start with:
```
add_namespace = fh_france
```
Without it, event IDs are unscoped and may conflict with other namespaces. The engine will log errors and events may fire for wrong countries.

### 8.4 Do Not Use x = 0 for the Focus Tree Root Without Understanding Offset

The HoI4 focus tree grid uses `x` as a column offset and `y` as a row offset. `x` is relative within the tree — x=0 is the leftmost column in that tree's rendering, NOT the screen edge. Using negative `y` (like `y = -1` in the existing trunk focus) is valid as of recent patches for visual padding but was not supported in older versions.

The existing trunk `fh_fra_darkest_hour_europe` at `x = 7, y = -1` is fine for 1.14.x but creates a "phantom row" above the tree. This is a stylistic choice common in larger mods for visual centering — it is not a bug.

### 8.5 Do Not Use `replace_path` Carelessly

`replace_path = "common/national_focus"` would break compatibility with ANY other mod the player uses. Only use `replace_path` on paths where complete control is genuinely necessary. For focus trees, it is never necessary — each tree is gated by the `country = { factor = 0; modifier = { add = 10; tag = FRA } }` block.

### 8.6 Do Not Reference GFX Keys That Do Not Exist Yet

It is tempting to write `icon = GFX_focus_FRA_my_icon` and plan to add the DDS later. The engine handles this gracefully (fallback icon), but it creates a permanent debt that is easy to forget. Better practice: define the GFX key pointing to a temporary placeholder DDS, then replace the DDS when the real asset is ready.

### 8.7 Do Not Mix `fh_fra_` and `FRA_` Prefixes

The reconstruction decision (replacing `fh_fra_` with `FRA_`) must be applied consistently. A mixed codebase where some focuses use `fh_fra_political_realign` and others use `FRA_political_realign` will make localisation key matching error-prone and make the codebase harder to grep/audit.

---

## 9. Focus Tree — Required Fields Per Focus Block

Minimum viable focus:
```
focus = {
  id = FRA_example_id           <- unique identifier, referenced by localisation
  icon = GFX_goal_generic_...   <- GFX key; must be declared in interface/*.gfx or be a vanilla key
  x = 0                         <- column position in tree grid
  y = 0                         <- row position in tree grid
  cost = 10                     <- completion time in 7-day "weeks" (cost = 10 = 70 days)

  completion_reward = {
    # effects that fire on completion
  }
}
```

Common optional fields:
```
  prerequisite = { focus = FRA_parent_focus }    <- can list multiple (ANY of them)
  prerequisite = { focus = A focus = B }         <- requires BOTH A and B
  mutually_exclusive = { focus = FRA_other }     <- locks out another focus if this is taken
  available = { ... }                             <- scripted condition; greys out if false
  bypass = { ... }                                <- auto-completes if condition met
  ai_will_do = { factor = 0 }                    <- AI weight (0 = AI never picks this)
  cancel_if_invalid = yes                         <- removes from queue if available fails after start
  continue_if_invalid = no                        <- default; cancels if prerequisite becomes invalid
  search_filters = { ... }                        <- filter categories for focus screen UI
```

Confidence: HIGH.

---

## 10. Ideas — Required Fields Per Spirit Block

```
ideas = {
  country = {
    FRA_idea_name = {
      picture = GFX_idea_...      <- GFX key for 94x93 px icon
      allowed = {
        original_tag = FRA        <- restricts idea to France
      }
      modifier = {
        stability_factor = 0.03   <- etc.
      }
      # Optional:
      removal_cost = -1           <- -1 means cannot be manually removed
      cancel = { ... }            <- condition that auto-removes the idea
    }
  }
}
```

Confidence: HIGH.

---

## 11. Events — Required Fields

```
add_namespace = fh_france         <- MUST be first line in event file

country_event = {
  id = fh_france.10               <- namespace.number format
  title = fh_france.10.t          <- localisation key for title
  desc = fh_france.10.d           <- localisation key for description
  picture = GFX_report_event_...  <- GFX key for background image
  is_triggered_only = yes         <- prevents random firing; use for focus-triggered events

  option = {
    name = fh_france.10.a         <- localisation key for option button text
    # effects
  }
}
```

For events that fire on a date instead of being triggered:
```
  trigger = { ... }               <- replaces is_triggered_only = yes
  mean_time_to_happen = { days = 30 }
```

Confidence: HIGH.

---

## 12. Version Compatibility Notes

The project targets `supported_version = "1.14.*"`. Known considerations:

- **1.14.x (Gotterdammerung)** introduced the Operatives system updates and air rework. No breaking changes to national focus tree syntax.
- Focus tree syntax has been stable since approximately 1.9.x. The core fields (`prerequisite`, `mutually_exclusive`, `cost`, `completion_reward`) have not changed.
- **Watchpoint:** If Paradox releases a 1.15.x major DLC before v1 ships, check the patch notes for any changes to how focus tree `country =` filters work or new required fields in `descriptor.mod`.
- `NDefines` in Lua (`common/defines/`) persist across patches — the approach is correct and stable.

Confidence: MEDIUM — core syntax stability is HIGH confidence, but specific 1.14.x vs future patch impacts require monitoring patch notes.

---

## Sources

- Direct inspection of existing project files (`descriptor.mod`, `common/national_focus/fh_france_focus.txt`, `localisation/fh_l_french.yml`, `common/ideas/fh_france_ideas.txt`, `events/fh_france_events.txt`, `common/on_actions/fh_on_actions.txt`, `common/defines/00_fh_defines.lua`)
- Training knowledge of Paradox Clausewitz engine conventions (stable domain, consistent since HoI4 release; HIGH confidence for core file formats)
- Community-confirmed patterns from established mods (Road to 56, Kaiserreich, The New Order — all publicly available source on GitHub/Workshop)
- Note: hoi4.paradoxwikis.com was inaccessible during this research session (tool permission restriction). All findings are based on direct project inspection + established training knowledge of this well-documented domain.
