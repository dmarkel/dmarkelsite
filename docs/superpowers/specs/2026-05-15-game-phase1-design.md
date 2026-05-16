# Game Phase 1 Design — Interactive Resume/Portfolio

**Date:** 2026-05-15  
**Status:** Approved

## Overview

A browser-based 2D platformer game that showcases David Markel's accomplishments and resume. The player controls a pixel-art character (David) who walks through themed city scenes. Each background zone will eventually contain interactive elements (NPCs, signs, dialogue) tied to career milestones. Phase 1 scope: scrolling Houston background with parallax, character movement, and sprite animation.

---

## Technology

- **Single HTML file** (`index.html`) with vanilla JavaScript — no build tools, no dependencies
- **HTML5 Canvas** with a `requestAnimationFrame` game loop (~60fps)
- Deployable as-is to GitHub Pages

---

## Assets

### Background — `img/1 Houston.png`
- Dimensions: 1536 × 1024px, RGB
- Represents Houston, TX — pixel-art city scene with sky, skyline, trees/bridge, fence, and sidewalk

### Sprite Sheet — `img/dmarkel_sprite_sheet.png`
- Dimensions: 1536 × 1024px, RGBA (transparent background)
- Grid: 5 columns × 3 rows
- Frame size: ~307 × 341px per cell

| Row | Animation | Frames used | Columns |
|-----|-----------|-------------|---------|
| 0   | Walk      | 5           | 0–4     |
| 1   | Jump      | 3           | 0–2     |
| 2   | Crouch    | 3           | 0–2     |

Idle state uses Walk frame 0 (no movement).

---

## Parallax Layer System

The single background image is divided into 4 depth bands. Each band is drawn to canvas at an x-offset proportional to `cameraX * speed`, then tiled horizontally so the world feels infinite (or until the next zone begins).

| Layer | Region (% of image height) | Scroll speed multiplier | Contents |
|-------|----------------------------|------------------------|----------|
| 1 — Sky | 0–38% | 0.05 | Sky gradient, clouds |
| 2 — Skyline | 30–58% | 0.20 | City buildings |
| 3 — Midground | 55–77% | 0.50 | Trees, bridge, water |
| 4 — Foreground | 77–100% | 0.90 | Fence, lamp, sidewalk |

Each layer is drawn with `ctx.drawImage(img, srcX, srcY, srcW, srcH, destX, destY, destW, destH)` where `srcX` shifts by the layer's parallax offset. Layers tile horizontally.

---

## Character Physics & Controls

### Ground plane
The sidewalk surface sits at approximately **83% of canvas height** (scaled from the image). The character's feet rest on this line when grounded.

### Physics values (tunable)
- `gravity`: 0.5 px/frame²
- `jumpVelocity`: −12 px/frame (upward)
- `walkSpeed`: 4 px/frame
- `animFPS`: 8 frames/sec for walk cycle

### Input → State mapping

| Key | State | Animation |
|-----|-------|-----------|
| `←` | Walking left | Walk row, 5-frame cycle, flipped horizontally |
| `→` | Walking right | Walk row, 5-frame cycle |
| `↑` | Jumping (one-shot, grounded only) | Jump row, plays through 3 frames once |
| `↓` | Crouching (hold) | Crouch row, 3-frame cycle |
| None | Idle | Walk row, frame 0, static |

No double-jump in phase 1.

---

## Camera

- Camera tracks the character horizontally, keeping them roughly centered on screen
- Camera X is clamped so it never shows before the start of the world (x < 0)
- Future: clamp at world end, or transition to next zone

---

## Extensible World System

The world is defined as an array of zone configs. Phase 1 has one zone:

```js
const WORLD = [
  {
    image: houstonImg,         // HTMLImageElement
    width: 4096,               // how far this zone extends (pixels in world space)
    groundY: 0.83,             // ground as fraction of canvas height
    layers: [
      { srcYStart: 0,    srcYEnd: 0.38, speed: 0.05 },
      { srcYStart: 0.30, srcYEnd: 0.58, speed: 0.20 },
      { srcYStart: 0.55, srcYEnd: 0.77, speed: 0.50 },
      { srcYStart: 0.77, srcYEnd: 1.00, speed: 0.90 },
    ]
  }
];
```

To add a new background: add a new object to `WORLD`. The camera transitions to the next zone when `cameraX` exceeds the current zone's `width`.

---

## File Structure

```
index.html              ← game entry point (single file)
img/
  1 Houston.png
  dmarkel_sprite_sheet.png
docs/superpowers/specs/ ← design docs
```

---

## Out of Scope for Phase 1

- Platforms / obstacles / collision boxes
- Dialogue / NPC interaction
- Sound / music
- Mobile / touch controls
- Additional background zones
- Resume/accomplishment content
