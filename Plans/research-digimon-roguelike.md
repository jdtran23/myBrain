# Research: Digimon-Themed Roguelike with Turn-Based RPG Mechanics

**Researcher:** Uxie | **Date:** 2026-04-04 | **Status:** Complete

---

## Key Takeaways

1. **Primary Engine Recommendation: Godot 4 with C#** — Free, open-source, excellent 2D support (TileMap system, built-in A* navigation), supports C# natively via .NET 8+, and has the gentlest learning curve of full-featured engines. Joe's C# knowledge transfers directly.
2. **Start with a minimal roguelike first** — Build a simple "@-walking-on-tiles" prototype before adding any Digimon mechanics. The classic "roguelike in 15 steps" approach from RogueBasin is proven effective.
3. **Digivolution is the killer feature** — The branching evolution tree (one Digimon → multiple possible forms) creates natural replayability that pairs perfectly with roguelike randomness. Per-run evolution with light meta-progression is the sweet spot.
4. **Mystery Dungeon is your design bible** — The Pokémon/Chocobo Mystery Dungeon games are the exact genre intersection (monster collection + roguelike). Study Shiren the Wanderer for pure roguelike mechanics.
5. **Use original creatures inspired by Digimon** — You cannot legally use the Digimon IP. Create your own "Digital Monsters" with similar mechanics but original designs.

---

## 1. Game Engine Selection for Beginners

### Comparison Matrix

| Engine | Language | Learning Curve | 2D Roguelike Fit | Cost | Joe's Skill Match |
|--------|----------|---------------|------------------|------|-------------------|
| **Godot 4** | GDScript, **C#**, C++ | ⭐ Low | ⭐⭐⭐ Excellent | Free (MIT) | **⭐⭐⭐ C# direct** |
| **Unity** | **C#** | ⭐⭐ Medium | ⭐⭐ Good | Free < $200K rev | **⭐⭐⭐ C# direct** |
| **GameMaker** | GML | ⭐ Low | ⭐⭐⭐ Excellent | Free / $99 | ⭐ New language |
| **MonoGame** | **C#** | ⭐⭐⭐ High | ⭐⭐ Good (bare) | Free | **⭐⭐⭐ C# direct** |
| **Pygame** | **Python** | ⭐⭐ Medium | ⭐⭐ Good | Free | **⭐⭐⭐ Python** |
| **Phaser.js** | **TypeScript/JS** | ⭐⭐ Medium | ⭐⭐ Fair | Free | **⭐⭐⭐ TypeScript** |

### Detailed Engine Analysis

#### Godot 4 (⭐ RECOMMENDED)
- **Language:** GDScript (Python-like) or C# via .NET 8+. C# requires downloading the .NET version of the editor.
- **2D Features:** Built-in TileMap system (perfect for grid-based roguelikes), 2D physics, sprite animation, 2D lighting with normal maps, built-in A* pathfinding via `AStar2D`/`AStarGrid2D`
- **Roguelike Suitability:** Excellent. TileMap node is purpose-built for tile-based games. Navigation system has A* built in. Scene tree architecture maps well to entity management.
- **Community:** Rapidly growing. Active Discord, subreddit (r/godot), and community tutorials. Official docs are comprehensive with both GDScript and C# examples.
- **Cost:** Completely free, MIT license. No royalties ever. You own your game completely.
- **Caveat:** C# in Godot cannot export to web platform (only desktop, Android, iOS). GDScript can export everywhere.
- **Why for Joe:** C# transfers directly. The engine is lightweight (~100MB), no bloat. Scene-based design is intuitive for beginners. Error messages are clear. Live editing and hot reload supported.

#### Unity
- **Language:** C#
- **Pros:** Massive tutorial ecosystem, industry standard, huge Asset Store
- **Cons:** Overkill for 2D roguelikes, heavier/slower editor, runtime fee controversy (resolved but eroded trust), complex UI for simple 2D games, frequent API changes between versions
- **Verdict:** Joe *could* use it since he knows C#, but Unity's overhead and complexity make it a poor first-game engine for a solo dev making a 2D roguelike.

