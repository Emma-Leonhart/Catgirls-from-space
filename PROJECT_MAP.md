# Catgirls from Space - Project Map

Where everything is, what it does, and how the JSON works.

This is a **Cultist Simulator mod** — there is zero C# code. Everything is data-driven JSON that the Cultist Simulator engine reads. The engine defines what fields like `id`, `aspects`, `xtriggers`, `slots`, `requirements`, `effects`, `linked`, `alt`, etc. mean. The mod just provides content files that the engine loads.

---

## How Cultist Simulator JSON Works

### Entity Types (Top-Level Keys)

Each JSON file contains one or more of these top-level arrays. The engine loads them by key name:

| Top-Level Key | What It Defines |
|---|---|
| `"elements"` | Cards/tokens the player sees and interacts with (items, followers, lore, books, locations, etc.) |
| `"recipes"` | Rules for what happens when cards are combined in verb slots (the core game logic) |
| `"verbs"` | Action slots on the table (Work, Study, Dream, Talk, etc.) |
| `"decks"` | Pools of cards to draw from randomly |
| `"legacies"` | Starting character options |
| `"endings"` | Win/lose conditions |
| `"cultures"` | Localization / UI text |

### Key Fields on Elements

| Field | Purpose |
|---|---|
| `id` | Unique identifier, referenced by recipes and other elements |
| `label` | Display name |
| `description` | Flavor text shown to player |
| `aspects` | Tags with numeric values (e.g. `"edge": 2, "heart": 3`) — used by recipes to match requirements |
| `isAspect` | If `true`, this element IS an aspect (like a tag definition), not a card |
| `xtriggers` | Transformations triggered externally (e.g. `"book_to_lore": "fragmentheart"` means studying this book yields a heart fragment) |
| `unique` | Only one copy can exist at a time |
| `uniquenessgroup` | Only one element in this group can exist |
| `inherits` | Inherit fields from a parent element template (e.g. `"inherits": "_vault.exile"`) |
| `icon` | Image filename (without extension) from `images/elements/` |
| `induces` | Probabilistic side effects (`"chance": 10` = 10% to trigger) |
| `slots` | Sub-slots where the player can place additional cards |

### Key Fields on Recipes

| Field | Purpose |
|---|---|
| `id` | Unique identifier |
| `actionid` | Which verb triggers this recipe (e.g. `"study"`, `"work"`, `"dream"`) |
| `requirements` | What cards/aspects must be present to fire |
| `effects` | What cards are created/destroyed (positive = create, negative = destroy) |
| `aspects` | Tags added to the recipe result |
| `linked` | Chain to another recipe after this one completes (with `chance` %) |
| `alt` | Alternative outcomes (checked before `linked`) |
| `warmup` | Duration in seconds |
| `craftable` | If `true`, player can manually trigger this recipe |
| `startdescription` | Text shown while recipe is running |
| `description` | Text shown when recipe completes |
| `slots` | Additional card slots that appear during this recipe |

### Comments in the JSON

Cultist Simulator's engine **ignores unknown fields**, so comments are embedded as extra JSON keys. There is no standard — the codebase uses several styles interchangeably:

```json
"comments": "This explains why this recipe exists"
"comment": "Description written by GPT-3.5, might want to redo with GPT-4 later"
"comment from Immanuelle": "We are forming a polycule here"
```

These are NOT standard JSON comments (JSON has no comment syntax). They're just fields the engine skips. This means:
- You can't comment out a line with `//`
- You can't use `/* */` blocks
- Comments must be valid JSON key-value pairs
- The field name doesn't matter as long as it's not a recognized engine field

### Content Inheritance

Elements can inherit from templates using `"inherits"`:
```json
{
    "id": "vault_athiban_abyss",
    "inherits": "_vault.exile",
    "label": "Althiban Abyss",
    ...
}
```
The child only needs to specify fields it overrides. Convention: templates start with `_`.

### Load Order

- Files are loaded alphabetically within each directory
- Files prefixed with `z_` load last (used to override earlier content, e.g. `z_levers.json`)
- The `content/overwriting/` folder exists specifically to override base-game content

---

## Directory Map

### Root

```
Catgirls-from-space/
├── synopsis.json          # Mod metadata (name, author, version)
├── README.md              # Player-facing documentation
├── todo.md                # Development roadmap
├── LICENSE                # MIT License
├── cover.png              # Steam Workshop cover image
├── !runClaude.bat         # Dev tool: launches Claude in repo
├── content/               # ALL game content (JSON files)
└── images/                # ALL art assets
```

