# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository overview

A single, self-contained HTML file (`claude-runner.html`) implementing **ウェイッ！小金持ちランナー** — a Chrome-dinosaur-style endless runner. The player controls a lion-maned "Claude" character that double-jumps over poisonous mushrooms ("bugs") and collects yen coins and shields. UI strings are Japanese.

There is no build system, no package manager, no test suite, and no dependencies. The entire game (HTML, CSS, JavaScript) lives in one file with no imports.

## Running and developing

- **Run**: open `claude-runner.html` directly in a browser (`file://`) or serve the directory with any static server (e.g. `python3 -m http.server`). No build step.
- **"Tests"**: manual playtesting in a browser. There is no test framework — do not invent one.
- **High score** persists in `localStorage` under the key `cc_hi`. Clear it from the browser devtools to reset.
- **Audio** uses the Web Audio API and requires a user gesture before sound plays; the first `Space`/click initialises `audioCtx`.

## Architecture

Everything runs inside one `<script>` block in `claude-runner.html`. The structure, top to bottom, is:

1. **Canvas + constants** — `W`, `H`, `GROUND` (y of the floor line). The runner is pinned to `GROUND` and physics treat that as ground level.
2. **Audio helpers** — `beep()` plus `sfxJump/Coin/Shield/Hit/Land` build all SFX from oscillator primitives. No sample assets.
3. **Global game state** — module-level `let`s: `state` (`'idle' | 'running' | 'dead'`), `score`, `coins`, `hiScore`, `frame`, `speed`, and the entity arrays `obstacles`, `pickups`, `particles`, `floatTexts`. Mutated directly by the loop; no framework.
4. **Entities**:
   - `claude` object — player character with `update()` / `draw()` / `rect()` (hitbox is inset from sprite bounds via `rect()`) and a squash-and-stretch animation tied to landing.
   - `BUG_TYPES` + `spawnObstacle()` / `drawObstacle()` — mushroom enemies; obstacles are drawn as menacing toadstools, not labelled bugs.
   - `spawnPickup()` / `drawPickup()` — coins (75%) and shields (25%); coin spin is faked by scaling x-radius with `sin()`.
   - `spawnParticles()` / `addFloatText()` — short-lived visual effects consumed by `updateParticles()`.
5. **Main loop** — `loop()` is the single `requestAnimationFrame` driver. It branches on `state`:
   - `idle`: draws character + title overlay.
   - `running`: advances `score`/`speed`, computes level (`Math.floor(score/400)+1`), spawns obstacles/pickups on frame counters (`nextObstacle`, `nextPickup`), runs collision (`rectsOverlap` for obstacles, `circleRect` for pickups), and renders.
   - `dead`: freezes entities and shows game-over overlay.
6. **Input** — `Space` / `ArrowUp` / click / touchstart all route through `handleAction()`, which either calls `resetGame()` or `claude.jump()` depending on `state`. Double-jump is enforced by `claude.jumpCount < 2`.
7. **Utils** — `roundRect()` at the bottom.

### Conventions worth preserving

- **State machine via a single string** (`state`). Changes happen inline in the loop; keep it that way rather than introducing classes or a framework.
- **Entities are plain objects** in arrays, filtered each frame (`obstacles = obstacles.filter(...)`) and removed by setting `x = -200` after consumption. Match this pattern when adding new entity types.
- **Frame-based timing** — spawn cadence and shield duration (`shieldTimer = 300`) are in frames, not milliseconds. Difficulty curves key off `score` (e.g. `minGap = Math.max(55, 130 - Math.floor(score / 300) * 6)`).
- **Colour palette** is the GitHub dark theme (`#0d1117`, `#161b22`, `#30363d`, `#58a6ff`, `#f0883e`, `#3fb950`, `#da3633`). New visuals should pull from these.
- **All comments and user-facing strings are Japanese**; preserve the language when editing them.
- **No external assets** — every sprite is procedurally drawn with canvas primitives, every sound is synthesised. Don't add image/audio files unless the user asks.
