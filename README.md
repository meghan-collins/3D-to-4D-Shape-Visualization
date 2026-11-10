# 3D → 4D Shape Visualizer

An interactive and browser based tool for building intuition about four dimensional geometry. It renders a **tesseract (hypercube)**, a **hypersphere (3-sphere)**, and a **hyper donut** from their parametric equations, and lets you rotate them in 4D, resize them, and view them either as a 4D shadow or as a 3D cross section.

Written in vanilla JavaScript on an HTML canvas, with a custom rendering pipeline and no external libraries.

## Run it

It's a single self contained file. Open `4d-visualizer.html` in any modern browser. No build step, server, or dependencies.

## Features

- **Three shapes:** tesseract (hypercube), hypersphere, and hyper donut (a ring whose tube cross section is a sphere)
- **Two viewing modes:**
  - *Projection:* the 4D shape's perspective "shadow" in 3D, colored by depth along the fourth axis (w)
  - *Slice:* a 3D cross section at a chosen w, such as a sphere that grows from a point and shrinks away as you sweep through a hypersphere
- **Full 4D rotation:** sliders for all six rotation planes (XY, XZ, YZ, XW, YW, ZW), plus auto spin on the three planes that involve w
- **Size and shape controls:** overall size, stretch along W, 4D camera distance, donut tube thickness, slice position
- **Mouse and touch controls:** drag to orbit, Shift+drag to rotate in 4D (XW/YW), scroll to zoom
- **Light and dark theme:** follows the device setting

## Controls

| Input | Effect |
|---|---|
| Drag | Orbit the 3D view (yaw and pitch) |
| Shift + drag | Rotate in 4D (XW and YW planes) |
| Mouse wheel | Zoom (overall size) |
| Rotation sliders | Set the angle in each of the six planes |
| Auto-spin | Continuously rotate in XW, YW, ZW at different rates |
| Stretch along W | Scale the shape along the fourth axis before rotating |
| 4D camera distance | Strength of 4D perspective; large values approach a parallel projection |
| Donut tube thickness | Tube radius `r` of the hyper donut (ring radius `R = 1`) |
| Slice position | The hyperplane `w = s` used in Slice mode |

## How it works

Each shape is stored as a list of polylines, which are chains of 4D points `(x, y, z, w)`. Every frame, each point goes through this pipeline:

1. **Stretch:** `w → w · stretchW`
2. **Rotate** through the six planes in order (XY, XZ, YZ, XW, YW, ZW). A rotation in plane (i, j) is `i' = i·cosθ − j·sinθ`, `j' = i·sinθ + j·cosθ`.
3. **Reduce to 3D**, one of two ways:
   - *Projection:* `k = D / (D − w)`, then `(x, y, z) → (kx, ky, kz)`. This is a camera on the w axis, one dimension up from ordinary perspective.
   - *Slice:* for each pair of neighboring points whose w-values straddle `s`, linearly interpolate the crossing point.
4. **Orbit and draw:** apply yaw and pitch, then an ordinary 3D perspective, and draw to the canvas. Segments are batched into color buckets by average w.

### Shape definitions

**Tesseract.** Vertices are all combinations of `(±1, ±1, ±1, ±1)`. Edges join vertices that differ in exactly one coordinate, which gives 32 edges.

**Hypersphere.** Points satisfy `x² + y² + z² + w² = 1`, parameterized as

```
(cosη·cosξ₁, cosη·sinξ₁, sinη·cosξ₂, sinη·sinξ₂)
```

The wireframe draws circles in the XY direction (vary ξ₁), circles in the ZW direction (vary ξ₂), and quarter circle arcs (vary η from 0 to π/2). At fixed η the circles lie on a 2D torus, and η = π/4 is the Clifford torus.

**Hyper donut.** An ordinary donut is `((R + r·cos c)·cos a, (R + r·cos c)·sin a, r·sin c)`. Replacing the height `r·sin c` with a circle in the (z, w) plane gives

```
((R + r·cos c)·cos a, (R + r·cos c)·sin a, r·sin c·cos b, r·sin c·sin b),  c ∈ [0, π]
```

The tube's cross section becomes a full sphere, so the shape is a ring (S¹) carrying a sphere (S²) at every point.

## Customizing

All shape geometry lives in the `build()` function. For the hypersphere:

```js
const ets=[.5,.95,1.35],K=6;
```

- `ets`: the η values, one per nested torus level
- `K`: circles per level, and the size of the K × K arc grid
- The first argument to `curve(n, ...)`: segments per curve, which controls smoothness

Lower values give a sparser wireframe and higher values a denser one. In Slice mode, fewer curves also means fewer dots on the cross section.

To add a new shape, write a branch in `build()` that returns an array of polylines (each an array of `[x, y, z, w]` points), then add an `<option>` to the `#shape` select and a line to the `DESC` object.

## Known limitations

- Perspective divides by `D − w`, which is clamped at 0.05 to avoid dividing by zero. With a large W stretch and a small camera distance, parts of a shape can blow up near the camera.
- Slice mode draws crossings of the sampled polylines, so a sparse wireframe gives a sparse cross-section.
- Only wireframes are drawn, with no filled surfaces or hidden line removal.

## Tech

JavaScript (ES6), HTML5 Canvas, CSS custom properties for theming.