### `content/` — Game Content

#### Core Systems (`content/core/`)

The engine's fundamental mechanics. Edit these carefully — everything depends on them.

| File | Defines |
|---|---|
| `aspects.json` | All 12 core aspects (Hand, Heart, Winter, Secret Histories, Lantern, Grail, Edge, Knock, Forge, Moth, Dance, Imbalance) + 4 veneration types + technical aspects (frozen, facility, local, text languages) |
| `verbs.json` | Player action slots: Surf, Work, Dream, Talk/Communicate, Study, Explore, plus their card slot requirements |
| `time.json` | Time passage, seasons, money depletion, sickness, starvation, romance tick |
| `inductions.json` | Random events triggered by high aspect values (e.g. 10% chance of heart induction) |
| `general_recipes.json` | Core recipes: enlensing, intro sequences, legacy hooks, billionaire intro |

#### Starting the Game (`content/intro/`)

| File | Defines |
|---|---|
| `legacy.json` | 7 playable legacies: Conspiracy Theorist, Billionaire, Physician, Detective, Dancer, Exile, Cryonicist |
| `acolyte_elements.json` | Starting cards given to each legacy |
| `acolyte_intro_recipes.json` | Tutorial/intro recipe sequences |
| `disable legacies.json` | Which base-game legacies are blocked |

#### Endings (`content/endings/` + `content/endings.json`)

| File | Defines |
|---|---|
| `endings.json` (in folder) | Major/minor victories and standard failures |
| `DLC_EXILE_endings.json` | Exile-specific endings |
| `DLC_GHOUL_endings.json` | Ghoul DLC endings |
| `DLC_PRIEST_endings.json` | Priest DLC endings |
| `enemy ascension.json` | Endings where the enemy faction wins |
| `endings.json` (root content) | Top-level ending definitions |

#### Characters & Followers (`content/people/`)

Named characters with 4-stage progression: acquaintance → follower → disciple → exalted.

| File | Defines |
|---|---|
| `lilith.json` | Lilith (all 4 stages + corpse/prisoner/frozen states) |
| `ethan.json` | Ethan |
| `evelyn.json` | Evelyn |
| `generic_followers.json` | Template followers (non-unique) |
| `generic_secrethistories_follower.json` | Secret Histories aligned followers |
| `hirelings.json` | Temporary hired characters |
| `hunters.json` | Enemy hunter characters |
| `followers_cryonics.json` | Cryonics-capable followers |
| `unique_prisoners.json` | Special prisoner characters |
| `independent.json` | Independent (non-recruitable) characters |

Character ID convention: `name_a` (acquaintance), `name_b` (follower), `name_c` (disciple), `name_d` (exalted).

#### Romance (`content/romance/`)

| File | Defines |
|---|---|
| `polycule.json` | The polycule element (multi-partner relationships) |
| `new romance.json` | Romance initiation recipes |
| `talk_l_romance.json` | Romance dialogue/interaction recipes; heavily commented with design rationale |

#### Books & Study (`content/books/`)

| File | Defines |
|---|---|
| `new books.json` | All book elements — each has `xtriggers` mapping `book_to_lore` → a specific fragment. **Many have GPT-3.5 comments noting descriptions need rewriting.** |
| `new study system.json` | The single recipe for studying books (converts `text` → lore via `book_to_lore` trigger) |
| `books_other.json` | Additional/supplementary books |
| `disabling recipes.json` | Recipes that disable base-game book behavior |

**How books work:** A book element has `"aspects": {"text": 1}` and `"xtriggers": {"book_to_lore": "fragmentXXX"}`. When you study it, the `studybook_new` recipe fires, which has `"aspects": {"book_to_lore": 1}`, triggering the xtrigger to produce the fragment.

#### Lore Fragments (`content/fragments/`)

The largest subsystem by file count (~37 files). Handles fragment progression, splitting, combining, and "enlensing" (perspective shifts).

