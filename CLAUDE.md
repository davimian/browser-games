# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running the Games

Open either HTML file directly in a browser — no build step, server, or dependencies required:

```
open tictactoe.html
open shooter.html
```

## Repository Layout

Two standalone single-file games. All HTML, CSS, and JS are inline — no modules, no bundler, no package.json.

- `tictactoe.html` — DOM-based two-player Tic Tac Toe
- `shooter.html` — Canvas top-down wave shooter ("Neon Descent")

## shooter.html Architecture

~780 lines of vanilla JS on a 640×480 Canvas. The file is divided into named sections with `// ── SECTION ──` banners in this order:

**SPRITES → CONSTANTS → STATE → INPUT → INIT/RESET → WAVE SYSTEM → POWER-UPS → UPDATE SYSTEMS → PARTICLES → SHOOTING → SHAKE → DRAW → MAIN LOOP → BOOT**

### State machine

`gameState` is a string: `'MENU'` | `'PLAYING'` | `'LEVEL_COMPLETE'` | `'GAME_OVER'`. All transitions go through `setState(s)`, which resets arrays. `startGame()` and `advanceLevel()` bypass `setState` for PLAYING so they can configure wave state before assigning `gameState`.

### Game loop (`loop`)

`requestAnimationFrame` calls `loop(ts)` every frame. `dt` is capped at 50ms to survive tab-switch resume. Update and draw phases:

```
UPDATE (PLAYING only):
  tryShoot → updatePlayer → updateEnemies → updateBullets
  → updateParticles → updatePowerups → updateWave → checkCollisions

DRAW:
  applyShake (ctx.save + translate)
  drawBackground → drawPowerups → drawParticles → drawBullets
  → drawEnemies → drawPlayer
  resetShake (ctx.restore)    ← HUD is drawn AFTER restore so shake doesn't affect it
  drawHUD → [screen overlay]
```

### Entity arrays

`enemies`, `bullets`, `particles`, `powerups` are flat arrays of plain objects. All loops that may splice iterate **backward** (`for i = arr.length-1; i >= 0; i--`). Entities are created by spawn functions and removed inline during update/collision passes — no object pooling.

### Sprite system

Sprites (`S.PLAYER`, `S.GRUNT`, `S.RUNNER`, `S.TANK`) are 2-D arrays of palette indices (0 = transparent). `drawSprite(sprite, palette, cx, cy, scale, angle)` renders each non-zero index as a `fillRect` of `scale` pixels, centered on `(cx, cy)` and optionally rotated. All current sprites use `scale=2`. Palettes have 5 entries (indices 0–4).

### Enemy templates and wave definitions

`ENEMY_TEMPLATES` holds canonical stats (`r`, `speed`, `health`, `damage`, `score`, `glow`) for each type. `spawnEnemy(type)` spreads a template copy into a new object. `LEVELS` is keyed `1`–`10`; each value is an array of group descriptors `{ type, count, interval }`. `waveCtrl` consumes groups sequentially; when all groups are exhausted and `enemies` is empty, the level ends.

### Power-up system

Drop chance on enemy death is defined per type in `DROP_CHANCE`. `spawnPowerup` picks a type via weighted random from `POWERUP_POOL` / `POWERUP_WEIGHTS`. Each power-up object has `{ x, y, type, pulse, lifetime }` — `pulse` drives rotation and bob animation, `lifetime` starts at 10 000 ms and the pickup vanishes when it reaches zero. Pickup radius is 22 px.

`player.effects` holds four ms countdown fields: `rapidFire`, `shield`, `tripleShot`, `speedBoost`. `updatePlayer` decrements all of them each frame. Effects are read in `tryShoot` (fire rate and spread count) and `checkCollisions` (shield absorbs contact damage with a brief 300ms iframe instead of dealing damage).

### Collision detection

All collision is circle–circle (`dx²+dy² < (r₁+r₂)²`). Bullet–enemy loops iterate bullets in the outer loop and splice the bullet immediately on any hit to prevent double-hits.

### How to extend

**Add a level:** Append an entry to `LEVELS` and increment `MAX_LEVEL`. The win condition in `updateWave` and `advanceLevel` picks it up automatically.

**Add an enemy type:**
1. Add a pixel sprite to `S` and a palette to `PALETTES`.
2. Add a template to `ENEMY_TEMPLATES` and an entry to `DROP_CHANCE`.
3. Reference the type string in `LEVELS` wave groups.

**Add a power-up:**
1. Add an entry to `POWERUP_DEFS` (`color`, `glow`, `symbol`, `label`, `duration`).
2. Add the key to `POWERUP_POOL` and a weight to `POWERUP_WEIGHTS`.
3. Handle the key in `applyPowerup` and wherever it modifies gameplay (`tryShoot`, `updatePlayer`, etc.).
4. Add a HUD badge entry to the `effectDefs` array in `drawHUD`.

## Git workflow

Remote: `https://github.com/davimian/browser-games` (branch `main`). Every meaningful change is committed and pushed:

```
git add <files>
git commit -m "Short imperative summary

Longer explanation of why."
git push
```
