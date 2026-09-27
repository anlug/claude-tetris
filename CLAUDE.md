# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

A classic Tetris implementation in vanilla JavaScript (HTML5 Canvas + CSS). No dependencies, no build step, no package.json.

## Running the game

Open `index.html` directly in a browser, or serve it with any static server:

```bash
python3 -m http.server 8000
# or
npx serve .
```

There are no build, lint, or test commands — this project has no tooling configured. Verify changes by loading `index.html` in a browser (or via the `run`/`browser-automation` skills) and playing the game.

## Architecture

Three files, no modules, everything global:

- `index.html` — DOM structure: the main `#board` canvas (300×600, 10×20 cells at 30px), a `#next-canvas` (120×120) for the next-piece preview, HUD spans (`#score`, `#lines`, `#level`), and a `#overlay` div reused for both PAUSE and GAME OVER states.
- `style.css` — dark/retro arcade visual theme.
- `game.js` — all game logic, organized around a single mutable global state (`board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, `dropInterval`, etc.) rather than a class or module pattern.

### Core game loop

`init()` creates the board, spawns the first two pieces, and starts `requestAnimationFrame(loop)`. `loop(ts)` accumulates elapsed time in `dropAccum`; once it exceeds `dropInterval`, the current piece drops one row (or locks if it can't). `draw()` runs every frame regardless, rendering grid → locked board → ghost piece → current piece, in that order.

Keyboard input (`keydown` listener at the bottom of `game.js`) mutates `current` directly and is independent of the drop timer — movement/rotation is immediate, not queued.

### Key mechanics and where they live

- **Board model**: `ROWS × COLS` matrix; each cell is `0` (empty) or a color index `1–7` identifying which piece type locked there. `COLORS[]` and `PIECES[]` are parallel arrays indexed the same way.
- **Rotation** (`rotateCW`): transpose + reverse rows on the piece's shape matrix (pieces are always represented as square matrices).
- **Wall kicks** (`tryRotate`): after rotating, tries offsets `[0, -1, 1, -2, 2]` against `collide()` before giving up on the rotation.
- **Collision** (`collide`): checks board bounds and overlap with already-locked cells; used for movement, rotation, and ghost-piece projection alike.
- **Locking** (`lockPiece` → `merge` + `clearLines` + `spawn`): merges the current piece into `board`, clears completed rows (scanning bottom-up, re-checking the same row index after a splice since rows shift down), then spawns the next piece.
- **Game over**: detected in `spawn()` — if the newly spawned piece immediately collides, `endGame()` fires.
- **Scoring/leveling**: `LINE_SCORES = [0, 100, 300, 500, 800]` multiplied by `level`; hard drop adds 2 pts/row traveled, soft drop adds 1 pt/row. Level increments every 10 lines; `dropInterval = max(100, 1000 - (level-1)*90)`.
- **Ghost piece** (`ghostY`): projects the current piece straight down via repeated `collide()` checks, drawn at `globalAlpha = 0.2`.

### Tunable constants (top of `game.js`)

`COLS`, `ROWS`, `BLOCK` (cell size in px), `COLORS`, `LINE_SCORES`, `dropInterval` (initial). If `COLS`/`ROWS`/`BLOCK` change, the `#board` canvas `width`/`height` in `index.html` must be updated to match (`COLS×BLOCK` × `ROWS×BLOCK`).