#### GameMaker
- **Language:** GML (proprietary, C-like)
- **Pros:** Purpose-built for 2D games, many successful roguelikes made with it (Nuclear Throne, Spelunky)
- **Cons:** Joe would need to learn GML from scratch, not free for full features, less transferable skills
- **Verdict:** Great engine, but learning GML when Joe already knows C# is suboptimal.

#### MonoGame
- **Language:** C#
- **Pros:** Pure code framework, maximum control, used by Stardew Valley
- **Cons:** No editor — everything is code. No built-in tile editor, no scene system, no visual debugging. Steep learning curve for game dev beginners.
- **Verdict:** Too bare-bones for a beginner. Better for experienced devs who want total control.

#### Pygame
- **Language:** Python
- **Pros:** Joe knows Python, excellent roguelike tutorial tradition (the RogueBasin python+libtcod tutorial is legendary), simple API
- **Cons:** Performance ceiling, no built-in editor, limited to 2D, the classic libtcod tutorial is aging, packaging/distribution is painful
- **Verdict:** Good for pure-code prototyping, but lacks the tooling that Godot provides. Worth considering for the first "learning" project.

#### Phaser.js
- **Language:** TypeScript/JavaScript
- **Pros:** Joe knows TypeScript, runs in browser, good for web distribution
- **Cons:** Web-only focus limits options, less game-dev tooling, weaker for complex game state management
- **Verdict:** Fun for web prototypes but not ideal for a feature-rich roguelike.

### Final Recommendation

**Godot 4 with C#** is the clear winner. It directly leverages Joe's strongest language (C#), has purpose-built 2D tile tooling, costs nothing, and has an excellent beginner learning curve. The only real competitor is Pygame for rapid prototyping, but Godot provides better tools for when the project grows.

**Alternative path:** Start with the classic Python roguelike tutorial (rogueliketutorials.com) to learn roguelike concepts, THEN port to Godot C# for the real project.

---

## 2. Roguelike Game Design Fundamentals

### What Defines a "Roguelike"?

#### The Berlin Interpretation (2008)
The Berlin Interpretation was created at the International Roguelike Development Conference 2008. It defines "roguelike" through high-value and low-value factors:

**High-Value Factors:**
- **Random environment generation** — World is procedurally generated for replayability
- **Permadeath** — Death is permanent; you start over with a new character
- **Turn-based** — Each command = one action. No time pressure.
- **Grid-based** — World represented by a uniform grid of tiles
- **Non-modal** — Movement, battle, and other actions occur in the same mode
- **Complexity** — Enough depth to allow multiple solutions to problems
- **Resource management** — Limited resources (food, potions, etc.)
- **Hack'n'slash** — Killing monsters is central
- **Exploration and discovery** — Careful exploration of dungeon levels required

**Low-Value Factors:**
- Single player character
- Monsters similar to players (same rules apply)
- Tactical challenge
- ASCII display (traditional but not required)
- Dungeons (rooms + corridors)
- Numbers shown (HP, stats visible)

**Note:** The Berlin Interpretation is controversial. Darren Grey's "Screw the Berlin Interpretation" argues it's outdated. Modern usage is broader — "roguelike" often means any game with permadeath + procedural generation. "Roguelite" typically means roguelike + meta-progression (Hades, Dead Cells).

#### Roguelike vs Roguelite
| Feature | Traditional Roguelike | Roguelite |
|---------|----------------------|-----------|
| Death | Full permadeath | Some persistence between runs |
| Controls | Turn-based, grid | Often real-time, free movement |
| Graphics | ASCII or tiles | Usually graphical |
| Examples | DCSS, Caves of Qud, Shiren | Hades, Dead Cells, Slay the Spire |
| **Joe's Game** | **Core mechanics from here** | **Meta-progression from here** |

### Minimum Viable Systems for a Roguelike

Based on RogueBasin's "How to Write a Roguelike in 15 Steps":

