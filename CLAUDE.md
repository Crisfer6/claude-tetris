# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

A classic Tetris implementation in vanilla JavaScript, HTML5 Canvas, and CSS. No dependencies, no build step, no package.json.

## Running

Open `index.html` directly in a browser, or serve it with any static server (e.g. `python3 -m http.server 8000`, `npx serve .`). There is no build, lint, or test tooling in this repo.

## Architecture

Three files, no modules:

- `index.html` — DOM structure: a 300×600 `#board` canvas, a 120×120 `#next-canvas` for the next-piece preview, HUD spans (`#score`, `#lines`, `#level`), and a shared `#overlay` used for both PAUSE and GAME OVER states.
- `style.css` — dark/retro arcade theme.
- `game.js` — all game logic, using module-level `let` state (`board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, `dropAccum`, `dropInterval`, `animId`) rather than a class or state container.

### Core mechanics in `game.js`

- **Board**: `ROWS × COLS` matrix; each cell is `0` (empty) or a piece color index `1–7`.
- **Pieces**: square matrices in `PIECES`; rotation is `rotateCW` (transpose + reverse), not precomputed rotation states.
- **Collision**: `collide(shape, ox, oy)` checks board bounds and overlap with locked cells.
- **Wall kicks**: `tryRotate()` tries offsets `[0, -1, 1, -2, 2]` after rotating, keeping the first that doesn't collide.
- **Game loop**: `loop(ts)` runs via `requestAnimationFrame`, accumulates `dt` into `dropAccum`, and advances the piece (or calls `lockPiece()`) once `dropAccum >= dropInterval`.
- **Locking**: `lockPiece()` → `merge()` writes the piece into `board`, `clearLines()` removes full rows (scans bottom-up, splices + unshifts empty rows), then `spawn()` promotes `next` to `current` and generates a new `next`; if the new piece immediately collides, `endGame()` fires.
- **Scoring**: `LINE_SCORES = [0, 100, 300, 500, 800]` × current `level`; hard drop adds 2 pts/row dropped, soft drop adds 1 pt/row.
- **Leveling/speed**: level = `floor(lines / 10) + 1`; `dropInterval = max(100, 1000 - (level - 1) * 90)` ms.
- **Ghost piece**: `ghostY()` projects `current` straight down until collision; drawn at `globalAlpha = 0.2`.

Tunable constants live at the top of `game.js`: `COLS`, `ROWS`, `BLOCK`, `COLORS`, `LINE_SCORES`, initial `dropInterval`. If `COLS`/`ROWS`/`BLOCK` change, the `#board` canvas `width`/`height` in `index.html` must be updated to match (`COLS × BLOCK`, `ROWS × BLOCK`).

Input is handled by a single `keydown` listener at the bottom of `game.js` (arrow keys, `X` to rotate, `Space` for hard drop, `P` to pause); there's no input abstraction layer.


