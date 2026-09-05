# Riso Paper Fold

A single-file WebGL2 toy that prints an image as a risograph sheet, folds it in 3D, and scans the result.

Open `index.html` in any browser that supports WebGL2 — no build step, no dependencies.

## Controls

| | |
|---|---|
| drag | tilt the sheet |
| scroll | zoom |
| hover | uncoil the folds |
| double-click | reset the axis |
| `S` | toggle sunlight |
| `A` | toggle the axis gizmo |
| `R` / `Enter` | randomize |
| drop / paste an image | print it onto the sheet |

## How it renders

Four passes, all in `index.html`:

1. **Shadow** — the deformed mesh from the light's point of view, into a depth target.
2. **Scene** — arc-length-preserving folds on a 272×272 grid. Material past each crease plane rolls onto a cylinder of radius `w/θ` and then continues straight, so the sheet bends instead of stretching. Writes ink coverage and shading.
3. **Layers** — additive pass counting how many plies stack over each pixel, so overlaps saturate and translucency reads correctly.
4. **Print** — screening, ink bleed along paper fibre, misregistration, grain, and the dappled-sunlight pass.

Ink separation is a least-squares subtractive solve: colour is `paper × Π mix(1, ink_i, c_i)` per Beer–Lambert, so `c` comes from `(AᵀA)⁻¹Aᵀ log(image)` with the ink log-colours as columns.

## Sunlight

Optional dappled light. A warping noise field is thresholded into leaf gaps, then convolved with a 12-tap golden-angle disc — the disc is the sun's penumbra, and that blur is what rounds each gap into the soft overlapping blobs light makes on a wall. It falls on the sheet only; the backdrop stays clean.

The animation lives entirely in the print pass, so its frames redraw only a fullscreen quad and skip the mesh deformation. The heavy passes run only on a real state change.

## Parameters

Every slider is also a URL parameter, e.g. `index.html?mode=sculpt&curl=0.8&sun=1`.
