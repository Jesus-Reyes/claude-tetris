# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running the game

No build step, no dependencies. Open directly or serve statically:

```bash
open index.html                  # macOS — open directly
python3 -m http.server 8000      # then visit http://localhost:8000
npx serve .                      # alternative static server
```

## Architecture

Single-file game logic in `game.js` (~305 lines, `'use strict'`, no modules). All state is module-level `let` variables:

```
board         — ROWS×COLS matrix; 0 = empty, 1–7 = piece color index
current/next  — { type, shape, x, y } objects
score/lines/level/paused/gameOver/dropInterval/dropAccum/lastTime/animId
```

**Rendering pipeline** (`draw` → called every frame):
1. `drawGrid()` — faint grid lines on the board canvas
2. Board matrix → `drawBlock` for each non-zero cell
3. Ghost piece at `ghostY()` with `globalAlpha = 0.2`
4. Current piece on top

**Game loop**: `requestAnimationFrame(loop)` accumulates elapsed time in `dropAccum`; when it exceeds `dropInterval` the piece drops one row or locks.

**Piece locking sequence**: `lockPiece` → `merge` (stamp piece into board) → `clearLines` (scan bottom-up, splice full rows) → `spawn` (promote `next` to `current`, generate new `next`; if collision on spawn → `endGame`).

**Rotation**: `rotateCW` (transpose + reverse rows) with wall-kick offsets `[0, -1, 1, -2, 2]` tried in order.

## Key constants (all in `game.js`)

| Constant | Default | Notes |
|---|---|---|
| `COLS` / `ROWS` / `BLOCK` | 10 / 20 / 30 | If changed, also update `width`/`height` on `<canvas id="board">` in `index.html` (`COLS×BLOCK` × `ROWS×BLOCK`) |
| `COLORS` | 7 colors | Index 1–7 maps to piece types I/O/T/S/Z/J/L |
| `LINE_SCORES` | `[0,100,300,500,800]` | Multiplied by current level |
| Initial `dropInterval` | 1000 ms | Speed formula: `max(100, 1000 − (level−1) × 90)` |
