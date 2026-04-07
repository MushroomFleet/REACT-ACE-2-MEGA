<div align="center">
<img width="1200" height="475" alt="RA2-MEGA" src="https://raw.githubusercontent.com/MushroomFleet/REACT-ACE-2-MEGA/refs/heads/main/react-ace2mega.png" />
</div>

# REACT-ACE-2 MEGA
## Infinite Procedural Space Combat Inside a Koch Fractal Megastructure

A desktop space combat game set inside an infinite deterministic megastructure generated in real-time from a single seed. Built with React Three Fiber, Three.js, Zustand, and Tauri v2. Every wall, bridge, spire, biome, quest, explosion, sound effect, enemy voice, and skill tree node is derived procedurally — no stored maps, no pre-built levels, no randomness. The position IS the seed.



<div align="center">

### [📦 DOWNLOAD MSI INSTALLER](https://github.com/MushroomFleet/REACT-ACE-2-MEGA/releases) | [📖 FULL FEATURE HANDBOOK](https://scuffedepoch.com/ra2-mega)

**Windows x64** — Install via MSI for the best experience. Music files included in the installer.

</div>

---

## 🎯 Game Overview

You pilot a transforming mecha/flight ship inside the **Koch Megastructure** — an infinite procedural interior built from 5 architectural biomes (Industrial, Cathedral, Crystal, Collapsed, Transit). Defend the **Fleet Carrier** from waves of enemy fighters, bombers, frigates, and persistent nemesis rivals across escalating missions. Upgrade your ship in the **Hangar**, unlock nodes in the **ZeroTree** skill progression system, and explore a world that generates itself around you.

### Core Features

- **Infinite Procedural World** — Koch fractal megastructure with 5 biomes, foam-bubble void topology, and deterministic geometry from a single world seed
- **Mecha ↔ Flight Transform** — Ship morphs between humanoid and jet forms based on forward speed
- **13 ZeroFamily Systems** — Every game element (world, quests, sounds, voices, explosions, skill trees) derived O(1) from position hashing
- **6 Weapon Systems** — Swarm missiles, PDS auto-turret, dual drones, Quasar Wave beam, combat funnels (6 autonomous drones), missile engineering AOE
- **Nemesis System** — Defeated enemies return as escalating rivals with escorts, unique callsigns, and dialogue
- **ZeroTree Skill Progression** — 63-node procedural skill tree with 6 stat types, zoom/pan canvas, respec
- **3 Shading Modes** — Default brutalist, Toon (3-band cel), Anime (5-band + rim glow + specular)
- **ZeroBump Textures** — Shader-only procedural surface detail (panel seams, fractal roughness)
- **Adaptive Music** — 3 soundtrack stations (32 tracks) with context-aware crossfading
- **ZeroSpeech Voice Synthesis** — KokoroTTS ONNX enemy dialogue with ring modulation
- **Tutorial / Free Play** — Sandbox mode with max upgrades and discovery-based enemy spawning
- **ZeroQuest Missions** — 4 procedural quest types with deterministic narratives and objectives
- **POI Stations** — GLTF mesh structures at void-pocket locations in the megastructure

---

## 🕹️ Controls

### Movement
| Action | Input |
|--------|-------|
| Aim / Look | Mouse (pointer lock) |
| Forward / Reverse | W / S |
| Strafe Left / Right | A / D (mecha mode only) |
| Vertical Up / Down | Q / E |
| Afterburner | Tab (hold) |
| Dodge | Shift + W/A/S/D |
| Zero Throttle | X |
| Match Target Speed | F |

### Combat
| Action | Input |
|--------|-------|
| Fire Missiles | LMB |
| Fire Special (QWE Beam) | RMB |
| Lock Target | R |
| Deploy Funnels | 1 |
| Recall Funnels | 2 |
| Static Discharge | 3 |
| Pause | ESC |

---

## 🛠️ Upgrade Systems

### Hangar Shop (Score Currency)

**Core Systems** (max level 5): Engine (+10% speed), Hull (+10% health + handling), Weapon (+10% damage, -5% reload), Radar (+10% range)

**Special Systems** (max level 2): PDS Defense, Dual Drones, Quasar Wave Emitter, Missile Engineering

### ZeroTree Skill Progression

9×7 procedural skill tree with 6 stat bonuses (speed, damage, health, handling, reload, armor). Nodes unlocked with skill points earned on mission completion (3 base + 1 per level). Full respec available with 30% tax. Accessed from the Hangar menu.