1. **Map system** — 2D grid of tiles (walls, floors, doors)
2. **Player movement** — @ character moving on the grid with wall collision
3. **Dungeon generation** — Procedural rooms + corridors algorithm
4. **Field of View (FOV)** — What the player can currently see
5. **Monsters** — At least one enemy type with simple AI (chase player)
6. **Turn system** — Player acts, then all monsters act
7. **Combat** — HP-based, bump-to-attack
8. **Items** — Pickups with effects (healing potions, etc.)
9. **Inventory** — Player can carry and use items
10. **Multiple levels** — Stairs down to harder floors

Everything else (magic, shops, quests, digivolution) is built ON TOP of these fundamentals.

### Games to Study

| Game | Why Study It | Relevance |
|------|-------------|-----------|
| **Shiren the Wanderer** | The gold standard Mystery Dungeon game. Pure roguelike with grid movement, turn-based. | Closest mechanical template for Joe's game |
| **Pokémon Mystery Dungeon** | Monster collection + roguelike. Partner/recruit system. Type advantages. | Direct genre match — "monster roguelike" |
| **Chocobo's Mystery Dungeon** | RPG progression (jobs/classes) in roguelike framework | Shows how to add RPG systems to roguelikes |
| **Caves of Qud** | Deep mutation/evolution system, complex builds | Inspiration for branching digivolution system |
| **DCSS (Dungeon Crawl Stone Soup)** | Free, open-source, extensively documented. Best traditional roguelike. | Play this to understand core roguelike feel |
| **Hades** | Best roguelite design — meta-progression that respects player time | Model for how to add persistence between runs |
| **Digimon World** | Original Digimon game with raising/evolution mechanics | Direct IP reference for digivolution systems |
| **Digimon Survive** | Recent Digimon RPG with branching evolution based on choices | Shows modern approach to digivolution in games |
| **Digimon Story: Cyber Sleuth** | Most popular modern Digimon RPG, deep evolution tree | Best reference for how free-form digivolution works in practice |

---

## 3. Turn-Based RPG Mechanics

### Core Combat Loop

A turn-based RPG combat loop for a roguelike:

```
Player's Turn:
  → Choose action: Move / Attack / Use Skill / Use Item / Wait
  → Resolve action (damage calc, effects apply)

Each Monster's Turn (in speed order):
  → AI selects action based on state (chase, attack, flee, use skill)
  → Resolve action

Check for:
  → Deaths (remove dead entities)
  → Status effect ticks (poison, burn, etc.)
  → End-of-turn triggers
```

### Damage Calculation

A simple but expandable formula:

```
Base Damage = Attacker.ATK - Defender.DEF
Type Modifier = TypeChart[Attacker.Type][Defender.Type]  // 0.5x, 1.0x, 2.0x
Random Factor = Random(0.85, 1.15)
Final Damage = Max(1, Floor(Base_Damage * Type_Modifier * Random_Factor))
```

### How Digimon Games Handle Combat

Based on Digimon Story: Cyber Sleuth and Digimon Survive:

**Type System (Digimon's version of elemental advantage):**
- **Virus > Data > Vaccine > Virus** (rock-paper-scissors triangle)
- **Free** type is neutral against all
- Additional elemental types: Fire, Water, Plant, Electric, Earth, Wind, Light, Dark

**Stats used in Digimon games:**
- HP, SP (skill points/mana)
- ATK (physical attack), DEF (physical defense)  
- INT (special attack), SPD (speed/turn order)
- Support Skills (passive abilities gained from evolution)

**What makes Digimon combat unique vs Pokémon:**
- Digimon can digivolve/devolve mid-combat in some games
- Stats carry between evolutions (a well-trained Rookie becomes a stronger Champion)
- Support Skills stack from previous forms
- Party size is typically 3 active Digimon

### Keeping Turn-Based Combat Engaging

Key design principles from successful turn-based roguelikes:

1. **Meaningful choices every turn** — Never make the optimal action obvious. "Do I heal or attack? Do I use my limited skill now or save it?"
2. **Positional tactics** — Grid position matters. AOE attacks, line-of-sight, chokepoints.
3. **Risk/reward trade-offs** — Digivolution could be powerful but drains a resource. Higher floors = better loot but harder enemies.
4. **Status effects that matter** — Poison, confusion, paralysis should change your strategy, not just reduce numbers.
5. **Enemy variety** — Each enemy should require different tactics. Don't make 20 enemies that all just walk toward you and attack.
6. **Resource pressure** — Limited healing, limited skill uses. SP management creates tension.

---

## 4. Digivolution as Progression

### How Digivolution Works in Games

Digimon use a six-stage evolution system (English dub terms):

```
Fresh → In-Training → Rookie → Champion → Ultimate → Mega
(Baby I → Baby II → Child → Adult → Perfect → Ultimate in Japanese)
```

**Key characteristics from source material (Wikimon):**
- **Branching paths** — One Digimon can evolve into MANY different forms. For example, Agumon can become Greymon, Tyrannomon, GeoGreymon, or others at Champion stage.
- **Requirements vary** — Stats, experience, items, conditions, environment, and even "care mistakes" can determine evolution path.
- **Devolution is possible** — Digimon can revert to lower forms (common in partner/tamer systems).
- **Not linear** — The same species at one stage can lead to wildly different forms at the next stage.

### Digivolution in a Roguelike Context

Two complementary systems:

#### Per-Run Digivolution (Primary System)
Each dungeon run, your Digimon starts at Rookie stage. As you fight and explore:
- **Gain evolution energy** from combat, items, and floor completion
- **Choose evolution paths** when thresholds are met (present 2-3 options based on current stats/conditions)
- Evolution is **temporary to the run** — when you die, you start back at Rookie
- **Reaching Mega is the late-game power spike** — should require smart play and resource management
- **Branching creates replayability** — different paths = different builds = different strategies

**Design mechanic:** Instead of gaining traditional roguelike "levels," your Digimon digivolves. Each evolution is a significant power jump but also changes your available moveset and type advantages.

#### Meta-Progression (Between Runs)
To prevent pure frustration from permadeath:
- **Unlock new starting Rookies** by discovering them in runs
- **Unlock evolution paths** by seeing them (registered in a "DigiDex")
- **Permanent upgrades** to your hub/base (increased starting items, better shops)
- **Discovered data fragments** that slightly boost starting stats
- Keep meta-progression light — runs should stand on their own

#### Evolution Requirements For In-Run
| Requirement Type | Example | Design Purpose |
|-----------------|---------|----------------|
| **Stats threshold** | ATK > 20 to evolve into Greymon | Rewards build focus |
| **Floor reached** | Must be on floor 5+ for Champion | Paces progression |
| **Item consumed** | Use "Fire Data" item to get fire-type evolution | Adds resource decisions |
| **Combat condition** | Win 3 battles without taking damage → Angemon | Rewards skilled play |
| **Environmental** | Evolve on a water tile → aquatic form | Encourages exploration |

### Balancing Digivolution

**Problem:** Higher forms are naturally stronger. How to prevent "rush to Mega and steamroll"?

**Solutions:**
1. **SP cost increases** — Mega forms use more SP per skill. Sustainability becomes an issue.
2. **Floor-gating** — Can only reach higher stages on deeper floors (can't become Mega on floor 2).
3. **Trade-offs per form** — Mega forms might have less defense, or limited move slots, or no healing moves.
4. **Devolution mechanics** — Taking too much damage could force devolution. Creates tension.
5. **Branching rather than linear power** — Some Champion forms are better for certain dungeon types than any Ultimate form.
6. **Hunger/energy system** — Higher forms consume food/energy faster.

---

## 5. Learning Path for a Beginner

### Recommended Learning Order

#### Phase 1: Foundations (Weeks 1-3)
1. **Play roguelikes** — Download DCSS (free), play for 10+ hours to understand the genre feel. Try Pokémon Mystery Dungeon if you haven't.
2. **Install Godot 4 (.NET version)** — Follow the official "Your First 2D Game" tutorial at https://docs.godotengine.org/en/stable/getting_started/first_2d_game/index.html
3. **Learn Godot basics** — Scene tree, nodes, signals, TileMap, C# scripting. The official docs have C# examples for everything.
4. **Build "Pong" or "Breakout"** — Classic first project. Teaches input, collision, game loop.

#### Phase 2: Roguelike Fundamentals (Weeks 4-8)
5. **Follow a roguelike tutorial** — The Python+libtcod tutorial at https://rogueliketutorials.com/ (13 parts) or port its concepts to Godot C#
6. **Build a minimal roguelike in Godot** — Grid movement, random rooms, FOV, one enemy, bump-to-attack combat
7. **Add procedural dungeon generation** — BSP (Binary Space Partitioning) or room-and-corridor algorithms
8. **Implement turn-based system** — Player moves, then enemies move

#### Phase 3: Core Game Systems (Weeks 9-16)
9. **Add multiple enemy types** with different AI behaviors
10. **Implement stat system** — HP, ATK, DEF, SPD, type system
11. **Add items and inventory** — Potions, data chips, equipment
12. **Build the evolution system** — Start simple: Rookie evolves to one of 2-3 Champions based on a stat check
13. **Add multiple dungeon floors** with increasing difficulty

#### Phase 4: Polish and Expand (Weeks 17+)
14. **Expand the evolution tree** — More stages, more branches
15. **Add visual feedback** — Evolution animations, damage numbers, screen shake
16. **Meta-progression** — DigiDex, unlocks, hub area
17. **Playtest obsessively** — Balance, fun factor, pacing

### Baby Steps: First Project Before the Full Game

**Project: "Dungeon Walker"** (1-2 weeks)
- A 20x20 grid with walls
- An @ that moves with arrow keys
- 3 randomly placed enemies that chase you
- Bump to attack, 3 HP each
- When all enemies dead, you win
- When you die, game over screen

This teaches: grid movement, turn system, basic AI, collision, win/lose states. ALL of which you need for the full game.

### Common Beginner Mistakes

1. **Scope creep** — "I'll add 200 Digimon!" No. Start with 5. Add more later.
2. **Engine shopping** — Pick Godot and commit. Don't spend weeks comparing engines.
3. **Coding before designing** — Sketch your evolution tree on paper first.
4. **Perfect art early** — Use colored rectangles. Ugly prototypes are good prototypes.
5. **No playtesting** — Play your game every day. "Is this fun?" is the only question that matters.
6. **Big bang development** — Don't try to build everything then test. Build one system, test it, then build the next.
7. **Tutorial hell** — Watch one tutorial, then start coding. Google problems as they arise.
8. **Ignoring save/load** — Implement basic save/load early. It gets much harder to add later.

### Realistic Timeline

| Milestone | Estimated Time |
|-----------|---------------|
| Learn Godot basics | 2-3 weeks |
| Basic roguelike prototype (move, fight, die) | 3-5 weeks |
| Core systems (items, multiple enemies, floors) | 4-6 weeks |
| Digivolution system v1 | 3-4 weeks |
| Playable alpha (10+ Digimon, 5+ floors) | 2-3 months from start |
| Feature-complete beta | 4-6 months from start |
| Polished, releasable | 8-12 months from start |

These are rough estimates assuming ~10-15 hours/week of work. Could be faster or slower depending on pace.

---

## 6. Technical Architecture

### ECS vs OOP for Roguelikes

| Approach | Pros | Cons | Verdict for Joe |
|----------|------|------|-----------------|
| **OOP (Object-Oriented)** | Intuitive, maps to Godot's node system, Joe knows this | Can get messy with deep inheritance | **Use this** |
| **ECS (Entity Component System)** | Flexible composition, great for large games | Overkill for a beginner project, not native to Godot | Skip for now |

**Recommendation:** Use Godot's scene/node system, which is naturally OOP + composition. A Digimon is a scene with child nodes for Sprite, Stats, AI, etc. Godot's node composition IS a form of component architecture without the formal ECS overhead.

### Tile-Based Map Representation

```csharp
// Simplified map representation
public enum TileType { Wall, Floor, Door, StairsDown, Water }

public class DungeonMap {
    public TileType[,] Tiles;   // 2D array for tile types
    public bool[,] Explored;     // Has player seen this tile?
    public bool[,] Visible;      // Is this tile currently in FOV?
    public int Width, Height;
}
```

In Godot, use the **TileMap** node which provides this natively with layers, collision, and rendering built in.

### FOV and Pathfinding

**Field of View (FOV):**
- Recommended algorithm: **Recursive Shadow Casting** — Fast, produces good-looking results
- Godot has no built-in FOV for tile grids, but implementations are ~100 lines of C#
- Alternative: RayCasting from player position

**Pathfinding:**
- **A\* (A-Star)** — Best for single-target paths (monster chasing player). Godot has `AStarGrid2D` built in!
- **Dijkstra Maps** — Incredible for roguelikes. Pre-compute influence maps for flee behavior, multi-target pathing, autoexplore. RogueBasin's "The Incredible Power of Dijkstra Maps" is essential reading.
- For simple chase AI: Check if target x > current x → move right. Works for basic enemies, use A* for smarter ones.

### State Machines

Essential for managing:

1. **Game States:** MainMenu → Playing → Paused → GameOver → Victory
2. **Turn States:** PlayerTurn → ProcessingEnemies → Animating → PlayerTurn
3. **Monster AI States:** Sleeping → Wandering → Chasing → Attacking → Fleeing
4. **Evolution States:** Normal → EvolutionAvailable → Evolving → Evolved

Implement as simple enums + switch statements. Don't over-engineer with a formal state machine library.

### Digimon Data Architecture

```csharp
// Evolution tree data (load from JSON)
public class DigimonSpecies {
    public string Id;           // "agumon"
    public string Name;         // "Agumon"
    public EvolutionStage Stage;  // Rookie
    public string Type;         // "Vaccine"
    public string Element;      // "Fire"
    public BaseStats Stats;     // HP, ATK, DEF, INT, SPD base values
    public List<string> MovePool;  // Available moves
    public List<EvolutionPath> EvolvesTo;  // What it can become
}

public class EvolutionPath {
    public string TargetSpeciesId;  // "greymon"
    public EvolutionRequirement Requirements;  // Stats, items, conditions
}

public class EvolutionRequirement {
    public int MinATK;
    public int MinDEF;
    public int MinFloor;
    public string RequiredItem;  // null if no item needed
    public string SpecialCondition;  // "no_damage_3_fights", etc.
}
```

Store all species data in **JSON files**. Godot can load JSON natively. This keeps data separate from code and makes adding new Digimon trivial.

### Save/Load System

For roguelikes, save state must include:
- Current floor map (tiles, items on ground, enemy positions)
- Player state (Digimon form, HP, items, evolution progress)
- Enemy states (HP, position, AI state)
- Run metadata (floor number, score, time)
- Meta-progression (DigiDex, unlocks)

Godot approach: Serialize game state to JSON or use Godot's `ResourceSaver`. Save to `user://` directory. Delete save on death (permadeath!). Keep meta-progression in a separate file that persists.

---

## 7. Art & Assets Strategy

### Legal Considerations (CRITICAL)

**You CANNOT use "Digimon" or any Bandai Namco IP** in your game without a license. This includes:
- Digimon names (Agumon, Greymon, etc.)
- Digimon character designs
- The word "Digimon" in your game title
- Music or sound effects from Digimon media

**What you CAN do:**
- Create "Digital Creatures" or "Data Monsters" with original designs inspired by the concept
- Use a similar evolution system (evolution mechanics are game mechanics, not copyrightable)
- Reference common tropes (fire dinosaur creature, angel creature, etc.) as long as designs are original
- The type system (Virus/Vaccine/Data) cannot be trademarked as game mechanics

**Precedent:** Many successful fan-inspired games exist (Temtem inspired by Pokémon, Coromon, etc.) — they all use original creature designs.

**Recommendation:** Design original creatures with a "digital/cyber" aesthetic. Use the digivolution *system* but not the IP. Name your system something like "DataShift" or "ByteEvolve."

### Free/Low-Cost Art Options

**For Prototyping (Phase 1-2):**
- **Colored rectangles** — Seriously. Red = enemy, blue = player, green = item. This is how most roguelikes start.
- **ASCII art** — Classic roguelike look. Free by definition.
- **Kenney.nl** — Free game assets (https://kenney.nl/assets). Has roguelike tilesets.
- **OpenGameArt.org** — CC-licensed sprites and tilesets

**For Development (Phase 3+):**
- **itch.io asset packs** — Search "roguelike tileset" or "pixel RPG monsters." Many free or $1-5.
- **16x16 or 32x32 pixel art** — This is the standard for roguelikes. Small sprites = easier to create.

**For Creating Your Own:**
| Tool | Cost | Best For |
|------|------|----------|
| **Aseprite** | $20 (or free if compiled from source) | The standard pixel art tool. Animation support, tileset mode. |
| **LibreSprite** | Free | Open-source fork of Aseprite. Nearly identical features. |
| **Piskel** | Free (web-based) | Quick pixel art, runs in browser, good for beginners |
| **Pixelorama** | Free | Made in Godot! Integrates well with Godot workflow |
| **GIMP** | Free | General purpose, works but not purpose-built for pixel art |

### Placeholder Art Strategy

1. **Phase 1:** Use colored squares. Player = blue, enemies = red shades (darker = tankier), items = yellow/green.
2. **Phase 2:** Switch to simple pixel sprites. Even 8x8 or 16x16 blobs with distinguishing features (horns, wings, etc.)
3. **Phase 3:** Either create proper art, hire an artist on Fiverr/itch.io, or commission a sprite sheet.
4. **Phase 4:** Polish — animations, particle effects, evolution VFX.

The game MUST be fun with placeholder art. If it isn't, more art won't fix it.

---

## Sources

1. **RogueBasin** — Berlin Interpretation: https://www.roguebasin.com/index.php/Berlin_Interpretation
2. **RogueBasin** — How to Write a Roguelike in 15 Steps: https://www.roguebasin.com/index.php/How_to_Write_a_Roguelike_in_15_Steps
3. **RogueBasin** — Roguelike Dev FAQ: https://www.roguebasin.com/index.php/Roguelike_Dev_FAQ
4. **RogueBasin** — Articles index (design, implementation, AI, maps): https://www.roguebasin.com/index.php/Articles
5. **Godot Engine** — Official documentation: https://docs.godotengine.org/en/stable/
6. **Godot Engine** — Feature list: https://docs.godotengine.org/en/stable/about/list_of_features.html
7. **Godot Engine** — C#/.NET documentation: https://docs.godotengine.org/en/stable/tutorials/scripting/c_sharp/index.html
8. **Wikimon** — Evolution (comprehensive): https://wikimon.net/Evolution
9. **Roguelike Tutorials** — Complete Python+TCOD tutorial: https://rogueliketutorials.com/
10. **RogueBasin** — Complete Python+libtcod tutorial: https://www.roguebasin.com/index.php/Complete_Roguelike_Tutorial,_using_python3%2Blibtcod
11. **RogueBasin** — The Incredible Power of Dijkstra Maps: https://www.roguebasin.com/index.php/The_Incredible_Power_of_Dijkstra_Maps
12. **Kenney** — Free game assets: https://kenney.nl/assets
13. **OpenGameArt** — CC-licensed game art: https://opengameart.org/

---

## Brain Recommendation

If Joe proceeds with this project, I recommend creating or updating the following instruction files:

1. **New:** `.github/instructions/digimon-roguelike.instructions.md` — Project-specific conventions, Godot/C# patterns for this game, evolution data format, entity architecture
2. **New:** `Plans/010-plan-digimon-roguelike.md` — Structured plan following Plan-Generate-Verify-Iterate lifecycle with phased development milestones
3. **Optional:** `.github/instructions/godot.instructions.md` — Godot-specific coding standards (scene naming, signal patterns, C# style in Godot)
