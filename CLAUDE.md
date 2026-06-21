# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running the game

No build step. Open directly or serve statically:

```bash
open index.html                 # macOS
python3 -m http.server 8000     # then visit http://localhost:8000
npx serve .
```

## Architecture

Three files, no dependencies, no bundler:

- **`index.html`** — DOM structure: `<canvas id="board">` (300×600px) for the playfield, `<canvas id="next-canvas">` (120×120px) for the next-piece preview, a sidebar panel with score/lines/level displays, and a single `#overlay` div reused for both PAUSE and GAME OVER states.
- **`style.css`** — Dark/retro aesthetic. The overlay uses `backdrop-filter: blur` over the board canvas; `.hidden` simply sets `display: none`.
- **`game.js`** — All game logic (~300 lines, strict mode). Key design points:

  - **Board**: `ROWS × COLS` matrix of integers. `0` = empty; `1–7` = piece color index matching `COLORS[]`.
  - **Pieces**: defined as 2D integer matrices in `PIECES[]`. `rotateCW` uses transpose + row-reverse.
  - **Wall kicks** (`tryRotate`): tries offsets `[0, -1, 1, -2, 2]` before discarding the rotation.
  - **Game loop** (`loop`): `requestAnimationFrame`-based, accumulates elapsed ms in `dropAccum`, triggers gravity when `dropAccum >= dropInterval`.
  - **Speed formula**: `dropInterval = Math.max(100, 1000 − (level − 1) × 90)` ms; level increments every 10 lines.
  - **Ghost piece** (`ghostY`): projects current piece downward, drawn at `globalAlpha = 0.2`.
  - **State**: all mutable state lives in module-level `let` variables (`board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, `animId`, etc.) reset by `init()`.

## Tunable constants (top of `game.js`)

| Constant | Default | Notes |
|---|---|---|
| `COLS` / `ROWS` | 10 / 20 | Also update canvas `width`/`height` in `index.html` (`COLS×BLOCK` / `ROWS×BLOCK`) |
| `BLOCK` | 30 | Pixel size of each cell |
| `COLORS` | 7 entries | Index 0 is `null` (empty); indices 1–7 match piece types |
| `LINE_SCORES` | `[0,100,300,500,800]` | Multiplied by current level |