---

## 🎨 Visual Modes

| Mode | Description |
|------|-------------|
| **Default** | Smooth three-point brutalist lighting with quartic fog |
| **Toon** | Three-band cel shading with Fresnel silhouette outlines |
| **Anime** | Five-band cel ramp with cyan rim glow and specular hotspots |
| **ZeroBump** | Procedural surface textures via shader normal perturbation (toggle, combines with any mode) |
| **SVGA** | Retro solid-fill vertex shading on ships (toggle) |

---

## 🎵 Soundtrack

Music by **JXHNSXNT**. Three selectable stations, each mapping tracks to 5 game contexts (menu, exploration, combat, boss, victory):

| Station | Tracks | Character |
|---------|--------|-----------|
| **Platano** | 9 | Latin-flavoured electronic |
| **Failure** | 13 | Eastern-European cyberpunk |
| **KAN** | 10 | Minimal industrial |

Music volume and station selection persist across sessions. Context-aware crossfading transitions automatically during gameplay.

---

## 🏗️ ZeroFamily Architecture

Every procedural system in the game shares the same ZeroBytes hashing foundation. All produce identical output for identical input — no `Math.random()`, no network, no database.
- REPO: [ZERO-FAMILY-SKILL](https://github.com/MushroomFleet/ZeroBytes-Family-Skills)
- 'npx skills add MushroomFleet/ZeroBytes-Family-Skills'

| System | Domain |
|--------|--------|
| ZeroBytes | Position → uint32 hash (O(1)) |
| Zero-Field | Coherent noise fields (biomes, voids, dimensions) |
| Zero-Graph | Bridge connectivity, edge density |
| Zero-Cubic | Three-point cover properties (combat AI) |
| ZeroQuest | Position → quest type, narrative, stages, timer |
| ZeroVoice | Position → voice derivation (KokoroTTS) |
| ZeroSounds | Hash → waveform, frequency, envelope, effects |
| ZeroBump | Shader Koch noise → per-pixel normal perturbation |
| ZeroTree | Hash → skill node types, stats, edges, costs |
| ZeroExplosion | Position → explosion class, debris arcs, colours |
| ZeroStructureMask | Cell query → void pocket for POI placement |

---

## 📝 Tech Stack

| Layer | Technology |
|-------|-----------|
| UI Framework | React 19 |
| 3D Engine | Three.js + @react-three/fiber |
| State Management | Zustand |
| Desktop Runtime | Tauri v2 (Rust backend) |
| Voice Synthesis | KokoroTTS ONNX (ort crate, eSpeak-NG) |
| Music Playback | Howler.js |
| Procedural Audio | Web Audio API (ZBEngine) |
| Build Tool | Vite |
| Language | TypeScript |

---

## 💿 Installation

### Recommended: MSI Installer (Windows x64)

Download the latest `.msi` from [Releases](https://github.com/MushroomFleet/REACT-ACE-2-MEGA/releases) and run it. The installer includes all game assets, music files, and the Tauri runtime.

### Development Setup

```bash
git clone https://github.com/MushroomFleet/REACT-ACE-2-MEGA.git
cd REACT-ACE-2-MEGA
bun install
bun run tauri dev
```

Requires: Node.js 18+ or Bun, Rust toolchain, eSpeak-NG (for voice synthesis).

---

## 📚 Citation

### Academic Citation

If you use this codebase in your research or project, please cite:

```bibtex
@software{react_ace_2_mega,
  title = {REACT-ACE-2 MEGA: Infinite Procedural Space Combat Inside a Koch Fractal Megastructure},
  author = {Drift Johnson},
  year = {2025},
  url = {https://github.com/MushroomFleet/REACT-ACE-2-MEGA},
  version = {0.36.0}
}
```

### Donate:

[![Ko-Fi](https://cdn.ko-fi.com/cdn/kofi3.png?v=3)](https://ko-fi.com/driftjohnson)

---

<div align="center">

**The position IS the seed. No storage. No network. No state. Same inputs → same outputs. Always.**

[🚀 DOWNLOAD & PLAY](https://github.com/MushroomFleet/REACT-ACE-2-MEGA/releases)

</div>

BUILT WITH TINS
- 'npx skills add MushroomFleet/TINS-skill'
![TINS-inside](https://raw.githubusercontent.com/MushroomFleet/TINS-directory/main/TINS-inside.png)

---
