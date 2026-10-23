# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Tetris in vanilla JavaScript + HTML5 Canvas. No dependencies, no build step, no package manager, no tests, no linter. The README and UI text are in Spanish; keep user-facing strings in Spanish.

## Running

Open `index.html` directly, or serve statically (e.g. `python3 -m http.server 8000`) and visit `http://localhost:8000`.

The Java/Spring/Maven conventions in the global CLAUDE.md do not apply here.

## Architecture

Three files: `index.html` (DOM + canvases), `style.css`, and `game.js` (all logic, loaded as a classic script with `'use strict'`; no modules).

`game.js` is a single global-state game loop:
- State lives in module-level `let` variables (`board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, `dropInterval`, ...), all reset in `init()`. Restart button calls `init()` again.
- `board` is a `ROWS×COLS` matrix of ints; `0` is empty and `1–7` index into both `COLORS` and `PIECES` (the piece type id doubles as its color id, so shape cells hold their own type number).
- `loop(ts)` is the `requestAnimationFrame` loop: accumulates `dropAccum`, applies gravity when it exceeds `dropInterval`, then calls `draw()`. Pause/game over work by `cancelAnimationFrame(animId)`; unpausing restarts `loop`.
- Piece lifecycle: `lockPiece()` → `merge()` → `clearLines()` (updates lines/score/level/`dropInterval`) → `spawn()` (promotes `next`, ends the game if the new piece collides immediately).
- `collide(shape, ox, oy)` is the single collision check used by movement, rotation (`tryRotate` tries wall-kick offsets `[0,-1,1,-2,2]`), ghost piece (`ghostY`), and spawn.
- Input is a single `keydown` handler (arrows, X to rotate, Space hard drop, P pause). Scoring: line clears use `LINE_SCORES * level`; soft drop +1/cell, hard drop +2/cell.

Board dimensions are coupled between `game.js` (`COLS`, `ROWS`, `BLOCK`) and the `<canvas id="board" width="300" height="600">` attributes in `index.html`; change both together. Likewise the next-piece preview assumes a 4×4 grid of 30px blocks on the 120×120 `next-canvas`.
