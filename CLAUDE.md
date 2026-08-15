# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Asteroids clone built with plain HTML5 Canvas and vanilla ES6+ JavaScript. No build tools, no bundler, no package manager, no dependencies, no tests. The entire game logic lives in a single file: `game.js` (~420 lines). `index.html` just sets up an 800x600 canvas and loads `game.js` as a plain script tag.

## Running

Open `index.html` directly in a browser, or serve it locally:

```bash
npx serve .
```

There is no build step, lint config, or test suite — changes to `game.js` take effect on browser reload.

## Architecture

Everything runs in `game.js` as a single `requestAnimationFrame` loop (`loop` → `update(dt)` then `draw()`) operating on module-level mutable state (`ship`, `bullets`, `asteroids`, `particles`, `score`, `lives`, `level`, `state`).

- **Entity classes** (`Bullet`, `Asteroid`, `Ship`, `Particle`) each own `update(dt)` and `draw()` methods and a `dead` flag; the main loop advances them and filters out dead ones each frame rather than using any entity-management/ECS abstraction.
- **Game state machine**: the `state` variable is one of `'playing' | 'dead' | 'gameover'`, checked at the top of `update()` to branch behavior (e.g. respawn countdown via `deadTimer`, restart-on-Space in game over).
- **Toroidal space**: all positions wrap via the `wrap(v, max)` helper — asteroids, bullets, and the ship all reappear on the opposite edge.
- **Asteroids split recursively**: `Asteroid.split()` produces two smaller asteroids (size 3 → 2 → 1, then destroyed) using the `RADII`/`SPEEDS`/`POINTS` arrays indexed by size.
- **Collision detection** is plain circle-distance checks (`dist(a, b) < radiusSum`) done in `update()`, not delegated to entity classes.
- **Input**: raw keyboard state lives in `keys` (held) and `justPressed` (edge-triggered, consumed via `pressed(code)`) — used for movement (`keys`) vs. one-shot actions like shooting/restart (`pressed`).

When adding new entity types or behaviors, follow the existing pattern: a class with `update(dt)`/`draw()`/`dead`, pushed into one of the module-level arrays, filtered each frame in `update()`.

Note: the README describes power-ups and a "shooting star" asteroid type — these were removed from the code (see git history) and the README is currently stale on that point.
