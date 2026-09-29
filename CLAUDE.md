# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Vanilla JavaScript Tetris on HTML5 Canvas. No dependencies, no `package.json`, no build step, no test suite, no linter. User-facing text (README, UI strings, overlay messages) is in Spanish — keep it that way.

## Running

Open `index.html` directly, or serve the folder statically (preferred):

```bash
python -m http.server 8000   # then http://localhost:8000
npx serve .
```

Verification is manual in the browser; there are no automated tests.

## Architecture

Three files: `index.html` (DOM: `#board` canvas, side panel with `#score`/`#lines`/`#level`, `#next-canvas`, `#overlay` for pause/game over), `style.css`, and `game.js`, which holds all logic as top-level functions sharing module-level mutable state (`board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, `dropInterval`, `animId`, …). `init()` resets all of it and is also the restart handler.

Key conventions in `game.js`:

- **One index for piece type, shape cell value, and color.** `PIECES[i]` shapes are filled with the value `i`, and `COLORS[i]` is its color; index 0 is `null`/empty. Board cells store that same index, so merging a piece just copies shape values. Adding a piece means extending both arrays in parallel and updating the `* 7` in `randomPiece()`.
- **Pieces** are `{ type, shape, x, y }` where `shape` is a square matrix; rotation is `rotateCW` (transpose + reverse) and `tryRotate` applies simple horizontal wall kicks `[0, -1, 1, -2, 2]`.
- **`collide(shape, ox, oy)`** is the single source of truth for legality (movement, rotation, ghost projection, spawn/game-over check). Cells with negative `y` are allowed above the board.
- **Game loop**: `loop` runs on `requestAnimationFrame`, accumulates `dt` into `dropAccum`, and drops one row per `dropInterval`. Pause/game over work by `cancelAnimationFrame(animId)`; resume restarts `loop`.
- **Lock path**: `lockPiece()` → `merge()` → `clearLines()` (updates lines/score/level/`dropInterval`) → `spawn()` (game over if the new piece collides immediately).
- **Rendering** is a full redraw each frame in `draw()` (grid → board → ghost at alpha 0.2 → current piece). `drawNext()` only runs on spawn and assumes a 4×4 grid of 30px cells (the 120×120 `#next-canvas`).

## Coupled values

- Canvas size in `index.html` (`300×600`) must equal `COLS × BLOCK` by `ROWS × BLOCK` in `game.js`.
- `#next-canvas` size (`120×120`) is tied to the `NB = 30` / 4-cell layout in `drawNext()`.
- Control keys appear in three places: the `keydown` handler, the controls list in `index.html`, and the README controls table.
