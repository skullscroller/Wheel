[README.md](https://github.com/user-attachments/files/28172485/README.md)
# Wheel of Life — 3D Viewer Prototype

## Overview

A self-contained, web-based interactive 3D viewer for the Tyre/Wheel model, built as a single HTML file that requires no server, build step, or internet connection beyond loading Three.js from a CDN.

---

## Tools Used

| Tool | Purpose |
|------|---------|
| **Three.js r160** | 3D rendering engine (WebGL) |
| **Three.js GLTFLoader** | Parses the `.glb` model file |
| **Three.js OrbitControls** | Interactive orbit / pan / zoom |
| **Three.js DRACOLoader** | (available, for compressed meshes) |
| HTML/CSS/JS | UI panels, controls, animations |

No build tool, framework, or package manager required.

---

## How the Wheel Was Imported

The provided `Tyre.glb` file is **base64-encoded and embedded directly** into the HTML file as a JavaScript string. At runtime:

1. The base64 string is decoded to an `ArrayBuffer` via `atob()`.
2. `GLTFLoader.parse()` loads the geometry, materials, and textures directly from memory — no HTTP request needed.
3. A `Box3` bounding box is computed to auto-center and normalize the model to a consistent scale (~2 units).
4. The model is positioned so it sits naturally above the floor grid.

This approach makes the file fully portable (single `.html` file, no assets folder).

---

## How Projection Views Were Implemented

Each named view (Top, Bottom, Left, Right, Front, Back) is defined as a data object:

```js
const VIEWS = {
  top:    { pos: [0, 6, 0],  target: [0,0,0], label: 'Top View',    ortho: true },
  right:  { pos: [6, 0, 0],  target: [0,0,0], label: 'Right View',  ortho: true },
  // ...
};
```

Clicking a view button triggers a **smooth animated transition**:
- Camera position and orbit target are interpolated over 600ms using a quadratic ease-in-out curve.
- For orthographic-style views, `OrbitControls` is temporarily disabled to lock the projection angle.
- A subtle grid overlay fades in during projection views for spatial reference.
- The view label appears briefly on screen during transitions.

The viewer uses a **PerspectiveCamera** throughout — true orthographic projection could be added in a production version (see improvements below).

---

## Features

- **Interactive orbit** — left-click drag to rotate
- **Pan** — right-click drag
- **Zoom** — scroll wheel
- **Auto-rotate** — toggle checkbox
- **Wireframe mode** — toggle checkbox
- **Lighting controls** — key light intensity + environment intensity sliders
- **Floor grid** — toggle visibility
- **Reset Camera** — returns to default perspective (`R` key shortcut also works)
- **6 projection shortcuts** — Top, Bottom, Left, Right, Front, Back
- Fully self-contained single HTML file (~14 MB with embedded model)

---

## How to Open

Simply open `wheel_of_life_viewer.html` in any modern browser (Chrome, Firefox, Edge, Safari). No server required.

```bash
open wheel_of_life_viewer.html        # macOS
start wheel_of_life_viewer.html       # Windows
xdg-open wheel_of_life_viewer.html   # Linux
```

For web hosting, upload the single HTML file to any static host (GitHub Pages, Netlify, Vercel, S3).

---

## Improvements for a Production Version

### 1. True Orthographic Camera
Switch to `THREE.OrthographicCamera` for projection views to eliminate perspective distortion — critical for engineering/CAD-style previews.

### 2. Separate Asset Hosting
Rather than embedding the GLB as base64 (which inflates file size ~33% and blocks the main thread during decode), serve the `.glb` as a separate static asset and stream it with `GLTFLoader.load()`. Use a loading progress bar tied to the actual XHR progress event.

### 3. DRACO / Meshopt Compression
Apply mesh compression to the GLB (via `gltfpack` or Blender's DRACO export) to reduce the 11 MB file to ~1–2 MB for faster web loads.

### 4. Environment Map (IBL)
Add a high-quality HDR environment map (e.g., via `RGBELoader`) for physically accurate reflections on the tyre's rubber and rim materials — makes a significant visual difference for automotive parts.

### 5. Measurement / Annotation Overlay
Add clickable hotspots or dimension lines (width, diameter, sidewall height) for product presentation use cases.

### 6. Screenshot / Export Button
Use `renderer.domElement.toDataURL()` to let users download a PNG snapshot of the current view — useful for documentation or quote requests.

### 7. Multi-model Support
Extend to accept a `?model=url` query parameter or a drag-and-drop file input so the viewer can be reused for any GLB/GLTF asset.

### 8. Responsive / Mobile
Add touch gesture support improvements and a collapsible UI panel for small screens.

### 9. Post-processing
Add `THREE.EffectComposer` with SSAO (ambient occlusion) and a subtle bloom pass for a more cinematic product-render look.

### 10. Accessibility
Add ARIA labels, keyboard navigation for all controls, and a reduced-motion media query that disables camera animation.

---

## File Structure

```
wheel_of_life_viewer.html   ← Complete self-contained viewer (GLB embedded)
README.md                   ← This file
Tyre.glb                    ← Original source model
```
