# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A single-page personal landing site for **sean.build** — a "links" / link-in-bio page with a heavy "forged in fire" ember/dragon visual theme. There is no build step, no framework, and no dependencies. Everything (HTML, CSS, JS) lives inline in [index.html](index.html); the only other assets are the favicons.

## Working in this repo

- **No build / lint / test.** Edit [index.html](index.html) and open it in a browser to see changes. Because animations key off `innerWidth`/`innerHeight`, `scrollY`, and pointer events, test by actually resizing, scrolling, and moving the mouse — not just a static load.
- **Serve over HTTP when testing icon/font paths.** Favicon links use absolute paths (`/favicon.ico`). Opening via `file://` will 404 those. Use any static server (e.g. `python -m http.server`) from the repo root.
- **External runtime deps are CDN fonts only** — Google Fonts (JetBrains Mono, Cinzel, Noto Serif JP). No package manager.

## Architecture

The page is one document with three layers stacked by `z-index`, all sharing the viewport via `position:fixed`:

- **Background (z 0):** `#boxes3d` canvas (3D neon wireframe cubes), the `#dragon-layer` SVG, and a CSS grid overlay on `body::before`.
- **Mid (z 1):** `#embers` canvas, `#fireflies` canvas, the `#bird` element, and `.glow` card backlights.
- **Content (z 2):** `.wrap` — avatar, heading, status, and the `.link` cards.

### The JavaScript is four independent IIFEs/blocks, each its own animation system

All share the same pattern: a `resize()` that sets canvas size to `devicePixelRatio`-scaled dimensions, a particle/object array, and a `requestAnimationFrame` loop. They do **not** share state — edit one without touching the others.

1. **Embers** — drifting upward particles on `#embers`.
2. **Bird flyby** — every 10s a blurred bird streaks across one random `.link` card, spawning a fire-particle trail (`spawnFire`) and toggling that card's `.glow` backlight. `cardGlows` are DOM divs positioned to track each card via `getBoundingClientRect()`; they are rebuilt/repositioned on resize and scroll. If you add or remove `.link` cards, the glow system adapts automatically (it queries `.link` live).
3. **3D boxes** (`#boxes3d`) — hand-rolled cube projection (`rot` → `project`), with `farPlace()` spacing them to the left/right gutters so they don't overlap the centered content. Interactive: pointer hit-testing lets you drag a box to spin it with inertia; `overCard()` ensures clicks/hover over `.link` cards pass through to the links instead of grabbing a box. Boxes have scroll parallax and wrap vertically to keep the field full.
4. **Fireflies** (`#fireflies`) — wandering glow particles with scroll parallax and cursor repulsion (`REPEL`/`FORCE`).

### Theme

Colors are CSS custom properties on `:root` (`--ember`, `--gold`, `--ash`, etc.). Canvas code uses its own hardcoded warm-hue values (e.g. `NEON='255,106,26'`, hue ranges in the particle spawners) rather than reading the CSS vars — change both if reskinning.

## Notable

- The link cards (`About Me`, `GitHub`, `LinkedIn`, `HackTheBox`, etc.) are currently placeholders pointing to `href="#"` except Contact (`mailto:i@sean.build`). Filling in real URLs is expected ongoing work.
- `.link` cards stagger their entrance via inline `animation-delay`; keep delays increasing if you add cards.
