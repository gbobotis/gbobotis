# Webapp-as-CV — Design Spec
**Date:** 2026-03-31
**Status:** Approved

---

## Overview

A single-page personal brand website that functions as an interactive CV. The app itself is the pitch — a cinematic scrollytelling experience with a sci-fi RPG aesthetic, driven by a live Three.js 3D animation that morphs as the visitor scrolls. Target audience is universal (Web3, traditional tech, freelance clients). The goal is equal parts craft demonstration and charisma.

---

## Architecture

**Single HTML file** deployed to GitHub Pages. No build step, no framework.

```
index.html
assets/
  css/style.css       ← Tailwind v4 CDN + custom CSS variables
  js/
    scroll.js         ← scroll progress tracking, chapter detection
    scene.js          ← Three.js scene, camera, lights, renderer, bloom
    animation.js      ← scroll-driven vertex morphing logic
```

**Rendering model:** A full-viewport Three.js `WebGLRenderer` canvas sits fixed behind all content (`position: fixed`, `z-index: 0`, `alpha: true`). HTML chapters scroll over it in transparent sections. On each `requestAnimationFrame` tick, `scroll.js` writes a `chapterProgress` value (0–1) to a shared state object, which `scene.js` reads to drive the 3D animation.

---

## Visual Language

**Palette:**
- Background: `#050508` (deep space black)
- Accent 1: `#00f5ff` (electric cyan)
- Accent 2: `#8b5cf6` (violet)
- Accent 3: `#ff0080` (hot magenta)
- Text: cold white `#f0f0f5` / mid-grey `#6b7280`

**Typography:**
- HUD / monospace elements: `JetBrains Mono`
- Body / headings: `Inter`

**HUD overlay** (fixed, always visible, `z-index: 50`):
- Top-left: callsign — your name/handle
- Top-right: chapter indicator — `[ CHAPTER 01 / 04 — ORIGIN ]`
- Bottom: thin scan-progress bar tracking scroll depth

**Per-chapter effects:**
- Headings: glitch-in text animation on viewport entry
- Skills: RPG stat chips — `[ SOLIDITY ██████░░ ] EXPERT`
- Project cards: scanline texture, CRT border glow, scanline-wipe reveal on scroll
- Cursor: custom CSS crosshair

**3D lighting:** Three `PointLight`s (cyan, violet, magenta) slowly orbiting the object. `UnrealBloomPass` for bloom/glow. Iridescent, vibrant, physically shaded.

---

## Scroll Chapters

Each chapter is ~100vh. Scroll progress per chapter is a `0–1` float driving the 3D morph.

### Chapter 0 — Origin
- **Content:** Full-viewport hero. Name glitches in letter by letter. One punchy tagline. 2–3 sentences of origin story.
- **3D state:** Single glowing node / tight particle seed. Object at rest, slowly pulsing.
- **CTA:** None. Pure presence.

### Chapter 1 — Skills `[ CAPABILITIES LOADED ]`
- **Content:** Skills grouped by domain (Frontend / Web3 / Tools & Infra). Each rendered as an RPG stat row: name + block bar + level label. Hover tooltip with brief context.
- **3D state:** Node expands outward into a molecular lattice / node web.

### Chapter 2 — Work `[ MISSION LOG ]`
- **Content:** 2–4 project cards. Each: title, one-line description, tech tag chips, external link. Cards reveal with scanline-wipe on scroll entry. Hover: border glow intensifies, scanlines animate.
- **3D state:** Lattice morphs into a rotating DNA double helix — fully formed and turning.

### Chapter 3 — Vision `[ NEXT OBJECTIVE ]`
- **Content:** Short paragraph on what you're building toward. Three CTA buttons emerge: `[ HIRE ME ]` (email) / `[ SEE MY WORK ]` (GitHub) / `[ LET'S TALK ]` (Telegram, Discord, or LinkedIn — to be chosen by you). All open in new tab.
- **3D state:** Helix dissolves into drifting light particles as scroll reaches bottom.

---

## Three.js Implementation

**Scene (`scene.js`):**
- `WebGLRenderer` with `alpha: true`, `antialias: true`
- `EffectComposer` + `UnrealBloomPass` (strength ~1.5, radius 0.4, threshold 0.1)
- `PerspectiveCamera` — fixed position, looking at origin
- Render loop only active when tab is visible (`visibilitychange` pause/resume)
- Canvas resolution: `devicePixelRatio` capped at 2 for performance

**3D Object:**
- `BufferGeometry` with custom vertex positions
- Under 2,000 vertices total for mobile safety
- Four pre-computed vertex target arrays (one per chapter): seed cluster → lattice → helix → dispersed particles
- On each frame: lerp current positions toward active chapter targets, weighted by `chapterProgress`

**Morphing (`animation.js`):**
- Reads `state.chapterIndex` and `state.chapterProgress` (0–1) from shared state
- Lerps vertex positions each frame — smooth, continuous, no jumps
- Lights orbit object on independent timed cycle (not scroll-driven)

**Scroll system (`scroll.js`):**
- Tracks `window.scrollY` on `requestAnimationFrame` (not `scroll` event — avoids jank)
- Computes active chapter index and progress within that chapter
- Writes to shared `state` object read by `scene.js`

---

## Interactions

| Element | Behaviour |
|---|---|
| Heading enter viewport | Glitch-in text animation (CSS + JS class toggle) |
| Skill bar hover | Tooltip with brief context note |
| Project card hover | Border glow intensifies, scanlines animate |
| CTA button hover | Pulse glow, slight scale-up |
| All external links | `target="_blank" rel="noopener"` |
| Cursor | Custom CSS crosshair via `cursor: none` + pseudo-element |

---

## Deployment

- **Host:** GitHub Pages via a dedicated `gh-pages` branch (keeps CV files separate from the portfolio showcase on `main`)
- **No build step** — Tailwind via CDN (v4 play CDN), Three.js via CDN import map
- **Performance targets:** First Contentful Paint < 2s, smooth 60fps scroll on mid-range mobile

---

## Out of Scope

- CMS or editable content — content is hardcoded in `index.html`
- Contact form backend — CTA links to email/social directly
- Analytics
- Dark/light mode toggle
- Internationalisation
