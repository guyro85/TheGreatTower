# Game Features Design — The Great Tower

**Date:** 2026-03-27

## Overview

Add four features to the existing HTML5 canvas platformer: a speed-based score system, falling platforms, floor labels, and a three-type enemy system. All changes are self-contained within `index.html`.

---

## Feature 1: Score System

**Scoring logic:** Speed Multiplier approach.

- `speedMult` starts at 1.0, max 4.0
- Increases by `scrollAmount * 0.03` each frame the player scrolls upward
- Decays by 0.01 per frame when not scrolling fast (floor at 1.0)
- `score += Math.ceil(scrollAmount * speedMult * 0.5)` per frame during upward scroll
- HUD (top-left): score value + current multiplier (highlighted gold when > 1.5×)
- Game over screen: final score + "Press R to restart"
- R key triggers `restartGame()` which resets all state

**Death conditions:** player.y reaches canvas bottom.

---

## Feature 2: Platform Floor Labels

- Global `platformCounter` increments on every platform creation and recycle
- Each platform stores its `number` field
- If `platform.number % 10 === 0`, draw `"FLOOR N"` centered on the platform
- Style: bold 9px Arial, white fill with black stroke (2px) for readability

---

## Feature 3: Falling Platforms

- Each platform gains: `standTimer` (ms), `isStoodOn` (bool), `falling` (bool), `fallSpeed` (px/frame)
- `isStoodOn` is reset every frame, set true during collision resolution
- `standTimer += FRAME_TIME` (~16.67ms) while stood on; resets to 0 when player leaves
- At `standTimer >= 4000`: `falling = true`
- Falling platforms skip collision checks; `fallSpeed += 0.3` per frame, `platform.y += fallSpeed`
- Visual warning: a colored bar above the platform fills left-to-right over 4s (green → red)
- Fallen platforms (y > canvas.height + 50) are recycled as new platforms at the top

---

## Feature 4: Enemy System

**Spawn rate:** 5% of platforms get one enemy (checked on creation and recycle).

**Type distribution** (via `Math.random()`):
- `roll < 0.5` → Type 1 (50%)
- `roll < 0.9` → Type 2 (40%)
- else → Type 3 (10%)

**Enemy storage:** `{ type, offsetX, width:20, height:20, velX, shootTimer, shootInterval }`
- `offsetX` is relative to `platform.x` (ensures correct scroll behavior)
- Sits on top of platform: absolute y = `platform.y - enemy.height`

**Type 1 (dark red):** Stationary. No movement logic.

**Type 2 (orange):** Moves side to side. `offsetX` changes by `velX (±1)` each frame, clamped to `[0, platform.width - enemy.width]`. Reverses direction at edges.

**Type 3 (purple):** Stationary. Shoots a bullet toward the player every 120 frames (2s at 60fps). Bullet velocity normalized toward player position at time of shot, speed = 3px/frame.

**Bullets:** `{ x, y, velX, velY, r:4 }` stored in global array. Scroll with world (y += scrollAmount). Removed when off-screen.

**Collision:** AABB check (player rect vs enemy rect, player rect vs bullet bounding box). Any hit → `gameOver = true`.

---

## Implementation Order (for commits)

1. Score system + game over + restart → commit + push
2. Platform floor labels → commit + push
3. Falling platforms → commit + push
4. Enemy system → commit + push
