# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## Project

Vanilla Tetris implementation: HTML5 Canvas + CSS + plain ES6+ JavaScript. No dependencies, no build step, no package.json.

## Running / testing

There is no build or test tooling. To run the game, open `index.html` directly or serve the directory statically:

```bash
open index.html                 # macOS, opens directly
python3 -m http.server 8000     # or any static server
```

There are no automated tests or linters configured. Verify changes by opening the page and playing.

## Architecture

Three files, no modules/bundler — `index.html` loads `game.js` directly as a classic script.

- `index.html` — DOM shell: `<canvas id="board">` (300×600, 10×20 grid of 30px blocks), a side panel (score/lines/level/next-piece canvas), and a hidden overlay div reused for both PAUSE and GAME OVER states.
- `style.css` — dark/retro arcade styling.
- `game.js` — all game logic, in one file with module-level mutable state (`board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, `dropAccum`, `dropInterval`, `animId`).

Key mechanics in `game.js`:

- **Board model**: `ROWS × COLS` matrix; each cell is `0` (empty) or a piece color index (1–7).
- **Pieces**: square matrices in `PIECES`; rotation is done via `rotateCW` (transpose + reverse), with wall-kick offsets `[0, -1, 1, -2, 2]` tried in `tryRotate`.
- **Collision**: `collide(shape, ox, oy)` checks bounds and overlap against `board`.
- **Game loop**: `loop(ts)` runs on `requestAnimationFrame`, accumulates elapsed time in `dropAccum`, and drops the piece one row once `dropInterval` is exceeded; otherwise locks it via `lockPiece` (merge → clearLines → spawn).
- **Line clears**: `clearLines` scans bottom-up, splices full rows and unshifts empty ones at the top; scoring uses `LINE_SCORES = [0,100,300,500,800]` × current `level`. Level increases every 10 lines, which recomputes `dropInterval = max(100, 1000 - (level-1)*90)`.
- **Ghost piece**: `ghostY()` projects the current piece straight down; drawn at `globalAlpha = 0.2`.
- **Input**: a single `keydown` listener switches on `e.code` (arrows, `KeyX` for rotate, `Space` for hard drop, `KeyP` for pause), ignored while paused/game-over except unpause.

Tunable constants live at the top of `game.js`: `COLS`, `ROWS`, `BLOCK`, `COLORS`, `LINE_SCORES`, initial `dropInterval`. If `COLS`/`ROWS`/`BLOCK` change, update the `<canvas id="board">` `width`/`height` in `index.html` to match (`COLS×BLOCK` and `ROWS×BLOCK`).
