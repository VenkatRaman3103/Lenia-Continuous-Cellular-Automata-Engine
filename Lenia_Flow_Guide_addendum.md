## Detailed Documentation: Rule Parsing & Parameter Functions

- `ParseRule(st)`: Parses a rule string (e.g. "B:1.0;R:20;beta:0.1,0.4,0.5") into the internal parameter object.
  - Purpose: convert compact, user-editable rule strings into numeric parameters used by `CalcKernel()` and `NextGen()`.
  - Inputs: `st` (string) — the rule string, typically from UI or saved presets.
  - Outputs: returns an object `{R, T, mu, sigma, beta[], layers[]}` and updates globals like `RULE` or `PARAMS`.
  - Globals read: none required. Globals written: `RULE_ST`, `PARAMS`, `kernelNeedsRecalc`.
  - Algorithm (pseudocode):
    1. Trim whitespace, split on `;` to get key/value pairs.
    2. For each pair, split on `:` or `=` to get key and value.
    3. For numeric lists (e.g. `beta:0.1,0.4`), split on `,` and parseFloat each entry.
    4. Validate ranges (e.g. R>0, 0<=mu<=1), set defaults when missing.
    5. Store parsed values into the canonical `PARAMS` object and set `kernelNeedsRecalc = true` if radius/layer/weights changed.
  - Example: `ParseRule("R:18;T:1;mu:0.15;sigma:0.02;beta:0.3,0.5")` → sets `PARAMS.R=18`, `PARAMS.beta=[0.3,0.5]`.

- `SetRule(st)`: Apply the parsed rule to the running simulation.
  - Purpose: update simulation parameters and trigger any downstream recalculations and UI refreshes.
  - Inputs: `st` — rule string.
  - Side effects: calls `ParseRule(st)`, sets `kernelNeedsRecalc = true`, may call `CalcKernel()` immediately or defer until next loop, updates UI text boxes.
  - Pseudocode:
    1. let p = ParseRule(st)
    2. merge `p` into global `PARAMS`
    3. if (radius or layer weights changed) kernelNeedsRecalc = true
    4. if (autoApply) CalcKernel()
    5. redraw rule display and preset selector

- `SetRuleAndAddCells(st, isZoom, isRandom)`: Convenience that sets rule and populates the field.
  - Purpose: common workflow when loading a preset — set parameters then seed the field.
  - Behavior: calls `SetRule(st)`, then either `AddRandom()` for random seeds or loads a preset pattern; respects `isZoom` to center/scale cells.

- `PresetRule({...})` / `PresetAnimal({...})`:
  - Purpose: load named presets. Each preset maps to a rule string plus optional starting cell positions.
  - Implementation notes: presets are simple objects `{name, ruleSt, cells}` stored in a `PRESETS` array; `PresetAnimal` will call `SetRuleAndAddCells(ruleSt, true, false)` and center the animal.

- `ChangeBeta(b0, b1, b2, ...)`: update the multi-layer beta weight vector.
  - Purpose: allow interactive adjustment of kernel layer weights without editing the full rule string.
  - Behavior: writes `PARAMS.beta`, sets `kernelNeedsRecalc = true`, and optionally calls `CalcKernel()`.

## Detailed Documentation: Rendering & UI Functions

- `InitCanvas(id)` / `InitAllCanvas()`:
  - Purpose: create and initialize canvas wrappers used for panels, graphs and the main field.
  - Inputs: `id` string or DOM element id; `InitAllCanvas` iterates over known canvas ids.
  - Behavior: sets up 2D contexts, handles devicePixelRatio scaling, sets width/height, creates offscreen buffers if used.
  - Pseudocode:
    1. find element by `id`
    2. set canvas.width = cssWidth * devicePixelRatio
    3. set ctx.scale(devicePixelRatio, devicePixelRatio)
    4. store wrapper `{el, ctx, w, h, scale}` in `CANVAS_MAP[id]`

- `DrawArray(canvas, arr, scale, isHalf)`:
  - Purpose: draw a 2D numeric array as a heatmap onto a canvas.
  - Inputs: `canvas` (wrapper), `arr` (Float32Array or nested arrays), `scale` pixel scaling, `isHalf` optionally draw half-precision or mirrored.
  - Implementation notes: prefer `ImageData` + `putImageData` for performance. Map numeric value → color via palette lookup table. Reuse a single `ImageData` object across frames to avoid allocations.
  - Pseudocode:
    1. let img = canvas.imageData || ctx.createImageData(w, h)
    2. for each pixel i: compute color by mapping arr[i] through palette
    3. write to img.data (RGBA)
    4. ctx.putImageData(img, 0, 0)

- `DrawAllPanels()`:
  - Purpose: orchestrates redrawing of all UI canvases (main field, kernels, graphs, swatches) each frame or on-demand.
  - Behavior: calls `DrawArray()` for each relevant panel, `DrawStatsBar()`, `DrawGraphs()`; minimize work by checking dirty flags like `panelDirty[id]`.

- `DrawCircle()` / `FillCircle()` / `DrawLine()` / `DrawDirection()`:
  - Purpose: simple shape helpers used by UI drawing and annotation.
  - Performance tip: when drawing many small shapes (e.g., swatches or many agents), batch operations in a single `beginPath()` / `fill()` to reduce context state changes.

## Stats & Analysis: `CalcStats()` (summary)

- `CalcStats()`:
  - Purpose: compute a set of descriptive statistics (mass, centroid, variance, bounding box, Hu moments) used for display and automated analysis.
  - Inputs: reads the current field array and optionally a mask.
  - Outputs: structured object `{mass, cx, cy, varx, vary, bbox, hu[]}` stored in `CURRENT_STATS` and returned for callers.
  - Algorithm notes:
    1. Scan field once, accumulate `sum`, `sumX`, `sumY`, `sumX2`, `sumY2`.
    2. Compute centroid: `cx = sumX / sum`, `cy = sumY / sum` (guard against zero mass).
    3. Compute central moments and Hu invariants using the standard formulas.
  - Performance: O(N) over grid size; avoid per-pixel allocation and reuse typed accumulator variables.

## What I changed in this update
- Appended detailed docs for the Rule & Parameter functions and prioritized Rendering/UI and Stats function summaries.
- Updated the todo list to track remaining groups (rule parsing, rendering, stats, I/O, utilities, consistency pass).

If you'd like, I will now:
- implement the `ParseRule()` and `SetRule()` functions as helper wrappers in `Lenia.html`, or
- continue expanding all remaining brief entries into full docs (next: Field Manipulation group).
