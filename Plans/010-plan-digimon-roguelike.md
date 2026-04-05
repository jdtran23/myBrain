# Plan 010: Digimon-Inspired Roguelike — From Zero to Playable Alpha
Status: Draft

## Objective
Joe — a complete game dev beginner with C#/.NET experience — builds a playable roguelike alpha featuring original digital creatures with a branching evolution system, turn-based Mystery Dungeon-style combat, and procedural dungeons. The project doubles as a structured learning journey through Godot 4 (C#), game design fundamentals, and roguelike architecture. Success = a playable 5-floor dungeon with 12+ creatures across 4 evolution stages, functional combat, and in-run evolution by ~month 5.

## Non-goals
- **Shipping a commercial product** — this is a learning project first, game second
- **Original pixel art** — use placeholders and free assets throughout; art polish is post-alpha
- **Multiplayer or online features** — single-player only
- **Mobile/web export** — desktop only (C# in Godot can't export to web)
- **200-creature Pokédex** — alpha targets 12-16 creatures; expand later
- **Complex narrative/story** — minimal framing; gameplay-first
- **Formal ECS architecture** — use Godot's scene/node OOP composition
- **Audio/music production** — use free placeholder audio or silence

## AI Usage Declaration
- **AI will generate:** Boilerplate Godot scene setup, dungeon generation algorithms, A* pathfinding integration, damage formula implementations, data models (JSON creature definitions), unit test scaffolding, FOV algorithm code, state machine skeletons
- **AI will NOT decide:** Creature designs and names (Joe's creative vision), evolution tree branching choices (game feel), difficulty balance numbers (requires playtesting), which features to cut when scope pressure hits, whether the game is fun (human judgment only), IP/branding decisions

## Risks & Rabbit Holes
### AI-Specific Risks
- **GDScript hallucination** — Most Godot tutorials and training data use GDScript. AI will frequently generate GDScript instead of C# or hallucinate C# APIs that don't exist in Godot. **Mitigation:** Always verify generated code compiles in Godot. @gardevoir teaches GDScript→C# translation patterns early.
- **Outdated Godot APIs** — Godot 4 broke many Godot 3 APIs. AI may reference `KinematicBody2D` (Godot 3) instead of `CharacterBody2D` (Godot 4). **Mitigation:** Pin to Godot 4.x docs, verify against official C# API reference.
- **Over-engineered architecture** — AI tends to suggest enterprise patterns (abstract factories, dependency injection) for a solo game project. **Mitigation:** KISS principle. If Joe can't explain why a pattern is needed, don't use it.
- **Balance number hallucination** — AI will confidently generate stat tables and damage formulas that are wildly unbalanced. **Mitigation:** All balance numbers are placeholders until playtested. Never trust AI-generated balance.

### Project Risks
- **Scope creep** — "Just one more creature/feature" is the #1 risk. Alpha scope is locked after Phase 4 design review.
- **Tutorial hell** — Spending weeks watching tutorials instead of coding. Each phase has a hard "stop learning, start building" boundary.
- **Motivation cliff** — The gap between "basic prototype" and "fun game" is demoralizing. Phases are designed so every 2-3 weeks Joe has something playable.
- **Godot C# ecosystem gaps** — Fewer C# examples/plugins than GDScript. Some community resources won't apply. Joe must learn to translate.
- **Evolution system complexity** — Branching evolution with requirements is genuinely complex. Phase 6 is scoped conservatively (3 evolution chains first, expand later).

### Rollback Strategy
- Git repo initialized in Phase 1. Commits at every task boundary.
- Each phase produces a runnable build. If a phase goes wrong, revert to prior phase's build.
- Evolution system (Phase 6) is designed as an overlay — the game works as a plain roguelike without it.

## Overall Success Metric
Joe can sit down and play a complete run: enter a procedurally generated dungeon, fight enemies with turn-based combat, evolve a Rookie creature through Champion and Ultimate to Mega across 5 floors, die and start over, and want to try again.

---

## Phase 1: Creative Foundation & Environment Setup (Weeks 1-2)
> *Establish the original IP direction, install Godot, and verify the C# development pipeline works end-to-end.*

**Gate:** human-approval (IP naming/branding decisions require Joe's creative input)
**Dependencies:** None

- [ ] **Task 1.1:** Play reference games for genre literacy
  - Play DCSS (free, browser version at https://crawl.develz.org/play.htm) for 3-5 hours — focus on how turn-based grid movement *feels*
  - Watch 1-2 Mystery Dungeon gameplay videos (Shiren the Wanderer or Pokémon Mystery Dungeon)
  - Jot down 5 things you liked and 5 things you didn't about each
  - Verify: Joe has a written list of "I want my game to feel like X" reference points

- [ ] **Task 1.2:** Establish original IP direction — creature branding
  - @rotom drafts 3 naming/branding proposals for the creature system (e.g., "DataMon," "ByteBeasts," "NetCreatures") with evolution terminology (e.g., "DataShift," "ByteEvolve," "Compile")
  - @klefki reviews proposals for coherence, appeal, and IP safety
  - Joe picks the direction he likes — this is a creative decision, not an AI decision
  - Verify: Joe has chosen a creature brand name and evolution term that are (a) original, (b) not trademarked (quick USPTO/Google search), (c) he's excited about

- [ ] **Task 1.3:** Install Godot 4 (.NET build) and verify C# pipeline
  - Download Godot 4.x **.NET build** (not standard build) from https://godotengine.org/download
  - Install .NET 8+ SDK if not already present
  - Create a new Godot project, add a C# script, run it — confirm `GD.Print("Hello from C#")` appears in output
  - Pin to a specific Godot version (e.g., 4.3.x) — do NOT upgrade mid-project unless a blocking bug requires it. Record the version in the project README.
  - @gardevoir explains: Godot project structure (project.godot, .tscn scenes, .cs scripts), the scene tree mental model, and how C# integrates differently from GDScript
  - Verify: `dotnet --version` returns 8.x+, Godot opens, a C# script compiles and runs, generated `.csproj` contains `<TargetFramework>net8.0</TargetFramework>` (not net6.0), Joe can articulate what a Node, Scene, and Signal are

- [ ] **Task 1.4:** Initialize Git repository for the game project
  - Create project directory (e.g., `C:\Users\jdtra\Projects\[creature-name]-roguelike`)
  - Initialize git repo with a proper `.gitignore` for Godot (use https://github.com/github/gitignore/blob/main/Godot.gitignore)
  - First commit: empty Godot project with .gitignore
  - Verify: `git log` shows initial commit, `.godot/` and `*.import` are in .gitignore

- [ ] **Task 1.5:** Complete Godot "Your First 2D Game" tutorial (C# version)
  - Follow https://docs.godotengine.org/en/stable/getting_started/first_2d_game/index.html
  - **Must use C# version** of every script (toggle in docs)
  - @gardevoir provides checkpoint explanations at: (a) scene instancing, (b) signal connections in C#, (c) the `_Process` vs `_PhysicsProcess` distinction
  - Verify: The "Dodge the Creeps" game runs, Joe can explain how scenes, signals, and the game loop work

### Acceptance Criteria (Phase 1)
- [ ] Joe has played at least one roguelike and can describe 3 design elements he wants in his game
- [ ] Original creature IP name and evolution term chosen (not Digimon/trademarked)
- [ ] Godot 4 .NET build installed, C# script compiles and runs
- [ ] Git repo initialized with proper .gitignore, first commit made
- [ ] "Dodge the Creeps" tutorial complete and running in C#
- [ ] Joe can explain: Node, Scene, Signal, `_Ready()`, `_Process()` in his own words

---

## Phase 2: Godot Fundamentals & Grid Movement (Weeks 2-4)
> *Learn core Godot 2D systems and build the absolute minimum roguelike: a character moving on a tile grid.*

**Gate:** standard review gates
**Dependencies:** Phase 1 complete

- [ ] **Task 2.1:** @gardevoir teaches GDScript→C# translation patterns (cheat sheet)
  - The vast majority of Godot tutorials, Stack Overflow answers, and YouTube content use GDScript. Joe MUST be able to translate on the fly.
  - Cover the 10 most common patterns: `@export` → `[Export]`, `@onready` → `GetNode` in `_Ready()`, GDScript `match` → C# `switch`, signal connection syntax (`Connect` vs `+=`), dictionary syntax, `var` typing, `func` → methods, `$NodePath` → `GetNode<T>`, `preload`/`load` → `GD.Load<T>`, `await` differences
  - Produce a written cheat sheet document (Markdown) that Joe can reference throughout the project
  - Verify: Cheat sheet document exists. Joe can translate a 10-line GDScript snippet to C# using only the cheat sheet.

- [ ] **Task 2.2:** @gardevoir teaches TileMap and grid-based movement concepts
  - Cover: TileMap node, TileSet creation, tile layers (ground, walls, entities), cell coordinates vs pixel coordinates
  - Cover: Why roguelikes use discrete grid movement (not continuous), input handling for 4/8-directional movement
  - Exercise: Create a 10x10 TileMap with floor and wall tiles, move a Sprite2D between cells with arrow keys
  - Verify: Joe completes the exercise, sprite snaps to grid cells, walls block movement

- [ ] **Task 2.3:** Build "Dungeon Walker" prototype — static map with grid movement
  - Create a 20x20 hardcoded map (walls around edges, some interior walls)
  - Player character (colored rectangle) moves tile-by-tile with arrow keys
  - Camera follows player
  - Wall collision prevents movement into wall tiles
  - @porygon implements the initial scene structure and grid movement system
  - @absol reviews: scene organization, C# code style, Godot best practices
  - Verify: Player moves on grid, can't walk through walls, camera follows. Build runs without errors.

- [ ] **Task 2.4:** Add simple enemies with chase AI
  - Place 3-5 enemies (different colored rectangles) on the map
  - Enemies take one step toward the player each turn (simple manhattan distance chase)
  - @gardevoir teaches: the turn-based game loop concept (player acts → enemies act → repeat), why turn-based needs explicit "act" calls not `_Process`
  - Verify: After player moves, each enemy moves one tile toward the player. Movement is strictly turn-based — nothing moves until player inputs.

- [ ] **Task 2.5:** Implement bump-to-attack combat
  - Moving into an enemy = attack. Moving into player = enemy attacks.
  - Simple HP system: Player has 20 HP, enemies have 5 HP. Attack deals 3 damage.
  - Display HP as a number label above entities
  - Dead entities are removed from the map
  - Win condition: all enemies dead. Lose condition: player HP ≤ 0.
  - @porygon implements combat resolution
  - @absol reviews combat logic
  - Verify: Player can kill enemies by bumping. Enemies can kill player. Game ends on win/lose. No bugs when multiple enemies die same turn.

- [ ] **Task 2.6:** Add game over and restart flow
  - Game Over screen on death with "Try Again" button
  - Victory screen when all enemies defeated
  - "Try Again" reloads the scene from scratch
  - Verify: Full gameplay loop works — play, fight, win or die, restart. No memory leaks on restart (check Godot debugger).

### Acceptance Criteria (Phase 2)
- [ ] "Dungeon Walker" runs: grid movement, wall collision, turn-based enemy chase, bump combat, win/lose/restart
- [ ] Code passes @absol review with zero 🔴 and 🟡 findings
- [ ] Joe can explain: TileMap coordinates, turn-based loop architecture, bump-to-attack pattern
- [ ] Git commits at each task boundary (minimum 5 commits this phase)

---

## Phase 3: Procedural Dungeons & FOV (Weeks 4-7)
> *Replace the static map with generated dungeons and add fog-of-war — the two systems that make it feel like a real roguelike.*

**⚠️ Risk note:** This is the most algorithmically dense phase. If BSP/FOV take longer than expected, use fallback algorithms: random non-overlapping room placement + corridors instead of BSP, and basic raycasting FOV instead of shadow casting. These can be upgraded later without reworking other systems.

**Gate:** standard review gates
**Dependencies:** Phase 2 complete

- [ ] **Task 3.1:** @gardevoir teaches procedural generation concepts
  - Cover: BSP (Binary Space Partitioning) algorithm for room placement, room-and-corridor connection, random vs seeded generation
  - Cover: Why roguelikes need proc-gen (replayability), common pitfalls (unreachable rooms, too-open layouts)
  - Show reference: RogueBasin "Basic BSP Dungeon generation" article
  - Exercise: On paper or whiteboard, trace through BSP splitting a 40x30 grid into 4-6 rooms
  - Verify: Joe can describe the BSP algorithm steps without looking at notes

- [ ] **Task 3.2:** Implement BSP dungeon generator
  - Generate a dungeon with 5-8 rooms connected by corridors
  - Place player in a random room, place stairs-down in a different room
  - Enemies spawn in rooms (1-3 per room, not in player's starting room)
  - @porygon implements the generator with configurable parameters (min/max room size, split depth)
  - Verify: Run generation 10 times — every run produces a connected, playable map. No unreachable rooms. No rooms overlapping. Visual inspection in Godot editor.

- [ ] **Task 3.3:** @gardevoir teaches FOV (Field of View) algorithms
  - Cover: Why FOV matters for roguelikes (tension, exploration, information asymmetry)
  - Cover: Recursive Shadow Casting algorithm — how it works, why it's preferred
  - Cover: Explored vs Visible distinction (seen tiles stay dim, currently visible tiles are bright)
  - Verify: Joe can explain FOV to a rubber duck — what tiles are visible and why

- [ ] **Task 3.4:** Implement FOV with recursive shadow casting
  - Player has vision radius of 8 tiles
  - Unseen tiles are black, explored-but-not-visible tiles are dim, visible tiles are full bright
  - Enemies only visible when in FOV
  - Walls and closed doors block line of sight
  - @porygon implements FOV (can reference RogueBasin's C# shadow casting reference)
  - @absol reviews: algorithm correctness, edge cases (FOV at map edges, single-tile corridors)
  - Verify: FOV looks correct visually. Diagonal corridors don't "leak" light through walls. Performance is fine (< 1ms per FOV calc on a 50x50 map).

- [ ] **Task 3.5:** Implement stairs and multiple floors
  - Walking onto stairs-down generates a new floor (new random dungeon, harder enemies)
  - Floor counter displayed in UI
  - Enemies scale with floor: more enemies or tougher enemies per floor
  - @porygon implements floor transitions
  - Verify: Can descend 5+ floors. Each floor is unique. Difficulty visibly increases. No crashes on floor transition.

- [ ] **Task 3.6:** Commit and playtest checkpoint
  - Full playtest: Start game → explore dungeon → fight enemies → descend stairs → die or reach floor 5
  - Note what feels good and what feels bad (write it down!)
  - @absol reviews accumulated code for the phase
  - Verify: Game is playable end-to-end with proc-gen dungeons and FOV. Joe's playtest notes exist.

### Acceptance Criteria (Phase 3)
- [ ] Dungeons are procedurally generated and always connected/playable
- [ ] FOV system works — unexplored tiles hidden, explored tiles dimmed, visible tiles bright
- [ ] 5+ floor descent works with scaling difficulty
- [ ] Code passes @absol review with zero 🔴 and 🟡 findings
- [ ] Joe has written playtest notes identifying at least 3 "feels good" and 3 "needs work" items

---

## Phase 4: Game Design — Creatures, Combat & Evolution (Weeks 7-9)
> *Design the creature roster, type system, stat model, combat formula, and evolution tree ON PAPER before writing any implementation code.*

**Gate:** human-approval (Joe must approve creature designs — this is his creative vision)
**Dependencies:** Phase 3 complete

- [ ] **Task 4.1:** @rotom designs the type and stat system
  - Type triangle (e.g., Virus > Data > Vaccine > Virus) + elemental types (Fire, Water, Electric, Plant — start with 4)
  - Stat model: HP, SP (skill points), ATK, DEF, INT (special attack), SPD
  - Type effectiveness chart (0.5x / 1.0x / 2.0x multipliers)
  - @klefki reviews: Is the type system deep enough to create interesting choices but simple enough for a roguelike? Are 4 elements enough or too many for alpha?
  - Verify: Type chart exists as a document. No type is strictly dominant. Every element has at least one weakness and one resistance.

- [ ] **Task 4.2:** @rotom designs the combat formula
  - Damage formula: base damage, type modifier, random factor, stat scaling
  - SP cost system for skills (basic attack is free, skills cost SP)
  - Speed-based turn ordering for multi-creature scenarios
  - Status effects: at least Poison, Paralysis, and one buff (ATK Up)
  - @klefki reviews: Does the formula produce interesting decisions? Is there a reason NOT to always use the strongest skill? Is SP pressure real?
  - Verify: Hand-calculate 10 example combat scenarios on paper. Fights should last 3-6 turns on average, not 1-shot or 20-turn slogs.

- [ ] **Task 4.3:** @rotom designs the starter creature roster (alpha scope)
  - 12-16 creatures across 3 evolution chains × 4 stages:
    - Chain A: e.g., Fire/Vaccine path (aggressive attacker line)
    - Chain B: e.g., Water/Data path (defensive/healer line)
    - Chain C: e.g., Electric/Virus path (speed/debuff line)
  - Each creature: name, type, element, base stats, 2-4 moves, evolution requirements
  - Wild enemy creatures: 4-6 non-evolving species for dungeon encounters
  - @klefki reviews: Are chains distinct enough to feel different to play? Is each chain viable for a full run? Are evolution requirements achievable within normal gameplay?
  - Verify: Creature roster document exists. Each chain plays differently. No creature is obviously useless or overpowered on paper.

- [ ] **Task 4.4:** @rotom designs the in-run evolution system
  - How evolution energy is gained (combat XP? items? floor progress?)
  - What triggers evolution (threshold + choice from available paths)
  - What changes on evolution (stat boost, new moves, type may change, sprite changes)
  - How devolution works (or doesn't — design decision for Joe)
  - Floor-gating: earliest floor for each stage (e.g., Champion at floor 2+, Ultimate at floor 4+, Mega at floor 7+)
  - @klefki reviews: Does the evolution pacing feel rewarding? Is reaching Mega aspirational but achievable? Does the system create meaningful choices (which path to evolve)?
  - Verify: Simulate 3 hypothetical runs on paper — choose different evolution paths, check that each feels viable and distinct.

- [ ] **Task 4.5:** Joe reviews all designs and makes creative decisions
  - Read all design docs from Tasks 4.1-4.4
  - Decide: creature names, visual direction (even rough sketches or color descriptions), any design changes
  - Lock alpha scope: these creatures, this type chart, this combat system. No additions until alpha is playable.
  - Verify: Joe has signed off on all game design documents. Scope is locked and documented.

### Acceptance Criteria (Phase 4)
- [ ] Type system designed and reviewed (type chart, elemental matchups)
- [ ] Combat formula designed with hand-calculated example scenarios
- [ ] 12-16 creature roster across 3 evolution chains, each with stats/moves/requirements
- [ ] Evolution system designed (energy gain, triggers, effects, floor-gating)
- [ ] All designs pass @klefki review with zero 🔴 and 🟡 findings
- [ ] Joe has approved all designs and locked alpha scope

---

## Phase 5: Combat System Implementation (Weeks 9-12)
> *Implement the designed stat system, type chart, damage formula, skills, and items — turning the prototype into an actual RPG.*

**Gate:** standard review gates
**Dependencies:** Phase 4 complete (designs locked)

- [ ] **Task 5.1:** @gardevoir teaches data-driven design and JSON data loading in Godot
  - Cover: Why game data lives in JSON (not hardcoded) — moddability, separation of concerns, easy balancing
  - Cover: Loading JSON in Godot C# (`FileAccess`, `Json.ParseString()`)
  - Cover: C# data classes for creatures, moves, type chart
  - Exercise: Create a JSON file with 2 creature definitions, load and print them in Godot
  - Verify: Joe can load JSON data into C# objects in Godot without assistance

- [ ] **Task 5.2:** Implement creature data model and JSON loader
  - C# classes: `CreatureSpecies`, `CreatureInstance`, `Move`, `TypeChart`, `EvolutionPath`
  - JSON files: `creatures.json`, `moves.json`, `type_chart.json`
  - Load all creature data at startup, validate no missing references (e.g., a move references a type that exists)
  - @porygon implements data model
  - @absol reviews: data validation, null safety, separation of data from logic
  - Verify: All creature data loads without errors. Unit test: load creatures.json, assert expected count, assert each creature has valid type and moves.

- [ ] **Task 5.3:** Implement the combat system
  - Replace simple "3 damage bump attack" with full damage formula from Phase 4 design
  - Type effectiveness applied (show "Super Effective!" / "Not Very Effective..." text)
  - SP system: skills cost SP, basic attack is free
  - Speed-based turn order when multiple entities act
  - Status effects: Poison (damage per turn), Paralysis (skip turn chance), ATK Up (buff)
  - @porygon implements combat engine
  - @absol reviews: formula correctness vs design doc, edge cases (0 DEF, same-speed ties, status on death)
  - Verify: Fight a creature with type advantage — damage is visibly higher. Use a skill — SP decreases. Get poisoned — take damage each turn. Run 20 fights — average duration matches design target (3-6 turns).

- [ ] **Task 5.4:** Implement items and inventory
  - Item types: Healing Potion (restore HP), SP Potion (restore SP), Data Chip (evolution energy), Antidote (cure status)
  - Items spawn on dungeon floor (random rooms)
  - Player inventory (max 10 items) — pick up with dedicated key, use from inventory menu
  - @porygon implements inventory system
  - @absol reviews: inventory edge cases (full inventory, use last item, use item on full HP)
  - Verify: Can pick up items, use them, effects apply correctly. Inventory displays correctly. Can't exceed max.

- [ ] **Task 5.5:** Replace enemy rectangles with creature encounters
  - Dungeon enemies are now creatures from the roster (wild/non-evolving species)
  - Different creatures on different floors (weaker on floor 1, stronger deeper)
  - Enemies use their actual stats and moves (not just bump-attack)
  - Enemy AI: use skills when SP available, prioritize type-advantaged moves
  - @porygon implements creature-based enemies
  - Verify: Floor 1 enemies are visibly easier than floor 3 enemies. Enemies use skills. Type matchups matter in actual gameplay.

- [ ] **Task 5.6:** Implement basic HUD
  - Display: Player creature name/stage, HP bar, SP bar, current floor, minimap or floor indicator
  - Combat log: last 5 actions shown as text ("Firemon used Flame Burst! It's super effective! 12 damage!")
  - Inventory accessible via hotkey
  - @porygon implements UI
  - Verify: All displayed information is accurate. HUD doesn't obscure gameplay. Combat log correctly reports all actions.

### Acceptance Criteria (Phase 5)
- [ ] Full combat system: damage formula, type effectiveness, skills, SP, status effects
- [ ] Item system: 4+ item types, floor spawns, inventory management
- [ ] Creature-based enemies with stats, moves, and basic AI
- [ ] HUD displaying HP, SP, floor, combat log
- [ ] Code passes @absol review with zero 🔴 and 🟡 findings
- [ ] 10 manual combat encounters work correctly with no formula/logic bugs

---

## Phase 6: Evolution System Implementation (Weeks 12-16)
> *Implement the signature feature: in-run digivolution from Rookie through Mega, with branching paths and meaningful choices.*

**Gate:** standard review gates
**Dependencies:** Phase 5 complete

- [ ] **Task 6.1:** @gardevoir teaches state machine patterns for evolution
  - Cover: State machines for entity lifecycle (Normal → EvolutionReady → Evolving → Evolved)
  - Cover: How to present branching choices to the player (UI considerations)
  - Cover: Data-driven requirements checking (how to evaluate "ATK > 20 AND floor >= 3")
  - Verify: Joe can whiteboard the evolution state machine and requirements evaluation logic

- [ ] **Task 6.2:** Implement evolution energy and tracking
  - Player's creature gains evolution energy from: defeating enemies (scaled by difficulty), picking up Data Chips, completing floors
  - Track cumulative stats for requirement checking (total ATK, total damage dealt, etc.)
  - UI indicator: evolution energy bar or counter, "Evolution Ready!" flash when threshold met
  - @porygon implements evolution tracking
  - Verify: Energy accumulates during gameplay. Threshold notification appears at correct point. Energy values match design doc.

- [ ] **Task 6.3:** Implement evolution selection and execution
  - When evolution energy threshold met + requirements satisfied: pause gameplay, show evolution options
  - Present 1-3 available evolutions with stat previews (show what you'd become)
  - Player chooses → evolution animation (can be simple: flash + sprite swap) → stats update, new moves available
  - Handle edge case: player meets requirements for 0 evolutions (show "not ready" message)
  - @porygon implements evolution flow
  - @absol reviews: state transitions, edge cases (evolve during combat? evolve when at 1 HP?), data integrity after evolution
  - Verify: Evolve a Rookie → Champion → Ultimate across a 5-floor run. Stats change correctly. New moves appear. Old form's data is clean (no ghost references).

- [ ] **Task 6.4:** Implement all 3 evolution chains from the roster
  - Wire up all 12-16 creatures from Phase 4 design
  - Each chain fully playable from Rookie to Mega
  - Verify per-chain: Start as Chain A Rookie, evolve to Chain A Champion, continue to Ultimate, reach Mega. Repeat for B and C.
  - @porygon implements remaining creature data and chain wiring
  - Verify: All 3 chains work end-to-end. Each chain feels mechanically distinct (attacker vs defender vs speed). No missing data references.

- [ ] **Task 6.5:** Mechanical smoke test and playtest (NOT final balance — that's Phase 7)
  - Play 5 full runs (attempt to reach floor 5+ with each chain)
  - Focus: Are mechanics WORKING? Do evolutions fire correctly? Are there crashes, soft-locks, or broken interactions?
  - Record: average floor reached, evolution timing, fight duration, SP economy, item usage
  - Fix mechanical bugs found. Note balance concerns but do NOT spend time fine-tuning numbers — Phase 7 systems (meta-progression, items, QoL) will shift balance further.
  - This is HUMAN judgment — AI cannot tell you if the game is fun
  - Verify: Joe has playtest log with at least 5 runs. All 3 chains mechanically functional. No game-breaking bugs.

### Acceptance Criteria (Phase 6)
- [ ] Evolution energy system functional — accumulates from combat, items, floors
- [ ] Branching evolution UI: player sees options, chooses, creature evolves with stat/move changes
- [ ] All 3 evolution chains playable Rookie → Champion → Ultimate → Mega
- [ ] Code passes @absol review with zero 🔴 and 🟡 findings  
- [ ] 5+ playtest runs logged with mechanical correctness verified
- [ ] No game-breaking bugs — balance tuning deferred to Phase 7

---

## Phase 7: Alpha Polish & Meta-Progression (Weeks 16-20)
> *Add the systems that make runs feel meaningful beyond permadeath: meta-progression, save/load, and quality-of-life polish.*

**Priority tiers:** If behind schedule, cut in this order: Task 7.5 (QoL polish) first, then Task 7.3 (starter selection). Tasks 7.1 (CreatureDex), 7.2 (save/load), and 7.4 (menus/game flow) are **alpha-critical** and must ship.

**Gate:** standard review gates
**Dependencies:** Phase 6 complete

- [ ] **Task 7.1:** Implement the CreatureDex (meta-progression registry)
  - Persistent data file (saved to disk, survives game close)
  - Records every creature form the player has achieved in any run
  - Viewable from main menu: shows silhouettes for undiscovered, full entries for discovered
  - Tracks: times evolved to this form, highest floor reached with this form
  - @porygon implements CreatureDex
  - Verify: Evolve to a new form → CreatureDex updates. Close game → reopen → data persists. New run → previously discovered forms show in dex.

- [ ] **Task 7.2:** Implement save/load for run state
  - Save current run to disk (single save slot, delete on death = permadeath)
  - Save includes: map state, player creature state, inventory, evolution energy, floor number, enemy positions
  - Load from main menu ("Continue Run" button)
  - @gardevoir teaches: JSON serialization of game state, Godot's `user://` save directory
  - @porygon implements save/load
  - @absol reviews: save corruption edge cases, no save scumming (save deleted on load or on death), sensitive data handling
  - Verify: Save mid-run → close game → reopen → load → game state is identical. Die → save is deleted. No way to backup save to cheat death.

- [ ] **Task 7.3:** Implement starting creature selection
  - Main menu: "New Run" → choose your starting Rookie (one from each chain)
  - Show creature stats/type preview for each option
  - Initially only Chain A unlocked — Chain B and C unlock when their Rookie is encountered in the CreatureDex (via wild encounters with those species)
  - @porygon implements creature select
  - Verify: Can select different starting Rookies. Locked chains show as locked until unlock condition met.

- [ ] **Task 7.4:** Add main menu and game flow
  - Main Menu: New Run, Continue Run (if save exists), CreatureDex, Quit
  - Death screen: Show run stats (floor reached, creatures defeated, evolutions achieved), CreatureDex discoveries this run, "Try Again" / "Main Menu"
  - Pause menu (Escape): Resume, Save & Quit
  - @porygon implements menus
  - Verify: Full flow works: Main Menu → New Run → Play → Pause → Resume → Die → Death Screen → Main Menu → Continue (no save) shows disabled.

- [ ] **Task 7.5:** Quality-of-life polish
  - Smooth camera movement (lerp instead of snap)
  - Simple turn transition delay (enemies moves visible, not instant)
  - Damage numbers float up from hit targets
  - Evolution sequence has a brief visual flourish (screen flash, "EVOLVED!" text)
  - Simple sound effects for key actions (attack, evolve, pickup) using free sounds from freesound.org or Kenney
  - @porygon implements QoL features
  - Verify: Game feels better to play. Movements are readable. Evolution moment feels special.

- [ ] **Task 7.6:** Alpha playtest and final balance
  - Full playtest: complete 3+ runs with different chains
  - Test all paths: win run, lose run, save/load mid-run, force devolution, exhaust SP
  - Check CreatureDex persistence across multiple play sessions
  - File any bugs found, fix critical ones
  - Verify: No game-breaking bugs. All 3 chains feel viable. CreatureDex works correctly. Save/load is stable.

### Acceptance Criteria (Phase 7)
- [ ] CreatureDex persists between sessions, tracks all discovered forms
- [ ] Save/load works correctly with permadeath enforcement
- [ ] Starting creature selection with unlock progression
- [ ] Main menu, death screen, pause menu — full game flow
- [ ] QoL: smooth camera, visible turns, damage numbers, evolution VFX
- [ ] 3+ complete playtest runs with no game-breaking bugs
- [ ] Code passes @absol review with zero 🔴 and 🟡 findings
- [ ] Joe declares: "This is a playable alpha I'm proud of"

---

## Design Decisions

| Decision | Choice | Why | Alternative Rejected |
|----------|--------|-----|---------------------|
| Engine | Godot 4 (.NET) | Free, C# support, excellent 2D/TileMap, low learning curve | Unity (overkill for 2D), MonoGame (no editor), Pygame (less tooling) |
| Language | C# | Joe's strongest language, direct transfer to Godot .NET | GDScript (extra language to learn), C++ (overkill) |
| Architecture | Godot scene/node OOP | Natural fit for Godot, beginner-friendly | Formal ECS (over-engineered for solo project) |
| Dungeon gen | BSP (Binary Space Partitioning) | Well-documented, produces good room+corridor layouts, beginner-accessible | Cellular automata (caves, not rooms), Wave Function Collapse (too complex) |
| FOV algorithm | Recursive Shadow Casting | Fast, good visual results, well-documented C# implementations | Raycasting (slow for large areas), Permissive FOV (complex) |
| Data format | JSON | Godot native support, easy to edit, Joe knows JSON | Custom Resource (Godot-specific), YAML (no native support), SQLite (overkill) |
| Evolution scope | 3 chains × 4 stages for alpha | 12-16 creatures is testable and designable; more can be added post-alpha | Full 50+ creature roster (scope trap) |
| Art approach | Colored rectangles → free assets | Gameplay-first; art is the lowest priority until systems are fun | Custom pixel art (time sink, not Joe's skill), AI art (legal concerns, quality) |
| Permadeath model | True permadeath + light meta-progression (CreatureDex) | Core roguelike identity preserved; meta-progression reduces frustration | Full roguelite persistence (undermines run tension), Pure permadeath (too punishing for new players) |

## Validation Strategy
Every AI-generated artifact must have verification steps defined BEFORE generation:
- **Code:** Must compile in Godot without errors. Must pass @absol review. Game must run and demonstrate the feature.
- **Game design docs:** Must pass @klefki review. Must include hand-calculated examples where applicable (combat scenarios, evolution timing).
- **Dungeon generation:** Visual inspection of 10+ generated maps. Must be connected, playable, no degenerate layouts.
- **Balance numbers:** NEVER trusted without playtesting. All values are provisional until Joe plays and adjusts.
- **Godot C# API usage:** Verified against Godot 4.x .NET API reference. GDScript-to-C# translations double-checked.
- **JSON data files:** Schema validated on load. Missing references (creature points to nonexistent move) caught at startup.
- **Save/load:** Roundtrip test — save state, load state, verify identical. Fuzz test with invalid save files.
