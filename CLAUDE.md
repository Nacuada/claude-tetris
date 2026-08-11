# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Vanilla JavaScript implementation of Tetris using HTML5 Canvas. No build process, no package manager, no external dependencies — just three files: `index.html`, `style.css`, `game.js`.

## Running the game

Open `index.html` directly in a browser, or serve it locally:

```bash
python3 -m http.server 8000   # then open http://localhost:8000
```

There is no build, lint, or test tooling in this repo — changes to `game.js` are verified by playing the game in the browser.

## Architecture

All game logic lives in `game.js` as top-level functions operating on module-level mutable state (`board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, `dropInterval`, etc.) — there are no classes or modules.

- **Board model**: `board` is a `ROWS × COLS` matrix where each cell is `0` (empty) or an integer 1–7 identifying the piece color (see `COLORS`).
- **Pieces**: defined in `PIECES` as square matrices. Rotation is done via `rotateCW` (transpose + row reversal), not by storing pre-rotated states.
- **Collision** (`collide`): checks board bounds and overlap with locked cells for an arbitrary shape/offset.
- **Wall kicks** (`tryRotate`): after rotating, tries offsets `[0, -1, 1, -2, 2]` and keeps the first that doesn't collide.
- **Game loop** (`loop`): driven by `requestAnimationFrame`; accumulates elapsed time in `dropAccum` and advances the piece one row once `dropInterval` is exceeded.
- **Locking/scoring**: `lockPiece` → `merge` (writes piece into `board`) → `clearLines` (removes full rows, updates score/lines/level using `LINE_SCORES` × `level`) → `spawn` (promotes `next` to `current`, generates a new `next`, and calls `endGame` if the new piece immediately collides).
- **Level/speed**: level increases every 10 lines; `dropInterval = max(100, 1000 - (level - 1) * 90)` ms.
- **Ghost piece**: `ghostY` projects `current` straight down to its landing row; drawn with `globalAlpha = 0.2` in `draw`.
- **Rendering**: `draw()` redraws the full board canvas each frame (grid, locked cells, ghost, current piece); `drawNext()` renders the preview piece on a separate canvas.

If `COLS`, `ROWS`, or `BLOCK` are changed, the `<canvas id="board">` `width`/`height` in `index.html` must be updated to match (`COLS × BLOCK`, `ROWS × BLOCK`).

Controls (keydown handler at the bottom of `game.js`): arrows to move, `↑`/`X` to rotate, `↓` for soft drop, `Space` for hard drop, `P` to pause.