| File Pattern | Defines |
|---|---|
| `fragment breakdown.json` | Splitting higher fragments into lower ones |
| `fragment_updates_*.json` | Progressive description changes as you level fragments |
| `lore_inversions*.json` | Inverting lore perspective (multiple files) |
| `influence_inversions*.json` | Inverting influence perspective |
| `lore combining recipes*.json` | Combining 2 low fragments into 1 higher fragment |
| `influence combining recipes*.json` | Same for influences |
| `hand fragment editing.json` | Hand faction fragment modifications |
| `fragments_exhausted.json` | Exhaustion mechanic (prevents re-study) |
| `study renew lore.json` | Refreshing exhausted fragments |
| `add xtrigger rejuvenation.json` | Fragment rejuvenation triggers |

Fragment ID convention: `fragmentedge`, `fragmentedgeb`, `fragmentedgec` ... (a=level1, b=level2, etc.)

#### Dreaming & Mansus (`content/dreaming/`)

| File | Defines |
|---|---|
| `dreaming.json` | Core dream entry system |
| `dream_general.json` | General dream outcomes |
| `dream_mansus.json` | Mansus (dream realm) navigation |
| `dream mansus new recipes.json` | Additional Mansus recipes |
| `entering mansus with lore types.json` | How different lore types let you enter Mansus |
| `stag door answers.json` | Stag Door puzzle solutions |

#### Mansus Elements (`content/mansus/`)

Mansus locations and portal definitions (separate from dreaming recipes).

#### Capers & Expeditions (`content/capers and expeditions/`)

~27 files. Heist planning and vault exploration.

| File Pattern | Defines |
|---|---|
| `capers and expeditions.json` | Core caper mechanics |
| `explore_vaultentry.json` | How you enter vaults |
| `explore_vaults_a_capital.json` through `_h_floating.json` | 8 vault location sets (A-H, roughly by geography) |
| `explore_obstacles_curses.json` | Curse-type obstacles |
| `explore_obstacles_guardians.json` | Guardian-type obstacles |
| `explore_obstacles_perils.json` | Peril-type obstacles |
| `explore_obstacles_seals.json` | Seal-type obstacles |
| `explore_mothascension.json` | Moth ascension expedition |
| `explore_heartascension.json` | Heart ascension expedition |
| `exile_vaults.json` | Exile-specific vaults (use `"inherits": "_vault.exile"`) |
| `secret histories cult business.json` | Secret Histories faction business |

#### Exile DLC Content (`content/exile/`)

The largest subsystem (~37 files). A mid-game starting legacy with chase mechanics.

| File | Defines |
|---|---|
| **Core:** `exile_elements.json`, `exile_recipes.json`, `exile_verbs.json` | Base exile mechanics |
| **Jobs:** `exile_jobs.json`, `exile_job_recipes.json` | Exile-specific employment |
| **Wounds:** `emotional wound elements.json`, `mental wound elements.json`, `emotional_wound_recipes.json`, `mental_wound_recipes.json`, `exile_wound_recipes.json`, `relinquish mental wounds.json` | Trauma/wound system |
| **Cryonics:** `cryonics_elements.json`, `cryonics_recipes.json` | Freezing/revival within exile |
| **Hunting/Chase:** `exile_hunting.json`, `hunting escape.json`, `exile_scout.json`, `exile_rkx.json`, `exile_rkx_foe.json`, `rkx_emotional.json` | Reckoner pursuit mechanics |
| **Exploration:** `hq_exploration.json`, `Explore city automatic.json`, `spaces.json` | Exile-specific locations |
| **Detective:** `exile_detective_elements.json`, `exile_detective_recipes.json` | Detective path within exile |
| **Other:** `exile_opx.json`, `exile_use.json`, `exile_send.json`, `exile_relinquish.json`, `exile_decks.json`, `my opportunities.json`, `years_magic.json`, `hints.json`, `test poison.json`, `shrine consecration.json`, `adding local.json`, `exile_lore_elements.json`, `exile initial dreams.json` | Various exile systems |
| **Endings:** `endings.json` | Exile-specific endings |

#### Ascension Paths

**Catgirl Ascension (`content/catgirl ascension/`)** — 11 files:
| File | Defines |
|---|---|
| `ascension_elements.json`, `ascension_recipes.json` | Core ascension mechanics |
| `catgirl_ascension_elements.json`, `catgirl_ascension_marks.json` | Catgirl-specific ascension cards and marks |
| `catgirl 1to2.json` | Progression between catgirl stages |
| `catgirl waystag rituals.json` | Waystag ritual recipes |
| `alo ascensions.json` | Advanced ALO ascensions |
| `stag_door_story_moments.json` | Narrative beats at the Stag Door |
| `leaving capital.json` | Leaving the capital city |
| `exile initial dreams.json` | Exile's early dream content |
| `catgirl ascension misc elements.json` | Miscellaneous ascension elements |

