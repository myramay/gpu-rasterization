# GPU Rasterization Pipeline Simulator

An interactive, browser-based software renderer that visualizes every stage of the GPU rasterization pipeline — built with pure Canvas 2D, no WebGL, no dependencies.

![Pipeline Simulator](https://img.shields.io/badge/renderer-Canvas%202D-blue) ![No Dependencies](https://img.shields.io/badge/dependencies-none-green) ![Single File](https://img.shields.io/badge/build-single%20HTML-orange)

## Live Demo

Open `index.html` in any modern browser — no server or build step required.

## Features

### Pipeline Stages
Step through each stage of the rasterization pipeline using the stage buttons at the bottom:

| Stage | Name | What you see |
|-------|------|-------------|
| 0 | Vertex Input | Raw vertex buffer — dots + wireframe before any transform |
| 1 | Vertex Shader | MVP-transformed mesh with optional normal vectors |
| 2 | S-H Clip | Sutherland-Hodgman frustum clipping — original (dim) vs. clipped (bright) triangles |
| 3 | Rasterize | Perspective-correct barycentric color interpolation (unlit) |
| 4 | Z-Buffer | Grayscale depth buffer visualization |
| 5 | Fragment Out | Final shaded output |

### Shading Models
- **Flat** — one Blinn-Phong calculation per triangle at the face centroid
- **Gouraud** — lighting computed per vertex, interpolated across fragments
- **Phong** — normals interpolated per fragment, lighting computed per pixel

### Meshes
- Cube, Sphere, Torus (switchable at runtime)

### Display Toggles
- **Wireframe** — edge overlay on any pipeline stage
- **Overdraw Heatmap** — heat-map visualization of how many times each pixel was drawn (black → blue → cyan → green → red)
- **Normals** — per-vertex world-space normal vectors
- **Auto-Rotate** — continuous slow spin

### Stage Inspector (right panel)
- **Pseudocode** — GLSL-like pseudocode for the active stage, updates when you switch shaders
- **Pipeline Statistics** — vertex count, triangle count before/after clipping, max overdraw
- **Per-Pixel Inspector** — hover over the canvas to see: screen coords, NDC coords, triangle index, barycentric coordinates (λ₀ λ₁ λ₂), depth value, world position, world normal, fragment color + hex swatch, overdraw count
- **Depth Buffer** — live 164×164 thumbnail with depth range

## Controls

| Control | Action |
|---------|--------|
| Left drag on canvas | Rotate the mesh |
| Right-click on canvas | Place the light source |
| Rotation sliders | Fine-tune X / Y / Z rotation |
| Light X / Y / Z sliders | Move the point light |
| Ambient / Specular sliders | Adjust Blinn-Phong lighting parameters |

## Implementation Details

### Software Renderer
Everything runs on the CPU — no GPU acceleration. The renderer implements:

- **Row-major 4×4 matrix math** — model, view, perspective, lookAt
- **Vertex shader** — MVP transform, world-space position + normal output
- **Sutherland-Hodgman clipping** — 6 frustum planes in homogeneous clip space with perspective-correct attribute interpolation at intersection points
- **Edge function rasterizer** — incremental scanline traversal (additions instead of per-pixel multiplies) with perspective-correct barycentric coordinates
- **Z-buffer** — per-pixel depth test with Infinity initialization
- **Blinn-Phong lighting** — ambient + diffuse + specular with half-vector

### Perspective-Correct Interpolation
Attributes (color, normals, world position) are interpolated using the standard GPU formula:

```
Σ = λ₀/w₀ + λ₁/w₁ + λ₂/w₂
corrected_λᵢ = (λᵢ / wᵢ) / Σ
```

This prevents the affine texture distortion visible in early 3D games.

## File Structure

```
gpu-rasterization/
└── index.html   # Entire application — HTML + CSS + JS, ~1100 lines
```

## Browser Support

Any modern browser with Canvas 2D support (Chrome, Firefox, Safari, Edge).
