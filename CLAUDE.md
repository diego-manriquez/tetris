# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Vanilla JavaScript Tetris — HTML5 Canvas + CSS, no dependencies, no build step, no tests, no `package.json`. Three files: `index.html`, `style.css`, `game.js`.

## Running

Open `index.html` directly in a browser, or serve statically (`python3 -m http.server 8000`, `npx serve .`). No compile/lint/test tooling exists — changes are verified by reloading the page.

## Architecture (`game.js`)

Single-file IIFE-free script under `'use strict'`. All state lives in module-level `let` bindings (`board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, timing vars). `init()` resets everything and is also the restart handler.

- **Board model**: `board` is a `ROWS`×`COLS` array of ints. `0` = empty; `1–7` = piece type, which is also the index into `COLORS` and `PIECES`.
- **Pieces**: `PIECES[1..7]` are square matrices. Rotation (`rotateCW`) transposes + reverses rows; `tryRotate` applies basic wall kicks by testing x-offsets `[0,-1,1,-2,2]`.
- **Collision**: `collide(shape, x, y)` is the single gatekeeper for all movement, rotation, and drop logic.
- **Game loop**: `loop(ts)` via `requestAnimationFrame`; accumulates `dropAccum` and steps the piece down once it exceeds `dropInterval`. Pausing cancels the frame; resuming resets `lastTime` before re-entering `loop`.
- **Locking**: `lockPiece()` = `merge()` → `clearLines()` → `spawn()`. `spawn()` promotes `next` to `current`, generates a new `next`, and calls `endGame()` if the fresh piece already collides.
- **Scoring**: `LINE_SCORES` table × `level`; soft drop +1/row, hard drop +2/cell. Level = `floor(lines/10)+1`; speed = `max(100, 1000-(level-1)*90)` ms.
- **Rendering**: `draw()` clears and repaints grid + board + ghost piece (`ghostY`, alpha 0.2) + current piece every frame. `drawNext()` paints the preview canvas only on spawn.

Input is one `keydown` listener; arrows/X/Space act on `current` directly and then call `updateHUD()`. `P` toggles pause regardless of other state.

## Conventions

- UI-facing strings are Spanish (overlay text, README); code identifiers and comments are English.
- If you change `COLS`, `ROWS`, or `BLOCK`, update the `<canvas id="board">` `width`/`height` in `index.html` to match `COLS*BLOCK` × `ROWS*BLOCK`.