**Capitalist Ascension (`content/capitalist ascension/`)** — 3 files:
| File | Defines |
|---|---|
| `evil endgame elements.json` | Hand faction endgame cards |
| `evil endgame recipes.json` | Hand faction endgame recipes |
| `capital_endgame_script_recipes.json` | Scripted capitalist endgame sequences |

#### Culting (`content/culting/`)

| File | Defines |
|---|---|
| `culting.json` | Core cult management |
| `cults.json` | Cult definitions and types |
| `catgirl cult promotions.json` | Catgirl faction promotions |
| `cult business.json` | Cult business operations |
| `auction addition.json` | Auction house mechanics |

#### Influencer / YouTube (`content/influencer/`)

| File | Defines |
|---|---|
| `influencer_elements.json` | YouTube channel resources |
| `influencer_recipes.json` | Content creation recipes |
| `influencer_decks.json` | Video fragment decks |
| `influencer study.json` | Study through video |
| `influencer funds recipes.json` | Monetization |
| `influencer health recipes.json` | Health effects of influencing |
| `devotees.json` | Gaining followers through influence |
| `video fragment routing.json` | How fragments distribute through videos |
| `lore level markers.json` | Lore progression tracking |

#### Dancer Legacy (`content/dancer legacy/`)

| File | Defines |
|---|---|
| `dancer_elements.json`, `legacy_dancer_elements.json` | Dancer-specific cards |
| `dancer begin.json`, `dancer recipes.json` | Early dancer progression |
| `dancer final lessons.json` | Late-game dancer content |
| `dancer stag door.json` | Dancer's Stag Door interactions |
| `ecdysis club recipes.json` | Ecdysis Club location recipes |
| `scars.json` | Scar mechanic |
| `sulochanachat dancer.json` | Sulochana NPC interaction |

#### Detective (`content/detective/`)

| File | Defines |
|---|---|
| `detective_stuff.json` | Detective-specific elements and recipes |
| `legacy_detective_elements_overwrite.json` | Overrides for detective legacy |
| `legacy_detective_recipes_overwrite.json` | Recipe overrides for detective |

#### Card Decks (`content/decks/`)

| File | Defines |
|---|---|
| `contacts.json` | ~50 named acquaintance cards drawn randomly |
| `events.json` | Random event deck |
| `influences.json` | Influence card pool |
| `location_decks.json` | Location cards |
| `seasons.json` | Seasonal event cards |
| `tomes.json` | Book/tome cards |
| `vaults.json` | Vault location cards |
| `vault_rewards.json` | Rewards from vaults |
| `tryagain.json` | Default fallback deck |

#### Combat (`content/long_foe_combat/`)

| File | Defines |
|---|---|
| `berserker elements.json` | Berserker (Long) enemy cards |
| `berserker recipes.json` | Combat recipes vs berserkers |
| `berserker long decks.json` | Berserker combat decks |
| `long_elements.json` | General Long combat elements |
| `long_recipes.json` | Long combat recipes |
| `long_recipes_attacks.json` | Attack-specific recipes |

#### Plagues (`content/plagues/`)

Individual files per aspect: `edge.json`, `forge.json`, `grail.json`, `heart.json`, `knock.json`, `lantern.json`, `moth.json`, `winter.json`, `secret.json`, plus:
- `plague deck.json`, `plague deck cards.json` — plague card pools
- `plague endings.json` — plague-caused endings
- `inflict_random_plague.json` — random plague assignment

#### Other Content Systems

| File/Folder | Defines |
|---|---|
| `content/influences/` | Influence definitions, description overwriting, study recipes |
| `content/investing/` | `capitalismelements.json` (money cards), `moneyskill_recipes.json` (earning money) |
| `content/items/` | `tools.json` (equipment), `ingredients.json` (crafting materials) |
| `content/locations/` | `locations.json` — city locations |
| `content/levers/` | `levers.json`, `z_levers.json` — lever mechanics (z_ loads last to override) |
| `content/lore/` | `hand_fragments.json`, `challenges.json`, `study_3_research.json`, `lore hints.json`, `hand project.json` |

