# REACT-ACE-2 MEGA — Feature Handbook Part 2

**Covers**: v0.37.0 through v0.45.1
**Previous**: See [feature-list.md](feature-list.md) for v0.36.0 baseline
**Engine**: React Three Fiber + Tauri v2 Desktop
**Platform**: Windows (x64 MSI/NSIS distribution)

---

## Table of Contents

1. [Combat Performance & Mode Separation (v0.37.0)](#1-combat-performance--mode-separation-v0370)
2. [Toggle-Based Mecha/Flight Transform (v0.38.0)](#2-toggle-based-mechaflight-transform-v0380)
3. [Newtonian Mecha Momentum (v0.39.0)](#3-newtonian-mecha-momentum-v0390)
4. [Ship Handling Polish (v0.40.0)](#4-ship-handling-polish-v0400)
5. [Thruster & Afterburner Glow (v0.41.0)](#5-thruster--afterburner-glow-v0410)
6. [Constant-Size Enemy Indicators (v0.41.1)](#6-constant-size-enemy-indicators-v0411)
7. [Fleet Carrier Persona Speech (v0.42.0)](#7-fleet-carrier-persona-speech-v0420)
8. [QWE Beam Batching & Missile Accuracy (v0.43.0)](#8-qwe-beam-batching--missile-accuracy-v0430)
9. [Capital Ship Movement (v0.44.0)](#9-capital-ship-movement-v0440)
10. [INBOX Narrative System (v0.45.0)](#10-inbox-narrative-system-v0450)
11. [Updated ZeroFamily System Reference](#11-updated-zerofamily-system-reference)

---

## 1. Combat Performance & Mode Separation (v0.37.0)

### O(1) Enemy Lookup

Replaced O(n) `enemies.find()` with a per-frame `enemyById` Map across 7 call sites (missile targeting, PDS, drones, funnels, QWE beam, lock-on, speech triggers). Speech proximity checks capped at 20 enemies per frame.

### Synchronous Missile Spawning

Missiles now spawn synchronously in a single frame instead of staggered via `setTimeout`. Eliminates render-frame delays during burst fire.

### Weapon Mode Gating

All weapons (missiles, PDS, drones, QWE beam) are gated on mecha mode (`flightFactor <= 0.5`). Weapons are disabled in flight mode — funnels remain active as autonomous units in both modes.

### Afterburner 8x Multiplier

Afterburner speed increased from 4x to 8x max speed, providing meaningful transit speed through the megastructure.

### Funnel Visual Overhaul

Combat funnels replaced wireframe geometry with magenta glow spheres and trail particles. QWE beam and missile projectile sizes halved for visual clarity in dense combat.

---

## 2. Toggle-Based Mecha/Flight Transform (v0.38.0)

### F-Key Toggle System

Mecha/flight transformation redesigned from speed-triggered to explicit F-key toggle. Mecha is the default combat stance; flight is a transit mode toggled intentionally.

| Mode | Speed Cap | Controls | Weapons |
|------|-----------|----------|---------|
| **Mecha** (default) | 12.5% maxSpeed | Full 6-DOF: forward, reverse, strafe, vertical, dodge | All active |
| **Flight** (F toggle) | 100% maxSpeed | Forward only, no strafe/reverse/vertical | Disabled (funnels exempt) |
| **Afterburner** (Tab in flight) | 800% maxSpeed | Forward only | Disabled |

### Emergency Airbrake

Pressing F in flight mode instantly clamps velocity to the mecha speed cap, cancels throttle, and begins the mecha transform animation.

### Speed Scaling

All three speed tiers (mecha, flight, afterburner) scale proportionally with engine upgrades and ZeroTree speed bonuses.

| Example (Engine L5 + 20% Tree) | Speed |
|------|-------|
| Mecha cap | 4.2 u/s |
| Flight max | 33.6 u/s |
| Afterburner | 268.8 u/s |

### Paradistro Quest Profile

New `ZeroQuest-Paradistro.json` narrative profile added alongside the default profile. Replaces generic sci-fi quest text with Paradistro-themed language: lattice authority, shell coordinates, covenant governance, rogue machines, non-linear archives, and builder mythology.

8 quest categories with Paradistro theming: PROTECT_HOME, PROTECT_POI, DATA_DUMP, STRONGHOLD, briefing, stage_prompt, success, failure, and in_mission radio chatter.

---

## 3. Newtonian Mecha Momentum (v0.39.0)

### Decoupled Movement Model

Mecha mode upgraded from direct velocity to a Newtonian thrust model with persistent momentum. Mouse look rotates the player freely without affecting the motion vector — creating true arcade mecha drift.

| Property | Mecha Mode | Flight Mode |
|----------|-----------|-------------|
| Speed range | 0 to 100% maxSpeed | 0 to 800% maxSpeed |
| Movement model | Thrust → acceleration → velocity + dampening | Direct: throttle x maxSpeed along forward |
| Mouse look | Rotates player only (decoupled from motion) | Steers motion direction |
| No input | Auto-dampening to zero | Throttle decay |
| Drift | Yes — turn while maintaining momentum | No — always along facing |

### Mecha Speed Unlocked

Mecha throttle cap raised from 12.5% to 100% of maxSpeed. Mecha mode now reaches the same top speed as flight mode — the difference is the movement physics, not the speed ceiling.

### Thrust Physics

| Parameter | Value |
|-----------|-------|
| Thrust acceleration | maxSpeed x 4.0 u/s^2 |
| Dampening rate | 3.0 (velocity halves every ~0.23s) |
| Dead-stop threshold | 0.1 u/s |
| Speed clamp | maxSpeed (prevents multi-axis infinite acceleration) |

Directional keys (W/A/S/D/Q/E) apply thrust forces along the player's current facing direction. Releasing keys triggers auto-dampening. Strafing and vertical thrust produce lateral/vertical drift that persists after key release.

### Emergency Airbrake Updated

F-key airbrake now clamps velocity to full maxSpeed (not the old 12.5% cap), since mecha mode shares the same speed range as flight.

---

## 4. Ship Handling Polish (v0.40.0)

### Extended Dampening

Mecha auto-dampening reduced from ~0.5 seconds to ~4 seconds for a full stop from top speed. Dampening rate changed from 3.0 to 1.15 (`e^(-1.15 x 4) ≈ 0.01`). Drift now feels substantial — the player slides through space with genuine momentum.

### Afterburner Cancel on Transform

F-key now cancels both afterburner and autopilot when transitioning from flight to mecha. Prevents Tab-held afterburner from immediately re-engaging throttle after transform.

### 2-Second Braking Animation

Flight-to-mecha transform at high speed triggers a 2-second linear deceleration ramp instead of an instant velocity clamp. Speed decelerates smoothly from flight speed to mecha maxSpeed over the braking period. The player can steer during braking.

### PlayerShip 180-Degree Rotation Fix

PlayerShip model rotated 180 degrees around Y-axis (`rotation={[0, Math.PI, 0]}`) to align the nose with the Three.js `-Z forward` movement vector. Ship now faces the direction of travel.

---

## 5. Thruster & Afterburner Glow (v0.41.0)

### Thrust-Linked Engine Lighting

Orange engine sockets now brighten proportionally with forward throttle. Dark at zero thrust, full intensity at maximum throttle. The existing throb/flicker animation is preserved and multiplied with the thrust factor.

| Socket Type | Colour | Behaviour |
|------------|--------|-----------|
| Engine | Orange (#ff6600) | Intensity = throttleRatio x 3.0 x flicker. Proportional to throttle. |
| Thruster | Blue (#00ccff) | Intensity = afterburner ? 2.0 x flicker : 0. Boolean on/off. |

### Boolean Afterburner Emitters

Blue thruster sockets are completely dark unless the afterburner is actively engaged (Tab held or autopilot active). Provides clear visual feedback for afterburner state.

---

## 6. Constant-Size Enemy Indicators (v0.41.1)

### Fixed Screen-Space Indicators

Enemy target indicators changed from distance-scaled (`distanceFactor={200}`) to constant screen-space size. Indicators are now visible at all ranges including 1000+ units.

| Property | Before | After |
|----------|--------|-------|
| Size | 40px (shrinks with distance) | 24px (constant) |
| Opacity | 0.8 | 1.0 |
| Centre dot | 4px, 4px glow | 3px, 6px glow |
| Visibility at 1000+ units | Nearly invisible | Clearly visible |

Colour coding preserved: red (weapon range), yellow (out of range), purple (nemesis), orange (bomber).

### .gitattributes Cleanup

Binary asset tracking cleaned up for GLTF, audio, and image files.

---

## 7. Fleet Carrier Persona Speech (v0.42.0)

### Multi-Pool Speech Architecture

Speech pool system expanded from 32 goon-only slots to a structured multi-pool layout:

| Pool | Slots | Profile | Ring Preset | Status |
|------|-------|---------|-------------|--------|
| Goon | 10 | `goon-pilot.json` | `classicDalek` | Active (reduced from 32) |
| Persona | 10 | `fleetcarrier-profile.json` | `alienRadio` | Active |
| Narrator | 10 | (reserved) | (reserved) | Reserved |
| Menu | 2 | (reserved) | (reserved) | Reserved |

### Fleet Carrier ATC Comms

New `fleetcarrier-profile.json` provides ATC-style allied chatter from UNF ALBATROSS. 8 broadcast categories with Paradistro megastructure theming:

| Category | Content |
|----------|---------|
| `status_broadcast` | Carrier status, system checks, fleet condition |
| `navigation_advisory` | Spatial warnings, biome transitions, corridor alerts |
| `patrol_report` | Sector sweeps, recon results, threat assessment |
| `combat_alert` | Contact reports, tactical updates, weapons authorization |
| `operations_chatter` | Flight deck ops, maintenance, logistics |
| `lore_fragment` | Supreme echoes, archive transmissions, builder mythology |
| `morale_broadcast` | Fleet encouragement, recognition, solidarity |
| `return_advisory` | Landing guidance, deck status, recovery ops |

~800,000+ unique callout combinations across all categories.

### Persona Speech Timing

First persona broadcast plays 5 seconds after mission start. Subsequent broadcasts every 60 seconds during gameplay. Uses `alienRadio` ring modulation preset (distinct from goon `classicDalek`).

### Nested Profile Support

`GoonPool` constructor extended with `profileKey` and `poolName` parameters to support nested profile structures (e.g., extracting `status_broadcast` sub-object from the fleet carrier profile).

---

## 8. QWE Beam Batching & Missile Accuracy (v0.43.0)

### Batched QWE Kills

QWE beam mass-kill refactored from per-enemy state updates to batched dispatch. Eliminates 2-second frame drops at 250+ enemies.

| Before | After |
|--------|-------|
| N x `removeEnemy()` + N x `addExplosion()` + N x `addScore()` | Collect all kills → single `addScore()` → batch remove → batch explosions |
| `.clone()` per enemy | Inline component math, squared-distance early exit |
| Chain lightning: N_hits x N_enemies nested | `Set`-based dedup, 3-chain cap per primary |

### Missile Direct Hit Damage

Removed `computeCoverFactor` from direct missile hits. Missiles that physically reach the hit radius now always deal full base damage (100). AOE splash retains the cover modifier for wall-blocked splash.

### Tutorial POI Clustering Fix

Replaced single-cell `lastDiscoveryCell` tracking with a `Set<string>` of visited cells. Prevents repeated enemy/POI spawns from revisiting cells. Only newly explored cells trigger discovery spawns.

### Q/E Key Inversion

Vertical controls swapped: E = up, Q = down (previously reversed).

---

## 9. Capital Ship Movement (v0.44.0)

### Fleet Carrier Cruise

UNF ALBATROSS now cruises at 2 u/s along the X axis during gameplay, height-locked to Y=2900 with no rotation. Live position synced to `WorldState.carrierPosition` so the defense zone and extraction tracking follow the moving carrier.

| Parameter | Value |
|-----------|-------|
| Cruise speed | 2 u/s (X axis) |
| Height lock | Y = 2900 |
| Rotation | None (stable platform) |
| Defense zone | 500 units radius, follows carrier |

### Goon Frigate Orbit

Goon Frigates orbit the carrier at 600-unit radius using tangent vector movement, height-locked to carrier Y + 50. Frigates act as mobile spawn platforms.

| Parameter | Value |
|-----------|-------|
| Orbit radius | 600 units |
| Height offset | Carrier + 50 |
| Reinforcement spawn | 3-5 enemies (1 bomber + fighters) every 120 seconds |
| Retreat trigger | Player within 300 units |

### Missile Crash Fix (v0.44.1)

Fixed critical `ReferenceError` where `MissileController` referenced `weaponStats.damage` from the out-of-scope Player component. Replaced with hardcoded base damage (100). Also added persona speech debug button to ZeroVoice Debug screen and console warning for null persona claims.

---

## 10. INBOX Narrative System (v0.45.0)

### Choice-Based Narrative Inbox

Full branching narrative system integrated into the Hangar menu. The INBOX presents a terminal-themed message interface with character conversations, binary choices, and 16 possible endings.

### Story Structure

| Element | Count |
|---------|-------|
| Binary choice points | 4 |
| Unique endings | 16 (2^4) |
| Quest gates | 5 (Q0 through Q4) |
| Characters | 5 (VAEL, CAEL9, ALBATROSS, UNKNOWN, PLAYER) |
| Story threads | ~20 (P0, P1A/B, P2AA/AB/BA/BB, P3*, E_*) |

### Quest Gate Schedule

New narrative content unlocks every 5 levels. At milestone levels, the Mission Complete screen shows "NEW INBOX MESSAGES AVAILABLE" and routes the player to the Hangar instead of auto-continuing.

| Level Completed | Gate Unlocked | Content |
|----------------|--------------|---------|
| 5 | Q0_LAUNCH | P0 thread — first contact, first choice |
| 10 | Q1_CONTACT | P1A or P1B (based on P0 choice) |
| 15 | Q2_ENGAGEMENT | P2 thread — third arc |
| 20 | Q3_CRISIS | P3 thread — climax |
| 25 | Q4_RESOLUTION | Ending thread (1 of 16) |

### INBOX Interface

- **Thread sidebar** with quest-gated visibility and unread indicators
- **Message bubbles** — NPC messages left-aligned, player responses right-aligned
- **Character styling** — unique colours and avatars per character (VAEL, CAEL9, ALBATROSS, UNKNOWN)
- **Attachment chips** with paperclip icon (visual metadata on messages)
- **Choice buttons** [A][B][C][D] with keyboard shortcuts and staggered animation
- **Ending screen** with unique title, body text, and ending counter
- **Choice progress bar** in sidebar footer (5 green/grey dots tracking quest gates)
- **Path flags** — PATIENT, AGGRESSIVE, LISTENER, SPEAKER, etc. stored per choice

### Narrative Persistence

Player choices and quest gate progress persist to IndexedDB alongside other save data. Choices save immediately on selection (not deferred to level completion). Full save/load cycle:

| Event | Persistence |
|-------|------------|
| Make a choice | Saved to IndexedDB immediately |
| Close/reopen INBOX | Choices preserved (in-memory store) |
| Exit to menu → Continue | Choices loaded from IndexedDB |
| Close app → relaunch → Continue | Choices loaded from IndexedDB |
| New Game | All narrative progress reset |

### Quest Gate Interstitial (v0.45.1)

At levels 5, 10, 15, 20, and 25, the Mission Complete screen:
1. Hides the "START LEVEL" continue button
2. Shows "EXIT TO HANGAR" button (routes to Hangar for INBOX access)
3. Displays blue "NEW INBOX MESSAGES AVAILABLE" notification banner

### Level Progression Fix (v0.45.1)

Fixed continue button passing the current (completed) level instead of `level + 1`. Players now correctly advance to the next level.

### Speech Cache Key Fix (v0.45.1)

Fixed cache key mismatch between `pool.ts` (using `poolName` prefix) and `baker.ts` (hardcoded `goon:` prefix). Persona pool now correctly caches and retrieves baked voice lines across sessions.

### MissileController Scope Fix (v0.45.1)

Fixed `enemyById` being out of scope in MissileController. Added per-frame enemy lookup Map inside MissileController's `useFrame` callback, restoring missile kill functionality.

### POI Spawn Distance Guard (v0.45.1)

Added carrier-proximity guard: POIs no longer spawn within Manhattan distance 2 of the carrier's cell position. Prevents visual clutter near the player's home base.

---

## 11. Updated ZeroFamily System Reference

New systems added since v0.36.0:

| System | Domain | Key Principle |
|--------|--------|--------------|
| **ZeroResponse** | Procedural text | Template + pool selection via hash. Profile-driven, O(1) per line. |

### Updated Systems

| System | Change |
|--------|--------|
| **ZeroQuest** | Added Paradistro narrative profile with shell/covenant/builder theming |
| **ZeroVoice** | Multi-pool architecture (goon + persona + reserved narrator/menu), `alienRadio` ring preset |
| **ZeroCover** | Removed from direct missile hits; retained for AOE splash only |

### Controls Reference (Updated)

| Action | Input | Notes |
|--------|-------|-------|
| Transform Mecha ↔ Flight | F | Toggle (was speed-triggered) |
| Afterburner | Tab (hold) | Flight mode only, 8x maxSpeed |
| Vertical Up | E | Changed from Q |
| Vertical Down | Q | Changed from E |
| Forward Thrust (Mecha) | W | Newtonian — thrust applies, drift on release |
| Strafe (Mecha) | A / D | Newtonian — lateral drift persists |
| Reverse (Mecha) | S | Newtonian — backward thrust |
| Emergency Airbrake | F (in flight) | 2-second braking ramp, cancels afterburner |

### Version History (v0.37.0 – v0.45.1)

| Version | Type | Summary |
|---------|------|---------|
| v0.37.0 | feat | O(1) enemy lookup, weapon mode gating, 8x afterburner, funnel visuals |
| v0.38.0 | feat | F-key mecha/flight toggle, Paradistro quest profile |
| v0.39.0 | feat | Newtonian mecha momentum, full maxSpeed, drift physics |
| v0.40.0 | fix | 4s dampening, 2s braking ramp, afterburner cancel, ship rotation |
| v0.41.0 | feat | Thrust-linked engine glow, boolean afterburner emitters |
| v0.41.1 | fix | Constant-size enemy indicators, .gitattributes cleanup |
| v0.42.0 | feat | Fleet Carrier persona speech, multi-pool architecture, ATC comms |
| v0.43.0 | fix | QWE batched kills, missile accuracy, POI clustering, Q/E swap |
| v0.44.0 | feat | Carrier cruise movement, frigate orbit + reinforcement spawns |
| v0.44.1 | fix | Missile ReferenceError crash, persona debug button |
| v0.45.0 | feat | INBOX narrative CYOA — 16 endings, 4 choices, 5 quest gates |
| v0.45.1 | fix | Missile kills, POI distance, speech cache, level progression, quest gates |
