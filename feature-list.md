# REACT-ACE-2 MEGA — Complete Feature Handbook

**Version**: 0.36.0
**Engine**: React Three Fiber + Tauri v2 Desktop
**Platform**: Windows (x64 MSI/NSIS distribution)

---

## Table of Contents

1. [World Generation](#1-world-generation)
2. [Rendering & Visual Modes](#2-rendering--visual-modes)
3. [Player Controls & Movement](#3-player-controls--movement)
4. [Weapons & Combat Systems](#4-weapons--combat-systems)
5. [Enemy Types & AI](#5-enemy-types--ai)
6. [Nemesis System](#6-nemesis-system)
7. [Upgrade & Progression Systems](#7-upgrade--progression-systems)
8. [Quest & Mission System](#8-quest--mission-system)
9. [Game Modes](#9-game-modes)
10. [Audio Systems](#10-audio-systems)
11. [UI Screens & Menus](#11-ui-screens--menus)
12. [Persistence & Save System](#12-persistence--save-system)
13. [Settings & Configuration](#13-settings--configuration)
14. [Debug & Development Tools](#14-debug--development-tools)
15. [ZeroFamily System Reference](#15-zerofamily-system-reference)

---

## 1. World Generation

### Koch Megastructure Interior

The game takes place inside an infinite procedural megastructure generated in real-time from a single world seed (`0x4B4F4348`). Every cell, wall, bridge, window, spire, and decorative element is derived deterministically from world-space coordinates — no stored maps, no pre-built levels. The same position always produces the same geometry on any machine.

**ZeroSystem**: ZeroFamily (ZeroBytes + Zero-Field + Zero-Graph + Zero-Quad)

| Parameter | Value |
|-----------|-------|
| Cell Size | 512 world units |
| Gap Between Cells | 96 units |
| Stride (Cell + Gap) | 608 units |
| Draw Distance | 3 cells (~1,824 units) |
| Max Cached Cells | 160 (LRU eviction) |
| LOD Levels | 3 (LOD 0: far, LOD 1: medium, LOD 2: full detail) |

### Biome System (5 Types)

Each region of the megastructure is classified into one of 5 architectural biomes via dual coherent noise fields. Biomes control cell height, window density, spire count, bridge frequency, and Koch displacement intensity.

| Biome | Character | Height | Koch Mult | Windows | Spires |
|-------|-----------|--------|-----------|---------|--------|
| INDUSTRIAL | Dense mechanical, factory panels | 0.7-1.0x | 1.0 | 6x3 grid | None |
| CATHEDRAL | Massive stone arches, vaulted | 1.5-2.5x | 0.6 | 4x2 grid | Up to 3 |
| CRYSTAL | Faceted, sharp geometric | 0.9-1.2x | 0.3 | None | Up to 5 |
| COLLAPSED | Ruined, debris-filled | 0.5-1.0x | 2.5 | None | None |
| TRANSIT | Dense corridors, piping | 0.4-0.6x | 0.8 | 8x4 grid | None |

### Foam Bubble Topology

Solid/void space is determined by a two-field system: a macro void field carves large caverns (cells with field value > 0.55 are always empty), while a meso field fills shell zones at ~50% density. This creates natural cavern networks with thin structural shells.

### ZeroStructureMask (Void Pockets)

Points of Interest (POI) are placed by carving rectangular void pockets from the megastructure. Each registered site suppresses Koch mesh generation within a configurable cell radius, creating clean flyable volumes for stations and objectives. Collision is also suppressed inside mask zones.

**ZeroSystem**: ZeroBytes (O(1) mask query per cell)

### POI Stations (GLTF Models)

Three GLTF station models render at POI anchor locations at 40x scale, replacing the placeholder diamond octahedra:
- **Node Station** (`poi-node-01.gltf`) — friendly/neutral hub
- **Fortress** (`poi-fortress-01.gltf`) — mini-boss headquarters
- **Stronghold** (`poi-stronghold-01.gltf`) — enemy wing base

---

## 2. Rendering & Visual Modes

### Shading Modes (3 Options)

Selectable in Settings > Graphics > SHADING MODE.

| Mode | Description | Bands | Outline | Extras |
|------|-------------|-------|---------|--------|
| **Default** | Smooth brutalist three-point lighting | Continuous | None | Ambient darkening, edge brightening |
| **Toon** | Three-band cel shading | 3 (0.12/0.55/0.95) | Fresnel silhouette | Same fog system |
| **Anime** | Five-band cel with specular + rim | 5 (0.05-1.0) | Thick Fresnel | Cyan rim glow, specular hotspots |

### ZeroBump Procedural Textures

Toggle in Settings > Graphics > ZEROBUMP TEXTURES (default OFF).

Per-pixel normal perturbation using GLSL Koch displacement noise. Adds visible panel seams, rivet patterns, and fractal roughness without modifying geometry. Uses world-axis-aligned gradient sampling for seamless edges across face boundaries. Combines with all 3 shading modes (6 shader variants total).

**ZeroSystem**: ZeroBump (shader-only, derived from ZeroFamily Koch patterns)

### SVGA Vertex Shading

Toggle in Settings > Graphics > SVGA VERTEX SHADING (default OFF).

Retro solid-fill rendering on enemy ships with 4-band intensity quantization. Adapts to active shading mode (3/5 bands for toon/anime).

### ZB Explosion Engine

Multi-layer explosion visuals replacing simple wireframe spheres. Deterministic — same position always produces the same explosion colours and debris arcs.

| Enemy Type | Class | Fireball | Shockwave | Debris | Duration |
|-----------|-------|----------|-----------|--------|----------|
| FIGHTER/DRONE | SMALL | 15 units | None | None | 0.6s |
| BOMBER | LARGE | 40 units | 60 units | 3 pieces | 1.0s |
| ACE/FRIGATE | BOSS | 60 units | 90 units | 3 pieces | 1.4s |

Layers: flash point light (0.15s) → core fireball (white/yellow) → fire shell (orange/red) → shockwave torus (LARGE/BOSS) → tumbling debris boxes with gravity (LARGE/BOSS).

**ZeroSystem**: ZeroBytes (posHash → per-debris arc properties)

### Ship Models (GLTF)

| Ship | Model | Scale | Fallback |
|------|-------|-------|----------|
| Fleet Carrier | `fleet-carrier-001.gltf` | 20x | Wireframe hexagon hull |
| Goon Frigate | `goon-frigate-001.gltf` | 20x | Blue wireframe jet |

---

## 3. Player Controls & Movement

### Control Scheme

| Action | Input | Notes |
|--------|-------|-------|
| Aim / Look | Mouse (pointer lock) | Pitch + Yaw, invertible in Settings |
| Forward Throttle | W (hold) | Ramps to 1.0 at 2.0/s acceleration |
| Reverse Throttle | S (hold) | Ramps to -1.0, mecha mode only |
| Strafe Left/Right | A / D | Mecha mode only, instant direction change |
| Vertical Up/Down | Q / E | Both modes |
| Afterburner | Tab (hold) | 4.0x throttle, triggers flight mode |
| Zero Throttle | X | Instant stop |
| Match Target Speed | F | Matches locked target velocity |
| Dodge | Shift + W/A/S/D | 1-second directional burst with roll animation |
| Fire Missiles | LMB | Lock-on guided swarm missiles |
| Fire Special (QWE) | RMB | Quasar beam (requires QWE upgrade) |
| Deploy Funnels | 1 | 6 combat drones (requires Drones upgrade) |
| Recall Funnels | 2 | Returns drones to player |
| Lock Target | R | Cycles nearest enemy |
| Pause | ESC | Opens pause overlay |
| Autopilot | Tab (double-tap) | Full afterburner, cancelled by any input |
| Camera Zoom | Scroll Wheel | 1.0x – 3.0x chase distance |

### Mecha ↔ Flight Mode Transform

The player ship smoothly morphs between mecha (humanoid) and flight (jet) geometry based on forward speed. Transform takes 0.25 seconds. Uses clamped throttle ratio (0-1) so the threshold scales proportionally with engine upgrades.

| Condition | Result |
|-----------|--------|
| Forward throttle > 0.5 | Flight mode activates |
| Strafing (A/D held) | Forces mecha mode |
| Reverse throttle | Forces mecha mode |
| Afterburner (Tab) | Flight mode (throttle clamped to 1.0 for check) |

### Collision System

Player and entities collide with solid megastructure cells via AABB resolution. The system checks a 3x3x3 neighbourhood and pushes entities out along the shallowest penetration axis ("slide along walls"). ZeroStructureMask void pockets are excluded from collision — players fly freely through POI areas.

---

## 4. Weapons & Combat Systems

### Primary Weapon — Swarm Missiles (LMB)

Lock-on guided missiles that seek enemies within a 45-degree forward cone. Auto-target the nearest valid enemy. Cover system applies damage modifiers based on line-of-sight through megastructure cells.

| Stat | Base | Per Weapon Level | Per Tree Bonus |
|------|------|-----------------|----------------|
| Damage | 100 | +10% | +damage% |
| Reload Time | 0.5s | -5% (min 0.1s) | -reload% |
| Range | 600 units | — | — |
| Missile Speed | 800 u/s | — | — |
| Ammo | 40 per mission | — | — |

### Missile Engineering (Special Upgrade)

| Level | Speed Mult | AOE Radius | AOE Damage |
|-------|-----------|-----------|-----------|
| 0 | 1.0x | None | None |
| 1 | 1.5x | 100 units | 250 |
| 2 | 2.0x | 200 units | 500 |

### PDS — Point Defense System (Auto)

Automated turret that intercepts nearby enemies. Fires every 0.05-0.1 seconds, dealing 20 damage per shot.

| Level | Range | Fire Rate |
|-------|-------|-----------|
| 0 | Inactive | — |
| 1 | 150 units | Every 0.1s |
| 2 | 300 units | Every 0.05s |

### Dual Drones — Wingmen (Auto)

Two escort drones flank the player and fire 50-damage slugs at the nearest 2 enemies within range.

| Level | Range | Fire Rate |
|-------|-------|-----------|
| 0 | Inactive | — |
| 1 | 150 units | Every 0.5s |
| 2 | 300 units | Every 0.25s |

### QWE — Quasar Wave Emitter (RMB)

Long-range piercing energy beam. Ignores all megastructure geometry (no line-of-sight check). Deals 9999 damage (instant kill). 20-second cooldown, 0.2-second active duration.

| Level | Range | Radius | Chain Lightning |
|-------|-------|--------|----------------|
| 0 | Inactive | — | — |
| 1 | 1,000 units | 150 units | No |
| 2 | 2,000 units | 300 units | Yes (3 targets, 800u range) |

### Funnels — Combat Drones (Keys 1/2)

6 autonomous micro-drones that seek enemies, orbit at 80-unit radius, and fire magenta beams.

| Phase | Duration | Behaviour |
|-------|----------|-----------|
| Idle | — | Ready to deploy |
| Active | 2 minutes | Orbit + fire at 0.8s intervals, 18 base damage |
| Cooldown | 30 seconds | Returning to player, recharging |

Transit speed: 440 u/s. Fire range: 160 units. Visible on radar as magenta dots.

---

## 5. Enemy Types & AI

| Type | Health | Speed | Score | Behaviour | Mesh |
|------|--------|-------|-------|-----------|------|
| FIGHTER | 100 | Fast | 500 | Approach → Attack → Retreat cycle, cover-seeking | Red wireframe jet |
| DRONE | 40 | Medium | 500 | Swarm behaviour (level 3+) | Red wireframe jet (small) |
| BOMBER | 200 | Slow | 1,000 | Targets carrier, larger hitbox | Orange wireframe jet |
| ACE | Variable | Fast | 2,000 | Nemesis pilot with escorts | Purple wireframe jet |
| FRIGATE | 500 | Very slow | — | Capital ship, anti-missile PDS, retreats when close | GLTF model (20x scale) |

### Cover-Based AI (ZeroCover)

Enemies use 3D Bresenham cell traversal + ZeroCubic triple hashing to compute line-of-sight through megastructure cells. When under fire, they relocate to covered positions. Re-check interval: 2.0 seconds.

**ZeroSystem**: ZeroCover (Zero-Cubic triple hash for emergent cover properties)

---

## 6. Nemesis System

Defeated enemies have a chance to spawn nemesis seeds — persistent rival pilots that return in future missions with escalating power and escort squads.

| Tier | Title | Health Mult | Damage Mult | Escort |
|------|-------|-------------|-------------|--------|
| 1 | ROOKIE | 1.5x | 1.2x | 1 drone |
| 2 | VETERAN | 2.0x | 1.5x | 2 fighters |
| 3 | ACE | 3.0x | 2.0x | 2 fighters + 1 bomber |
| 4 | ELITE ACE | 4.0x | 2.5x | 3 fighters + 1 bomber |
| 5 | NEMESIS | 5.0x | 3.0x | Full wing |

- Max 4 active nemeses at once
- On player death, the highest-tier nemesis is promoted
- Callsigns, dialogue, and properties derived deterministically from seed via ZeroBytes
- HUD shows nemesis alert with callsign + rank when within 600 units

**ZeroSystem**: ZeroBytes (seed → callsign, tier, escorts, dialogue)

---

## 7. Upgrade & Progression Systems

### Hangar Shop (Score Currency)

Core upgrades (max level 5, cost = baseCost x level):

| System | Base Cost | Effect Per Level |
|--------|-----------|-----------------|
| Turbine Engine | 1,000 | +10% speed and afterburner |
| Nano-Composite (Hull) | 1,500 | +10% health and handling |
| Missile Systems (Weapon) | 2,000 | +10% damage, -5% reload |
| Avionics Suite (Radar) | 1,200 | +10% radar range |

Special systems (max level 2, tiered costs 25K/50K):

| System | Level 1 | Level 2 |
|--------|---------|---------|
| PDS Defense | Auto-turret (150m, 0.1s) | 2x range + fire rate |
| Dual Drones | Wingmen deployed | 2x range + fire rate |
| Quasar Wave | Beam (1000m range) | Chain lightning + 2x range |
| Missile Eng. | +50% speed, AOE blast | +100% speed, massive AOE |

### ZeroTree Skill Progression

Procedurally generated 9x7 skill tree accessed from the Hangar. 63 nodes with deterministic types, stats, costs, and connections derived from a fixed tree seed (`0x5B455254`).

**Node Types**: STAT (most common), PASSIVE, ACTIVE, KEYSTONE (tier 5+ only)

**6 Stat Bonuses** (multiplicative with upgrade levels):

| Stat | Effect | Magnitude Range |
|------|--------|----------------|
| Speed | +N% maxSpeed / afterburner | 1-8% per node |
| Damage | +N% weapon damage | 1-10% per node |
| Health | +N% max health | 2-12% per node |
| Handling | +N% turn rate | 1-6% per node |
| Reload | -N% reload time | 1-5% per node |
| Armor | Flat damage reduction | 1-8 per node |

**Skill Points**: Earned on mission completion (3 base + 1 per level number). Spent to unlock tree nodes. Respec available with 30% tax.

**Features**: Zoom (scroll), pan (drag), click to select, double-click to unlock, colour-coded stats, pulse animation on available nodes, checkmarks on unlocked.

**ZeroSystem**: ZeroBytes (FNV-1a hash → node properties, edge connections)

---

## 8. Quest & Mission System

### ZeroQuest Procedural Missions

Each mission's type, narrative, timer, stages, and objectives are derived deterministically from world coordinates.

**ZeroSystem**: ZeroQuest (ZeroBytes seed → quest type, narrative, timer, stages)

| Quest Type | Timer | Stages | Win Condition |
|-----------|-------|--------|---------------|
| PROTECT_HOME | 60-180s | 1 | Kill all enemies + extract at carrier |
| PROTECT_POI | 90-240s | 1 | Kill all enemies + extract at carrier |
| DATA_DUMP | 30-60s/stage | 3 | Complete all stages + extract |
| STRONGHOLD | None | 1 | Kill all enemies (no extraction needed) |

### Enemy Spawning

Enemies spawn at 300ms intervals around quest objective coordinates (or carrier for PROTECT_HOME). Round-robin across multiple spawn centres for multi-point quests. Base count: `debugEnemyBaseCount + (level x 4)`. One FRIGATE per level. Nemesis ACEs with tier-scaled escorts.

### Mission Flow

```
MENU → startLevel() → LOADING (voice pool prewarm)
→ QUEST_BRIEFING (narrative + accept)
→ PLAYING (combat + objectives)
→ MISSION_COMPLETE (score + time bonus)
→ Next Level or MENU
```

---

## 9. Game Modes

### Campaign (New Game / Continue)

Progressive missions with escalating difficulty. Score persists as currency for upgrades. Nemesis rivals carry between levels. Save data preserved in IndexedDB.

### Tutorial (Free Play)

Sandbox mode accessed from main menu. Player spawns at carrier with max upgrades (all core level 5, all special level 2), 99 missiles, no mission objective, no timer, no initial enemies.

Enemies spawn dynamically as the player explores: every 3rd new megastructure cell visited spawns 2-4 enemies at that location with a POI void pocket registered. No win or lose condition — the player can explore and practice indefinitely.

HUD shows amber "FREE PLAY / EXPLORE & PRACTICE" instead of the mission timer.

---

## 10. Audio Systems

### Three Independent Audio Layers

| Layer | Technology | Purpose |
|-------|-----------|---------|
| AudioSystem | Native Web Audio synthesis | Engine hum (30-80 Hz sawtooth), shoot/explosion/lock SFX |
| ZeroSounds | Procedural hash-based synthesis | 49 named UI/game sounds, fully deterministic from seed |
| MusicManager | Howler.js file playback | Background music with crossfading, playlists, context switching |

### ZeroSounds (Procedural SFX)

All sound parameters derived O(1) from `(soundName, worldSeed)` via hash chain. No `Math.random()` — same seed always produces identical sounds. Supports tonal, noisy, and arpeggio synthesis with ADSR envelopes, filters, echo, and bit-crush effects.

**ZeroSystem**: ZeroBytes (hash → waveform, frequency, envelope, filter, effects)

### Music Manager

Adaptive background music with context-aware playlist switching. Crossfades between tracks over 2 seconds when the game state changes.

| Game Status | Music Context |
|-------------|--------------|
| Menu / Hangar / Settings / Pause | `menu` |
| Playing (exploration) | `exploration` |
| Mission Complete / Victory | `victory` |
| Game Over | `menu` |

### Music Stations (3 Soundtracks)

Selectable in Settings > Audio > MUSIC STATION.

| Station | Tracks | Character |
|---------|--------|-----------|
| **Platano** (default) | 9 tracks | Latin-flavoured electronic |
| **Failure** | 13 tracks | Eastern-European cyberpunk |
| **KAN** | 10 tracks | Minimal industrial |

Each station maps tracks to all 5 context types (menu, exploration, combat, boss, victory).

### ZeroSpeech (Voice Synthesis)

Enemy dialogue generated via KokoroTTS ONNX inference (Tauri Rust backend) or fallback oscillator synthesizer. 32 voice lines pre-baked during loading screen into an in-memory PCM cache. Claims during gameplay are fully synchronous — zero awaits, zero combat freezing.

Ring modulation post-processing via OfflineAudioContext applies "Classic Dalek" radio effect to all goon voices.

**ZeroSystem**: ZeroVoice (xxHash64 position → voice derivation)

---

## 11. UI Screens & Menus

| Screen | Access | Purpose |
|--------|--------|---------|
| **Main Menu** | App start | New Game, Continue, Hangar, Tutorial, Settings |
| **Hangar** | Menu (with save) | Purchase upgrades, access Skill Tree |
| **Skill Tree** | Hangar → SKILL TREE | ZeroTree node graph with zoom/pan |
| **Settings** | Menu → Settings | Audio, Graphics, Controls |
| **Loading Screen** | Auto (after New Game) | Voice pool prewarm progress bar |
| **Quest Briefing** | Auto (after loading) | Mission narrative, objective, timer |
| **In-Game HUD** | During gameplay | Speed, altitude, missiles, hull, score, radar, quest, nemesis alerts, funnel status, megastructure position/biome |
| **Pause Menu** | ESC during gameplay | Resume, Abandon Mission (with nemesis warning) |
| **Mission Complete** | Win condition met | Score summary, time bonus, next level |
| **Game Over** | Player death | Retry or main menu |
| **Poly-Forge Editor** | Debug menu | Low-poly ship designer tool |
| **ZeroVoice Debug** | Debug menu | Speech synthesis testing |
| **Music Manager Debug** | Debug menu | Playlist browser, context testing |

### Elite Radar 3D

Holographic dish-style radar overlay showing:
- Red contacts: hostile enemies
- Magenta contacts: missiles
- Magenta small dots: deployed funnels
- Blue octahedron: carrier
- Green centre: player position
- Scanline sweep animation

---

## 12. Persistence & Save System

| Data | Storage | Saved When |
|------|---------|-----------|
| Player Score | IndexedDB | Upgrade purchase, level complete |
| Upgrade Levels (8 systems) | IndexedDB | Purchase, level complete |
| Current Level | IndexedDB | Level complete |
| Nemesis Records (up to 4) | IndexedDB | Level complete, player death |
| ZeroTree Bitmask | IndexedDB | Skill tree changes |
| Skill Points | IndexedDB | Level complete |
| Game Settings | localStorage | Any setting change |

Database: `REACT_ACE_DB` v2, object store `progress`, key `player_save`. Connections closed after each transaction to prevent leaks.

---

## 13. Settings & Configuration

### Audio
- **Master Volume** — 0-100% (controls SFX synthesis)
- **Music Volume** — 0-100% (controls Howler.js playback, independent)
- **Music Station** — Platano / Failure / KAN

### Graphics
- **SVGA Vertex Shading** — Retro solid-fill on ships (default OFF)
- **ZeroBump Textures** — Procedural surface detail (default OFF)
- **Shading Mode** — Default / Toon / Anime

### Controls
- **Invert Pitch (W/S)** — Toggle
- **Invert Yaw (A/D)** — Toggle

All settings persist to localStorage across sessions.

---

## 14. Debug & Development Tools

Accessed via `[ DEBUG CONFIG ]` button (top-right of main menu).

| Tool | Description |
|------|-------------|
| Time Limit Slider | Override mission timer (1-10 minutes) |
| Base Enemy Count | Override spawn count (8-1000) |
| Small Loan (1M pts) | Checkbox: grants 1M score on mission complete |
| Small Loan (1000 skill pts) | Button: instant skill point grant |
| Sound Seed | Hex slider for deterministic sound derivation |
| Poly-Forge Editor | Full low-poly ship design tool |
| ZeroVoice Debug | Speech synthesis backend testing |
| Music Manager | Playlist browser with context switching |

---

## 15. ZeroFamily System Reference

The game is built on the **ZeroFamily** suite of deterministic generation systems. All share the same hashing foundation (ZeroBytes) and produce identical output for identical input — no randomness, no network, no database.

| System | Domain | Key Principle |
|--------|--------|--------------|
| **ZeroBytes** | Position hashing | `posHash(x,y,z,salt)` → uint32, O(1) |
| **Zero-Field** | Coherent noise | Trilinear interpolated, multi-octave, Y-stratified |
| **Zero-Graph** | Connectivity | Symmetric pair hashing for bridges and edges |
| **Zero-Quad** | Pair properties | Facade alignment, arch triggers |
| **Zero-Cubic** | Triple hashing | Fully symmetric 3-point cover properties |
| **ZeroCover** | Combat AI | Line-of-sight + relocation through cell grid |
| **ZeroQuest** | Missions | Position → quest type, narrative, stages, timer |
| **ZeroVoice** | Speech | xxHash64 → voice derivation from spawn coords |
| **ZeroSounds** | Audio | Hash → waveform, frequency, envelope, effects |
| **ZeroBump** | Textures | Shader Koch noise → per-pixel normal perturbation |
| **ZeroTree** | Skill tree | FNV-1a → node types, stats, edges, costs |
| **ZeroExplosion** | VFX | posHash → explosion class, debris arcs, colours |
| **ZeroStructureMask** | World editing | Cell mask query → void pockets for POI |

**The position IS the seed. No storage. No network. No state. Same inputs → same outputs. Always.**
