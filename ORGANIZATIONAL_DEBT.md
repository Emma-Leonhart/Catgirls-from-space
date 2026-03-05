# Organizational Debt Analysis

Specific issues found and concrete recommendations for cleanup.

---

## Critical Issues

### 1. Duplicate Endings Files

The endings system is split across 4+ files with overlap:

| File | Contents |
|---|---|
| `content/endings.json` (root) | 28 endings — subset of what's in the folder |
| `content/endings/endings.json` | 453 endings — the main file |
| `content/endings/DLC_EXILE_endings.json` | Exile-specific endings |
| `content/exile/endings.json` | More exile endings (overlaps with above?) |

**Action:** Verify `content/endings.json` is fully redundant with `content/endings/endings.json`, then delete the root copy. Clarify relationship between the two exile ending files.

### 2. The Misc Folder

`content/misc probably needs better categorization or merging/` contains only 2 files:

- `begging.json` — work recipes for begging (exile/poverty context) → move to `content/exile/`
- `murder hunter.json` — talk recipe for killing hunters → move to `content/overwriting/`

**Action:** Move files and delete the directory.

---

## Cross-Directory Sprawl

### Exile Content (worst offender)

Exile files found **outside** `content/exile/`:

| File | Current Location | Suggested |
|---|---|---|
| `exile_vaults.json` | `capers and expeditions/` | `exile/` |
| `exile initial dreams.json` | `catgirl ascension/` | `exile/` |
| `DLC_EXILE_endings.json` | `endings/` | `exile/` or keep but clarify |

### Cryonics Split Across 3 Locations

| Location | Files |
|---|---|
| `content/cryonics legacy/` | `cryonics legacy elements.json`, `cryonics legacy recipes.json` |
| `content/exile/` | `cryonics_elements.json`, `cryonics_recipes.json` |
| `content/people/` | `followers_cryonics.json` |

**Question:** Is the cryonics legacy system separate from exile cryonics, or should they be merged?

### Detective Split Across 2 Locations

| Location | Files |
|---|---|
| `content/detective/` | `detective_stuff.json`, `legacy_detective_elements_overwrite.json`, `legacy_detective_recipes_overwrite.json` |
| `content/exile/` | `exile_detective_elements.json`, `exile_detective_recipes.json` |

**Question:** Is detective a standalone legacy or an exile subsystem? Clarify and consolidate.

---

## Loose Root Files

10 JSON files sit directly in `content/` with no folder:

| File | Suggested Destination |
|---|---|
| `byt legacy overwriting.json` | `overwriting/` or `capitalist ascension/` |
| `cultures.json` | `core/` |
| `deaths overwriting.json` | `overwriting/` |
| `endings.json` | DELETE (duplicate) |
| `entrapment.json` | `core/` |
| `misc auctionables.json` | `items/` |
| `painting.json` | its own folder or `items/` |
| `poppy.json` | `catgirl ascension/` or `people/` |
| `unimplemented jobs.json` | `investing/` or DELETE if truly unimplemented |
| `uniqueness groups.json` | `core/` |

---

## Naming Inconsistencies

The project mixes spaces and underscores in both directory and file names:

**Directories with spaces (should use underscores):**
- `basic ability overwriting/`
- `capers and expeditions/`
- `capitalist ascension/`
- `catgirl ascension/`
- `cryonics legacy/`
- `dancer legacy/`
- `long foe combat/` (already `long_foe_combat/` — inconsistent with others)

**Files with spaces (sampling):**
- `disabling recipes.json`
- `new books.json`
- `new study system.json`
- `evil endgame elements.json`
- `dream mansus new recipes.json`
- `entering mansus with lore types.json`
- `stag door answers.json`
- etc.

**Recommendation:** Standardize everything to `snake_case`. This would require a single bulk rename pass. Since the engine loads by directory, not by filename, renaming files won't break anything as long as no file references others by name (they don't — content references use IDs, not filenames).

---

## Translation / Localization Coverage

`content/cultures.json` currently has **only English** with ~160 UI label strings. The structure supports additional languages by adding entries to the `cultures` array.

### What Would Need Translating

**Tier 1 — UI Labels (~160 strings in `cultures.json`):**
Button text, menu labels, status messages. These are the minimum for a language to be playable.

**Tier 2 — Element Labels and Descriptions (thousands of strings):**
Every element's `"label"` and `"description"` across all ~254 JSON files. This is the bulk of translatable content and would be a massive undertaking.

**Tier 3 — Recipe Text (~hundreds):**
Every recipe's `"label"`, `"startdescription"`, and `"description"`.

**Feasibility Assessment:**
A full translation audit would require extracting every `label`, `description`, and `startdescription` field from every JSON file and cataloging them. This is doable with a script but the volume is enormous — likely 5,000+ translatable strings across the mod. For now, the mod is English-only and that's reasonable for a conversion mod of this scope.

---

## Priority Order

1. **Delete misc folder** (move 2 files, delete directory)
2. **Delete duplicate `content/endings.json`** (after verification)
3. **Move loose root files** into appropriate subdirectories
4. **Consolidate exile sprawl** (3 files to move)
5. **Decide on cryonics/detective architecture** (needs design decision)
6. **Bulk rename to snake_case** (optional but recommended)
7. **Translation tooling** (future — extract all translatable strings to a manifest)
