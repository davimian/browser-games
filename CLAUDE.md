# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running the Games

Open either HTML file directly in a browser — no build step or server required:

- `tictactoe.html` — Tic Tac Toe
- `shooter.html` — Neon Descent (shooter)

## Architecture

Two fully self-contained single-file games. All HTML, CSS, and JavaScript are inline in each file — no external dependencies, no modules, no bundler.

### tictactoe.html
DOM-based game. State is held in plain JS variables: a 9-element board array, current player, game-over flag, and session score counters. Win detection checks 8 hardcoded index patterns against the board array.

### shooter.html (~783 lines)
Canvas-based (640×480) wave shooter. Key systems:

- **Game loop:** `requestAnimationFrame` drives `update()` → `draw()` each frame
- **Entities:** Player, enemies (Grunt / Runner / Tank), bullets, and power-ups are plain JS objects stored in arrays; dead entities are filtered out each frame
- **Sprites:** Characters are encoded as pixel arrays (hand-drawn pixel art) and rendered onto the canvas
- **Waves:** 10 levels with escalating spawn counts and enemy mixes defined in a `levels` config array
- **Power-ups:** Health, Rapid Fire, Shield, Triple Shot, Speed Boost — each sets a timed flag on the player object
- **Persistence:** Hi-score stored in `localStorage`
- **Input:** WASD / arrow keys for movement, mouse position for aim, click or Space to shoot
