# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

A classic Tetris implementation in vanilla JavaScript, HTML5 Canvas, and CSS. No dependencies, no build step, no package manager — just three files: `index.html`, `style.css`, `game.js`.

## Running the game

There is no build/lint/test tooling. To run:

```bash
start index.html       # Windows: open directly in browser
```

Or serve it locally (needed if you hit CORS/module issues, otherwise direct open works fine):

```bash
python3 -m http.server 8000
npx serve .
php -S localhost:8000
```

Then visit `http://localhost:8000`. After editing `game.js`, `style.css`, or `index.html`, just reload the browser tab — there is no compile step.

## Architecture

Everything lives in `game.js` as top-level module state (no classes, no build-time modules — plain script tag loaded by `index.html`).

- **Board model**: `board` is a `ROWS × COLS` (20×10) matrix; each cell is `0` (empty) or an integer 1–7 identifying which piece color occupies it (indexes into `COLORS`).
- **Pieces**: `PIECES` defines the 7 tetrominoes plus a special 3×3 "Tuerca" (nut) ring piece with an empty center, all as square matrices. `randomPiece` picks uniformly from `PIECES.length - 1` types. Rotation is done by `rotateCW`, a transpose + row-reverse — there is no precomputed rotation state table.
- **Collision**: `collide(shape, ox, oy)` checks a shape at an offset against board bounds and existing fixed blocks. It's the single choke point used by movement, rotation, spawning, and ghost-piece projection.
- **Wall kicks**: `tryRotate()` rotates then tries offsets `[0, -1, 1, -2, 2]` columns until one doesn't collide, otherwise the rotation is discarded.
- **Game loop**: `loop(ts)` runs via `requestAnimationFrame`, accumulating elapsed time in `dropAccum` and dropping the piece one row once it exceeds `dropInterval`. `animId` holds the current RAF handle so pause/game-over can cancel it.
- **Locking pieces**: `lockPiece()` → `merge()` (bakes the current shape into `board`) → `clearLines()` → `spawn()` (promotes `next` to `current`, generates a new `next`, and calls `endGame()` if the new piece immediately collides).
- **Scoring/leveling**: `LINE_SCORES = [0, 100, 300, 500, 800]` multiplied by `level`; hard drop adds 2 pts/row dropped, soft drop adds 1 pt/row. `level` increases every 10 lines; `dropInterval = max(100, 1000 - (level-1)*90)`.
- **Rendering**: `draw()` clears and redraws the grid, fixed board blocks, the ghost piece (via `ghostY()`, drawn at `globalAlpha = 0.2`), and the current piece, every frame. `drawNext()` renders the preview piece to the separate `next-canvas`.
- **Input**: a single `keydown` listener dispatches on `e.code` (arrows for move/soft-drop, Up/X to rotate, Space to hard-drop, P to pause). Input is ignored while `paused` or `gameOver` is true, except the pause key itself.

### Tunable constants (top of `game.js`)

`COLS`, `ROWS`, `BLOCK` (cell size), `COLORS`, `PIECES`, `LINE_SCORES`, and the initial `dropInterval`. If `COLS`/`ROWS`/`BLOCK` change, update the `<canvas id="board">` `width`/`height` attributes in `index.html` to match (`COLS × BLOCK` and `ROWS × BLOCK`).

## Files

- `index.html` — DOM structure: main board canvas (300×600), side panel (score/lines/level/next-piece preview/controls), and a pause/game-over overlay.
- `style.css` — dark/retro arcade theme; flexbox layout, monospace HUD values, backdrop-blur overlay.
- `game.js` — all game logic (~300 lines), described above.
