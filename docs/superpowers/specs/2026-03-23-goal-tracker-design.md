# Goal Tracker — Design Spec
**Date:** 2026-03-23  
**Status:** Approved

---

## Overview

A spatial, 3D goal tracker rendered in a full-screen Three.js canvas. Goals appear as frosted-glass cards floating in a deep-space void; tasks are luminous dot satellites orbiting their parent goal. The entire scene is interactive — draggable, zoomable, and click-navigable.

---

## Visual Theme

**Color palette — Everforest Dark:**
| Role | Token | Hex |
|------|-------|-----|
| Background | `--bg` | `#2d353b` |
| Surface (cards) | `--bg1` | `#343f44` |
| Border/dim | `--bg3` | `#475258` |
| Foreground | `--fg` | `#d3c6aa` |
| Accent (teal) | `--accent` | `#7fbbb3` |
| Green (done) | `--green` | `#a7c080` |
| Yellow (in-progress) | `--yellow` | `#dbbc7f` |
| Red (blocked) | `--red` | `#e67e80` |
| Purple (tag) | `--purple` | `#d699b6` |
| Teal2 | `--teal2` | `#83c092` |

**Typography:**
- Display/headings: `"JetBrains Mono"` (monospace, loaded from Google Fonts bunny.io proxy)
- Body: `"JetBrains Mono"` at reduced weight

---

## Scene Architecture

### Background
- Full-screen `<canvas>` via `THREE.WebGLRenderer`
- Starfield: ~600 `THREE.Points` scattered in a large sphere, slow auto-rotation
- Ambient fog: `THREE.FogExp2` with bg color for depth falloff

### Goal Cards
- Rendered as `CSS2DObject` overlaid on Three.js scene (HTML cards positioned by 3D coords)
- Each card: title, description, progress bar, status badge
- Frosted glass style: `backdrop-filter: blur`, semi-transparent bg `#343f44cc`, `1px` teal border
- Cards placed at hand-crafted X/Y/Z positions, staggered in depth
- Hover: border brightens, card scales up slightly (`transform: scale(1.04)`)
- Click: camera GSAP-tweens to focus on that goal

### Task Dots
- `THREE.Mesh` with `THREE.SphereGeometry(0.08)` + emissive `MeshStandardMaterial`
- Clustered loosely around parent goal's 3D position (randomised offset ~1.5–3 units)
- Color by status:
  - `todo` → `#475258` (dim grey-blue)
  - `in-progress` → `#dbbc7f` (amber, brighter emissive)
  - `done` → `#a7c080` (green)
- Slow individual wobble animation (each dot has a random phase offset)
- Hover: scales to 1.6x, tooltip appears with task name + status

### Interaction
| Action | Behaviour |
|--------|-----------|
| Drag | OrbitControls — rotate scene |
| Scroll | OrbitControls — zoom |
| Hover dot | Tooltip: task name + status chip |
| Click goal card | Camera flies to goal (GSAP tween on camera position + lookAt) |
| Click anywhere else | Camera eases back to default overview position |

---

## Data Model (hardcoded sample)

```js
const GOALS = [
  {
    id: 'g1', title: 'Ship MVP', status: 'in-progress',
    description: 'Core product, first paying users',
    position: { x: -3, y: 1.5, z: -1 },
    tasks: [
      { id: 't1', label: 'Auth flow', status: 'done' },
      { id: 't2', label: 'Dashboard', status: 'in-progress' },
      { id: 't3', label: 'Billing', status: 'todo' },
      { id: 't4', label: 'Onboarding', status: 'todo' },
      { id: 't5', label: 'Deploy to prod', status: 'todo' },
    ]
  },
  {
    id: 'g2', title: 'Health Ritual', status: 'in-progress',
    description: 'Daily movement & sleep',
    position: { x: 2.5, y: -0.5, z: 0.5 },
    tasks: [
      { id: 't6', label: 'Morning run', status: 'in-progress' },
      { id: 't7', label: 'Sleep by 23:00', status: 'todo' },
      { id: 't8', label: 'No phone after 22:00', status: 'done' },
    ]
  },
  {
    id: 'g3', title: 'Learn Rust', status: 'todo',
    description: 'Finish the Rust book + one project',
    position: { x: 0.5, y: 2.5, z: -3 },
    tasks: [
      { id: 't9',  label: 'Ch1–5 reading', status: 'done' },
      { id: 't10', label: 'Ch6–10 reading', status: 'in-progress' },
      { id: 't11', label: 'CLI project', status: 'todo' },
      { id: 't12', label: 'Async chapter', status: 'todo' },
    ]
  }
]
```

---

## File Structure

```
goal-tracker/
├── index.html          # Single file — all HTML/CSS/JS inline
└── docs/
    └── superpowers/
        └── specs/
            └── 2026-03-23-goal-tracker-design.md
```

---

## Tech Stack (prototype phase)

- **Three.js r165** via CDN (importmap)
- **Three.js CSS2DRenderer** for goal card overlays
- **OrbitControls** for drag/zoom
- **GSAP** for camera tweens
- Google Fonts (via Bunny.io) for JetBrains Mono
- Zero build step — open `index.html` directly in browser

---

## Out of Scope (prototype)

- CRUD for goals/tasks (hardcoded data only)
- Persistence (localStorage / DB)
- Auth / multi-user
- Mobile touch handling
