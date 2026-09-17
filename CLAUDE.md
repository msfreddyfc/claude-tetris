# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A single-page Tetris implementation in vanilla JavaScript (no framework, no build step, no dependencies, no `package.json`). Three files: `index.html`, `style.css`, `game.js`.

## Running / testing

There is no build, lint, or test tooling. To run the game, just serve or open `index.html`:

```bash
start index.html        # Windows, opens directly in default browser
python3 -m http.server 8000   # or any static server, then open http://localhost:8000
```

There are no automated tests. Verify changes by opening the game in a browser and playing it (check movement, rotation, line clears, scoring, pause, and game-over/restart).

## Architecture

All game logic lives in `game.js` as top-level functions operating on module-level mutable state (`board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, `dropInterval`, etc.) — there are no classes and no state management library.

- **Board model**: `board` is a `ROWS × COLS` array of arrays; each cell is `0` (empty) or an integer `1–7` indexing into `COLORS`/`PIECES` to identify which tetromino occupies it.
- **Pieces**: each of the 7 tetrominoes is a square matrix in `PIECES`. `randomPiece()` clones a shape and centers it at the top; `rotateCW()` rotates a shape via transpose + row-reverse (no separate rotation-state tracking, so kicks are recomputed from the freshly rotated shape).
- **Collision** (`collide`): the single source of truth for whether a shape can occupy a position — used by movement, rotation, spawn checks, and ghost-piece projection. Any new movement feature should go through this rather than re-deriving bounds checks.
- **Wall kicks** (`tryRotate`): after rotating, tries horizontal offsets `[0, -1, 1, -2, 2]` in order and keeps the first that doesn't collide.
- **Game loop** (`loop`): driven by `requestAnimationFrame`; accumulates elapsed time in `dropAccum` and advances the piece one row once `dropAccum >= dropInterval`. Pausing/resuming cancels/reschedules this loop rather than gating inside it.
- **Locking a piece** (`lockPiece`): merges the current piece into `board`, clears completed lines, then spawns the next piece — spawn immediately checks collision at the top to trigger `endGame()`.
- **Scoring/leveling**: line-clear points come from `LINE_SCORES` (`[0,100,300,500,800]`) multiplied by `level`; hard drop adds 2 points per cell dropped, soft drop 1 point per row. Level increases every 10 cleared lines, which recomputes `dropInterval` as `max(100, 1000 - (level-1)*90)`.
- **Rendering** (`draw`, `drawNext`, `drawBlock`, `drawGrid`): plain Canvas 2D, redrawn in full every frame — grid lines, locked board cells, a semi-transparent ghost piece (projected via the same `collide` logic used for movement), then the active piece. No dirty-rect optimization; keep this in mind before adding expensive per-frame work.

Tunable constants (`COLS`, `ROWS`, `BLOCK`, `COLORS`, `LINE_SCORES`, `dropInterval`) live at the top of `game.js`. Changing `COLS`/`ROWS`/`BLOCK` requires updating the `<canvas id="board">` `width`/`height` in `index.html` to match (`COLS×BLOCK` by `ROWS×BLOCK`).

The README (in Spanish) has additional detail on game flow and controls if needed.
