# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Classic Tetris in vanilla JavaScript, HTML5 Canvas, CSS. No dependencies, no build step, no `package.json`.

## Running

```bash
xdg-open index.html          # open directly, or:
python3 -m http.server 8000  # local server (recommended), then visit localhost:8000
```

No build, lint, or test commands exist in this project.

## Architecture

Three files, no modules:

- `index.html` — DOM shell: `#board` canvas (300×600, 10×20 grid at `BLOCK=30`px), `#next-canvas` preview, HUD spans (`#score`/`#lines`/`#level`), and `#overlay` for pause/game-over.
- `style.css` — dark/retro arcade visuals only.
- `game.js` — all game logic, single file, no classes, module-level mutable state (`board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, ...).

### Core model

- Board: `ROWS × COLS` matrix, each cell `0` (empty) or a color index `1–7` identifying the locked piece.
- Pieces: `PIECES` array of square matrices; `randomPiece()` picks one and spawns it centered at `y=0`.
- Rotation: `rotateCW` transposes + reverses rows; `tryRotate` applies wall kicks (`[0,-1,1,-2,2]` column offsets) until a non-colliding position is found.
- Collision: `collide(shape, ox, oy)` checks board bounds and overlap with locked cells.
- Locking: `lockPiece` → `merge` (bake piece into board) → `clearLines` (bottom-up full-row removal, unshifts empty row) → `spawn` (promote `next` to `current`, generate new `next`; if the new piece immediately collides, `endGame()` fires).

### Game loop

`requestAnimationFrame`-driven `loop(ts)` accumulates elapsed time in `dropAccum`; when it exceeds `dropInterval` the piece drops one row (or locks). `dropInterval = max(100, 1000 - (level-1)*90)` ms, level increments every 10 cleared lines. Scoring uses `LINE_SCORES = [0,100,300,500,800]` × level, plus 2 pts/cell for hard drop and 1 pt/row for soft drop.

Ghost piece (`ghostY`) projects the current piece straight down to its landing row and is drawn at `globalAlpha = 0.2`.

### Input

Single `keydown` listener switches on `e.code`: arrows move/soft-drop, `ArrowUp`/`KeyX` rotate, `Space` hard-drops (with `preventDefault`), `KeyP` toggles pause independent of `paused`/`gameOver` guards.

### Tunable constants (top of `game.js`)

`COLS`, `ROWS`, `BLOCK`, `COLORS`, `LINE_SCORES`, `dropInterval`. If `COLS`/`ROWS`/`BLOCK` change, update the `#board` canvas `width`/`height` in `index.html` to match (`COLS×BLOCK` by `ROWS×BLOCK`).