#### Overwriting Base Game (`content/overwriting/`)

Files that replace original Cultist Simulator content:

| File | What It Overwrites |
|---|---|
| `city_locations.json` | Base game city locations |
| `edit_elements.json` | Various base game elements |
| `kill followers.json` | Follower death mechanics |
| `derange followers.json` | Follower madness mechanics |
| `secret histories followers a/b/c/d.json` | Secret Histories follower variants |
| `secret histories followers uqgroups.json` | Uniqueness groups for SH followers |
| `talking overwrite.json` | Dialogue system modifications |

#### Ability Overwriting (`content/basic ability overwriting/`)

| File | Defines |
|---|---|
| `abilities.json` | Replacement ability definitions |
| `influences.json` | Replacement influence abilities |
| `skills.json` | Replacement skill definitions |
| `study_2_stat_gains.json` | Stat gain recipes from studying |

#### Loose Files in `content/`

| File | Defines |
|---|---|
| `cultures.json` | UI text localization (English only). ~160 UI labels. |
| `endings.json` | Top-level ending definitions |
| `entrapment.json` | Risk/entrapment mechanics |
| `painting.json` | Painting creation system |
| `poppy.json` | Poppy Lascelles questline |
| `deaths overwriting.json` | Death mechanic overrides |
| `byt legacy overwriting.json` | "Bright Young Thing" legacy overrides |
| `uniqueness groups.json` | Global uniqueness group definitions |
| `unimplemented jobs.json` | Placeholder jobs (not yet functional) |
| `misc auctionables.json` | Miscellaneous auction items |

#### Misc / Unsorted (`content/misc probably needs better categorization or merging/`)

| File | Defines |
|---|---|
| `begging.json` | Begging mechanic |
| `murder hunter.json` | Hunter murder recipes |

---

### `images/` — Art Assets

```
images/
├── aspects/          # Aspect icons (edge, grail, hand, imbalance, moth, veneration*)
├── burns/            # Abstract symbols (dialectics, nsa, spiral, triangle)
├── elements/         # Card illustrations (100+ files — the bulk of art)
├── endings/          # Ending screen artwork
├── legacies/         # Legacy selection portraits
├── statusbaricons/   # UI status bar icons
├── verbs/            # Action/verb slot icons
├── neko.legacy.png   # Catgirl legacy image
├── new mansus.png    # New Mansus artwork
├── old mansus.png    # Old Mansus artwork (reference?)
└── triangle.svg      # SVG asset
```

Image files are referenced by their filename (without extension) in element `"icon"` fields.

---

## Known Pain Points

### No Validation or Testing Framework
- Cultist Simulator has no built-in JSON validator for mod content
- Typos in IDs (e.g. `"fragmenthert"` instead of `"fragmentheart"`) silently fail
- Broken recipe chains only show up when you play through that content
- No way to unit-test individual recipes outside the full game

### Comments Are Fragile
- JSON has no comment syntax; comments are stored as ignored fields
- No consistent convention: `"comments"`, `"comment"`, `"comment from Immanuelle"` are all used
- Easy to accidentally use a field name the engine DOES recognize
- Comments don't appear in any tooling or editor outline

### Localization
- `cultures.json` only has English — the structure supports more but nothing is translated

### Book Languages (Intentionally Disabled)
- The base game requires players to learn ancient languages (Latin, Greek, Sanskrit, etc.) before reading certain books
- This mod intentionally disables that mechanic — it doesn't fit the mod's vision
- Some book `"comments"` note "Cannot be the sanskrit book since we do not use foreign languages in this mod"
- This is NOT a translation/localization issue — it's a deliberate game design choice to remove the language-gating mechanic

### Organizational Debt
- The `misc probably needs better categorization or merging/` folder name speaks for itself
- Some systems span multiple directories (exile content touches `exile/`, `capers and expeditions/`, `catgirl ascension/`, `endings/`)
- Loose JSON files in `content/` root could be organized into folders
- Duplicate endings files exist at both `content/endings.json` and `content/endings/endings.json`
- The `todo.md` acknowledges "unnecessary bloat" and "content duplication"

### GPT-Generated Content
- Multiple books have `"comment": "Description written by GPT-3.5, might want to redo with GPT-4 later"`
- No systematic tracking of which content was AI-generated vs hand-written
