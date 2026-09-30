# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Classic Tetris in vanilla JavaScript + HTML5 Canvas + CSS. No dependencies, no package.json, no bundler/transpiler, no tests or linter. UI text and README are in Spanish; keep user-facing strings in Spanish.

## Running

Open `index.html` directly in a browser, or serve the folder statically (preferred):

    python -m http.server 8000     # then http://localhost:8000
    npx serve .

There is no build step; edits are picked up by reloading the page.

## Architecture

All game logic lives in `game.js` (one script, `'use strict'`, loaded at the end of `index.html`; no modules). It grabs DOM elements by id at load time, so ids in `index.html` (`board`, `next-canvas`, `score`, `lines`, `level`, `overlay`, `overlay-title`, `overlay-score`, `overlay-record`, `highscore`, `restart-btn`) are a hard contract with `game.js`.

- **State** is a set of module-level `let` globals (`board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, `dropInterval`, `animId`, …) that are all reset in `init()`. `init()` is also the restart handler.
- **Board/piece encoding**: `board` is a `ROWS × COLS` matrix. 0 = empty, 1–7 = piece type. The same index selects the shape in `PIECES`, the color in `COLORS`, and is the value stored in the shape's cells. Add or change pieces by keeping all three arrays aligned (index 0 is `null`).
- **Movement** always goes through `collide(shape, x, y)`, which checks board bounds and locked cells and allows negative y (above the board). Rotation is `rotateCW` (matrix rotation) plus `tryRotate`, which applies simple horizontal wall kicks `[0,-1,1,-2,2]`. This is not SRS.
- **Lock sequence**: `lockPiece()` → `merge()` → `clearLines()` (updates lines/score/level/`dropInterval`) → `spawn()`. `spawn()` promotes `next` to `current` and calls `endGame()` if the new piece collides immediately.
- **Loop**: `loop(ts)` runs on `requestAnimationFrame` and accumulates `dt` into `dropAccum` for gravity. It redraws the whole board every frame (`draw()`: grid → locked cells → ghost at alpha 0.2 → current piece). The next-piece preview (`drawNext`) is redrawn only on `spawn()`. Pause and game over stop the loop with `cancelAnimationFrame(animId)`. Unpausing restarts it by calling `loop()` directly.
- **HUD** (`updateHUD`) is refreshed after every keydown and on line clears, not every frame.
- **Scoring**: `LINE_SCORES[cleared] * level`. A soft drop scores +1 per row and a hard drop +2 per row. Level = `floor(lines/10)+1`. `dropInterval = max(100, 1000 - (level-1)*90)` ms.

## Coupled values

- `COLS`, `ROWS`, and `BLOCK` in `game.js` must match `<canvas id="board" width="300" height="600">` in `index.html` (`COLS*BLOCK × ROWS*BLOCK`).
- The preview assumes a 4×4 grid of 30px cells (`NB = 30` in `drawNext`), which matches `<canvas id="next-canvas" width="120" height="120">`.
