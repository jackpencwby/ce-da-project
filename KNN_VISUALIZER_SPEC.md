# KNN Interactive Algorithm Visualizer — Implementation Spec

> **For Claude Code.** Read this whole file before writing any code. Build exactly what is described here.
> When something is unspecified, choose the option that is simplest for a beginner to understand and note the decision in a code comment.

---

## 0. TL;DR

Build a **single-file** web app (`index.html` with embedded CSS + JS, **no external JS libraries, no build step**) that teaches the **K-Nearest Neighbors (KNN)** classifier step by step, with a focus on the **math** behind each step, for complete beginners. All UI text is in **English**. It must run by simply opening the file in a browser and must deploy to **GitHub Pages** as-is.

Style reference (same family of visualizers): https://github.com/LLM-CE-KMITL/learning — e.g. `ml-viz/linear.html`. Borrow the feel: dark theme, numbered panels (`01 Data Canvas`, `02 ...`), collapsible panels, live metric chips, sliders that show their current value in bold.

---

## 1. The core design problem (read carefully)

The assignment asks for **Step-by-step Execution**, **Play until Convergence**, a **Model State** panel (matrices / weight vectors) and a **Loss Graph that decreases per step/epoch**. KNN, however, is a *lazy, non-parametric* algorithm: it has **no weights and no gradient descent**. Its "training" is just **storing the data**.

We must not fake a gradient-descent curve. Instead we use an honest framing that maps every required element onto what KNN really does:

| Required element | Honest KNN interpretation used in this app |
|---|---|
| "Training" / model update | **Memorization**: each epoch the model stores the next `B` training points into its **Memory** |
| Model parameters / weight vector | The **Memory table** (stored points `X`, labels `y`) — *the data IS the model* |
| One step | One **sub-step** of the KNN prediction pipeline (memorize → distance → sort & select K → vote → evaluate) |
| Loss | **Validation cross-entropy (log-loss)** of the current memory-based classifier on a held-out validation set |
| Convergence | All training points memorized (memory complete), **or** optional early stop when loss stops improving |
| Decision boundary on canvas | Recomputed from the current memory after every evaluate step — visibly sharpens as memory grows |

The loss **generally decreases** as more examples are memorized, but small bumps are normal. The UI must say so (one sentence under the loss chart) rather than hiding it.

The app must also explicitly teach this point in plain English somewhere visible, e.g. in the Model State panel:

> "KNN has no weights to learn. Its 'model' is simply the memory of training examples. Training = storing points. Prediction = finding the K closest stored points and letting them vote."

---

## 2. Deliverables & constraints

1. `index.html` — the entire app (HTML + `<style>` + `<script>`), at the repo root so GitHub Pages serves it at `https://<user>.github.io/<repo>/`.
2. `README.md` — short project description, screenshot placeholder, how to run locally, how to deploy, group info.

Constraints:
- **Zero runtime dependencies.** No CDN JS (no Chart.js, D3, KaTeX, React…). Draw charts and the canvas with the Canvas 2D API; render math with HTML/CSS (`<sup>`, `<sub>`, Unicode `√ Σ ≤ ŷ`).
- Google Fonts (`Inter`, `JetBrains Mono`) are allowed **only** with full fallback stacks (`system-ui, -apple-system, Segoe UI, Roboto, sans-serif` / `ui-monospace, Menlo, Consolas, monospace`). The app must look fine offline.
- No `localStorage` requirement; the app works without persistence.
- Vanilla ES2020, no modules split across files. Organize the single `<script>` into clearly commented sections (see §11).
- Must work in latest Chrome, Edge, Firefox, Safari. HiDPI-aware canvases (`devicePixelRatio`).

---

## 3. Page layout

```
┌──────────────────────────────────────────────────────────────────────┐
│ HEADER: title + one-paragraph principle + 3 "how it works" chips     │
├───────────────────────────────────┬──────────────────────────────────┤
│ LEFT COLUMN (own vertical scroll) │ RIGHT COLUMN (own vertical scroll)│
│ ┌───────────────────────────────┐ │ ┌──────────────────────────────┐ │
│ │ 01 Data Canvas                │ │ │ 03 Model State               │ │
│ └───────────────────────────────┘ │ └──────────────────────────────┘ │
│ ┌───────────────────────────────┐ │ ┌──────────────────────────────┐ │
│ │ 02 Hyperparameters & Controls │ │ │ 04 Loss Graph                │ │
│ └───────────────────────────────┘ │ └──────────────────────────────┘ │
├───────────────────────────────────┴──────────────────────────────────┤
│ FOOTER: group name, members (ID + name), CE KMITL                    │
└──────────────────────────────────────────────────────────────────────┘
```

- Use CSS Grid: `body` = `grid-template-rows: auto 1fr auto; height: 100vh`.
- The two columns are **floating cards**: rounded corners (14–16px), subtle border, soft shadow, gap between them and from edges (~16px). Each column has `overflow-y: auto` with its **own independent styled scrollbar** (thin, dark track, accent thumb). The page body itself does not scroll on desktop.
- Column widths: left ≈ 52%, right ≈ 48% (`minmax` so neither collapses).
- **Responsive**: below 1024px wide, stack columns vertically (left then right), disable the inner-scroll behavior, and let the whole page scroll.
- Each panel has a numbered title bar (`01 Data Canvas — add, erase & generate`), a small status chip on the right (e.g. `124 points`, `epoch 12 / 30`), and a collapse caret `⌄`.

### 3.1 Theme

- Background `#07090c`, column surface `#0e1116`, panel surface `#141922`, border `#1f2733`, text `#e6edf3`, muted text `#8b98a9`, accent `#7c9cff` (buttons/focus), success `#3ddc97`, danger `#ff6b6b`, warning `#ffc857`.
- Class colors (max 4 classes): **A** `#22d3ee` (cyan), **B** `#f472b6` (pink), **C** `#fbbf24` (amber), **D** `#a3e635` (lime). Class is also encoded by **letter labels in tables** so color is never the only cue.
- Professional, calm, no neon overload. Subtle header gradient allowed.

### 3.2 Header (exact copy, may be lightly edited)

- Kicker: `Interactive ML Demo`
- Title: **K-Nearest Neighbors (KNN) Visualizer · learning by remembering**
- Principle (1–2 sentences): *"KNN classifies a new point by looking at the K closest points it has stored and letting them vote. There are no weights to train: the stored data itself is the model."*
- Three chips: `① Measure distance` → `② Pick the K nearest` → `③ Majority vote`

### 3.3 Footer

Comfortable height (~64–88px), muted text, centered or two-column on desktop:

- `Group: ManchesterLiver`
- Members (Student ID — Name):
  - 66010064 — กาลัญญ จินดา
  - 66010089 — ขจรยศ จูเลิศ
  - 66010324 — ธนกฤต สุไชยชิต
  - 66011463 — รัตนพล มาสูตร
  - 66011520 — อาณา สุขอุดม
  - 66011471 — วรพัฒน์ คำศรี
- `Computer Engineering, KMITL (CE KMITL)`

> **TODO (owner):** the page must be in English. Replace the Thai names above with the members' official English names before final deployment. Keep them in a single `MEMBERS` array at the top of the script so they are easy to edit.

---

## 4. Panel 01 — Data Canvas (left, top)

### 4.1 Coordinate system
- Data domain: `x₁ ∈ [0, 10]`, `x₂ ∈ [0, 10]` (friendly numbers for hand-checking math). Show light grid lines + axis ticks at every 1 unit, labels at 0, 2, 4, 6, 8, 10.
- Canvas is square, fills the panel width (max ≈ 600 CSS px), HiDPI-scaled.
- All displayed coordinates use 2 decimals.

### 4.2 Point types (visual encoding)
| Kind | Shape | Before memorized | After memorized |
|---|---|---|---|
| Training point | filled circle r≈5 | **dim**: 30% opacity, grey outline | full class color + white 1px outline + brief "pop" animation when stored |
| Validation point | hollow diamond, class-colored stroke | always visible | after each evaluate: small ✓ (green) or ✗ (red) badge |
| Probe (user query) | white crosshair ✚ with glow | — | — |
| Spotlight query (current pipeline query) | validation point with pulsing ring | — | — |

The dim → bright transition is the main visual cue of "the model is learning by memorizing".

### 4.3 Tools (toolbar above canvas)
- **Add** (default): click to add a point at cursor. Class picker chips `A B C D` (only as many as the current class count). Set toggle: `Train | Validation`.
- **Erase**: click or drag with a brush (radius slider 10–60px) to remove points.
- **Probe**: click to place a free query point. The canvas then live-draws its K nearest memorized neighbors, lines to them, the K-radius circle, and a small floating card with the predicted class + probabilities. Probe does **not** reset the model and does not affect loss. Clicking a validation point in Probe mode pins it as the spotlight query instead.
- Right-click on any point = delete (prevent context menu on canvas).
- Hover tooltip: `#12 · class B · train · memorized · (3.41, 7.02)`.
- Any add/erase while a run has started → **reset the model** (clear memory + loss history) and show a toast: *"Data changed — model reset."*

### 4.4 Dataset generator
Controls (compact, collapsible "Generate data" group):
- Pattern: `Gaussian blobs` (default), `Two moons`, `Concentric circles`, `XOR quadrants`, `Spiral`, `Uniform random`.
- Classes: 2–4 (default 3; moons/circles/XOR force 2 and disable the selector with a hint).
- Total points: 20–300 (default 90).
- Noise: 0–1 (default 0.3).
- Validation ratio: 10%–50% (default 25%).
- Seed (number input + 🎲 button). Use a seeded PRNG (`mulberry32`) so the same seed reproduces the same data.
- Buttons: `Generate`, `Clear all`.
- On page load: auto-generate default data so the canvas is never empty.

### 4.5 Display toggles
- Decision boundary (on), Confidence shading (on), Neighbor lines (on), K-radius circle (on), Point indices (off), Grid (on).
- Grid resolution for the boundary: `Low 40×40 / Medium 80×80 (default) / High 120×120`.

### 4.6 Required visual feedback per pipeline sub-step (the "Interaction" requirement)
| Sub-step | What the canvas shows |
|---|---|
| 1 Memorize | New point(s) pop from dim to bright; a small `+1 stored` label floats up |
| 2 Distance | Thin lines from the spotlight query to **every** memorized point, opacity inversely proportional to distance; distance values shown on hover |
| 3 Sort & select K | Non-neighbor lines fade out; the K neighbor lines become thick and class-colored, numbered `1..K` by rank; dashed circle with radius = distance to the K-th neighbor |
| 4 Vote | Neighbors pulse; a mini vote badge next to the query shows counts per class; query fills with the predicted class color and gets ✓/✗ |
| 5 Evaluate | Decision-boundary heatmap re-renders (cross-fade ~250ms); every validation point gets ✓/✗ |

Decision boundary rendering:
- Evaluate the classifier on every grid cell center using the **current memory**.
- Cell color = predicted class color, alpha = `0.12 + 0.45 · (p_max − 1/C) / (1 − 1/C)` when confidence shading is on (else fixed 0.3).
- Draw a thin, slightly brighter line where adjacent cells disagree (the actual boundary).
- Empty memory → uniform neutral grey (model knows nothing).
- Render into an offscreen canvas, then draw it under the points. Recompute only when memory or hyperparameters change, never on every animation frame.

---

## 5. Panel 02 — Hyperparameters & Controls (left, bottom)

### 5.1 Hyperparameters
| Control | Type | Range / options | Default | Help text (tooltip ⓘ) |
|---|---|---|---|---|
| K (number of neighbors) | slider | 1 – 25 | 5 | "How many nearest stored points vote. Small K = wiggly boundary (overfits); large K = smooth boundary (may underfit). Odd K avoids ties for 2 classes." |
| Distance metric | segmented | Euclidean, Manhattan, Chebyshev, Minkowski | Euclidean | Shows the formula of the selected metric |
| Minkowski p | slider (only for Minkowski) | 1 – 5, step 0.5 | 3 | "p=1 → Manhattan, p=2 → Euclidean" |
| Vote weighting | segmented | Uniform, Distance (1/d) | Uniform | "Distance weighting lets closer neighbors count more." |
| Points memorized per epoch (B) | select | 1, 2, 5, 10 | 2 | "How many training points the model stores each epoch." |
| Memorization order | select | Random (seeded), Class round-robin | Random | |
| Spotlight query | select | Auto (cycle validation points), Hardest (a currently misclassified one), Pinned | Auto | |

Advanced (collapsed `Advanced ▸` group):
- Smoothing α (Laplace) for probabilities: 0.1 – 2, default 0.5.
- Early stopping: checkbox (off by default), tolerance ε (default 0.001), patience (default 5 epochs).

**Changing K / metric / p / weighting / α mid-run must NOT reset memory.** Instead:
1. Immediately re-render the decision boundary and spotlight visuals for the current memory.
2. Recompute the **entire loss history** from epoch 0 to the current epoch with the new hyperparameters (cheap: re-evaluate each memory prefix), and draw the old curve as a faint ghost line labeled e.g. `K=3 (previous)` for comparison. Keep only one ghost.

Changing B, memorization order, or data → reset model.

### 5.2 Controls
- `⏭ Step` — advance exactly **one sub-step** (Memorize → Distance → Sort & select K → Vote → Evaluate → next epoch's Memorize …).
- `⏩ Next epoch` — finish the remaining sub-steps of the current epoch instantly.
- `▶ Play until convergence` / `⏸ Pause` (same button toggles).
- `↺ Reset model` — clear memory and loss history, keep data.
- Speed slider: 1 – 20 (sub-steps/sec when sub-step animation is on; epochs/sec when off). Default 4.
- Checkbox `Animate sub-steps while playing` (default on). When off, Play runs whole epochs quickly (only the Evaluate visuals are shown).
- Live metric chips under the buttons: `Epoch t / T`, `Memory |M| / N_train`, `Val loss`, `Val accuracy`, `Phase`.

Convergence:
- Stop when all training points are memorized (`t = T = ⌈N_train / B⌉`), or when early stopping triggers (`|L_t − L_{t−1}| < ε` for `patience` consecutive epochs).
- Show a success banner in Panel 03: *"Converged at epoch 30 — all 68 training points memorized. Validation loss 0.214, accuracy 91.3%."* (or the early-stop variant, stating the reason).
- After convergence, Step/Play are disabled until Reset; Probe still works.

Keyboard shortcuts (show in a small `?` popover): `Space` play/pause, `→` step, `E` next epoch, `R` reset model, `G` generate data, `1–4` select class, `A` add tool, `X` erase tool, `P` probe tool. Ignore shortcuts while typing in inputs.

---

## 6. Panel 03 — Model State / Structure (right, top)

This panel is the heart of the "explain the math" goal. Top to bottom:

### 6.1 Pipeline stepper
Five chips in a row: `1 Memorize · 2 Distance · 3 Sort & Select K · 4 Vote · 5 Evaluate`. The current one is highlighted, completed ones get a ✓. Below: `Epoch t / T · |M| = 24 stored · K = 5 · Euclidean · Uniform`.

### 6.2 Explanation card (changes every sub-step)
A short plain-English explanation + the formula **with the actual numbers substituted** for the spotlight query `q`. Examples:

**Memorize**
> "The model stores 2 new training points. KNN does no other learning: memory grew from 22 to 24 points."
> `M ← M ∪ {(x₂₃, y₂₃), (x₂₄, y₂₄)}`

**Distance (Euclidean shown; adapt per metric)**
> "Measure how far the query q = (4.10, 6.25) is from every stored point."
> `d(q, x₇) = √((3.20 − 4.10)² + (5.90 − 6.25)²) = √(0.81 + 0.1225) = √0.9325 = 0.966`
Show one fully worked example (the nearest point) plus a note "…computed the same way for all 24 points (see table)".

Metric formulas to use:
- Euclidean: `d = √((a₁ − b₁)² + (a₂ − b₂)²)`
- Manhattan: `d = |a₁ − b₁| + |a₂ − b₂|`
- Chebyshev: `d = max(|a₁ − b₁|, |a₂ − b₂|)`
- Minkowski: `d = (|a₁ − b₁|ᵖ + |a₂ − b₂|ᵖ)^(1/p)`

**Sort & Select K**
> "Sort all distances from smallest to largest and keep the first K = 5. The 5th-nearest distance (1.84) becomes the radius of the neighborhood circle."

**Vote**
> Uniform: `score(A) = 3, score(B) = 2, score(C) = 0`
> Distance-weighted: `w = 1 / d`, e.g. `w₁ = 1 / 0.966 = 1.035`, `score(c) = Σ wᵢ for neighbors of class c`
> Probabilities with smoothing α: `p(c) = (score(c) + α) / (Σ scores + α·C)` → `p(A) = (3 + 0.5)/(5 + 1.5) = 0.538`
> `ŷ = argmax p(c) = A` ✓ (true label A)

**Evaluate**
> "Predict every validation point with the current memory, then measure the loss."
> `L = −(1/|V|) Σ ln p(true class)` → `L = 0.412`, `accuracy = 17 / 22 = 77.3%`
> One sentence of intuition: "If the model gives the true class a high probability, ln p is close to 0 → small loss. Confident mistakes are punished hard."

### 6.3 Memory table ("the model's parameters")
Columns: `# | x₁ | x₂ | class | d(q, ·) | rank`.
- Title: **Memory M — stored training points (this is the whole model)**.
- New rows flash on Memorize. During Distance the `d` column fills in (animated top-to-bottom is nice but optional). During Sort the table re-orders by distance with a short FLIP animation; top-K rows get a class-colored left border and bold rank. 
- Scroll inside the table if long (max-height ≈ 260px), sticky header.
- Hovering a row highlights that point on the canvas, and vice versa.

### 6.4 Distance vector and vote vector
- Compact vector view: `d = [0.97, 1.12, 1.40, 1.66, 1.84 | 2.03, 2.47, …]` with the first K entries highlighted and a `|` marking the K cut.
- Vote/probability bar chart: one horizontal bar per class, showing `score` and `p(c)`; predicted class bar outlined.

### 6.5 Validation confusion matrix
C×C matrix (rows = true class, cols = predicted), updated on Evaluate. Diagonal cells tinted green, off-diagonal tinted red with intensity by count. Show accuracy under it.

### 6.6 Converged banner
See §5.2.

---

## 7. Panel 04 — Loss Graph (right, bottom)

- Canvas line chart, x-axis = epoch (0 … T), y-axis = loss. Hand-drawn with Canvas 2D (axes, ticks, grid, labels).
- **Epoch 0 baseline**: with empty memory, every class has `p = 1/C`, so `L₀ = ln C` (e.g. 1.099 for 3 classes). Plot it — this gives the curve a meaningful starting point.
- Series:
  1. **Validation loss (cross-entropy)** — solid accent line with dots, primary y-axis.
  2. **Validation error rate** — dashed line, secondary y-axis 0–100% on the right (toggleable via legend click).
  3. Ghost line from previous hyperparameters (see §5.1), faint and dashed.
- Hover crosshair + tooltip: `epoch 12 · |M| = 24 · loss 0.412 · error 22.7%`.
- Current epoch marker (vertical line). Mark the convergence/early-stop point with a small flag.
- Caption under the chart (always visible): *"Loss usually drops as the model remembers more examples. Small bumps are normal — a newly stored point can occasionally flip a nearby prediction."*

**Optional stretch (implement only after everything else works):** a second tab `Error vs K` — after convergence, compute validation error for K = 1…25 with the full memory and plot it, highlighting the best K. This teaches hyperparameter selection.

---

## 8. Algorithm reference (exact math to implement)

Let `C` = number of classes, `M = {(xᵢ, yᵢ)}` = memory, `V` = validation set, `q` = query.

1. **Distance** `dᵢ = d(q, xᵢ)` for all `i ∈ M` using the selected metric.
2. **Sort** ascending; ties broken by smaller index (stable sort).
3. **Neighbors** `N = first min(K, |M|)` items. If `K > |M|`, use all of `M` and show a warning chip `K > stored points — using all |M|`.
4. **Weights**: uniform `wᵢ = 1`; distance `wᵢ = 1 / (dᵢ + 1e−6)`. If any `dᵢ = 0` in distance mode, use only the zero-distance neighbors (exact match dominates).
5. **Scores** `s_c = Σ_{i∈N, yᵢ=c} wᵢ`.
6. **Probabilities** `p_c = (s_c + α) / (Σ_c' s_c' + α·C)`. Empty memory → `p_c = 1/C`.
7. **Prediction** `ŷ = argmax_c p_c`. Tie-break: among tied classes pick the class of the nearest neighbor; if still tied, smallest class index.
8. **Loss** `L = −(1/|V|) Σ_{j∈V} ln(max(p_{yⱼ}(qⱼ), 1e−6))`; **error** = misclassified / |V|.
9. **Epoch t**: memorize the next `B` points from the memorization order, then evaluate. `T = ⌈N_train / B⌉`.

Edge cases:
- No validation points → show a clear message in Panels 03/04 ("Add validation points or regenerate data with a validation ratio > 0") and disable Play; Step still allowed for Memorize/Distance/Vote using a probe or the first training point as query.
- No training points → disable Step/Play with a hint.
- A class with zero training points is fine (it just never gets votes); keep it in C.
- Max 300 points total; show a toast if the user tries to exceed it.

Performance targets: boundary grid 80×80 with 300 stored points ≈ 2M distance ops → must finish < 50ms. Precompute flat `Float32Array`s for coordinates. Recomputing the full loss history (≤ 300 epochs × ≤ 150 validation × ≤ 300 memory) must stay < 200ms; if slower, run it in chunks with `requestAnimationFrame`.

---

## 9. State machine

```
state = {
  data: { points: [{id, x1, x2, cls, set: 'train'|'val'}], classCount },
  hp:   { K, metric, p, weighting, alpha, B, order, spotlightMode, earlyStop, eps, patience },
  run:  { order: [trainIds], memory: [trainIds], epoch, T, phase: 0..4 (MEMORIZE..EVALUATE),
          spotlightId | probe, lastDistances, lastNeighbors, lastScores, lastProbs,
          history: [{epoch, memSize, loss, error}], ghost: {label, history} | null,
          status: 'idle'|'running'|'paused'|'converged' },
  ui:   { tool, addClass, addSet, toggles, gridRes, speed, animateSubsteps }
}
```

- One function `stepOnce()` advances `phase` and performs that phase's work, then calls `renderAll()` (or targeted renders).
- `play()` schedules `stepOnce()` with `setTimeout` based on speed; stops on convergence or pause.
- All KNN math lives in **pure functions** (`distance`, `knnPredict`, `evaluate`, `computeBoundary`) that never touch the DOM — so they can be tested.

---

## 10. Accessibility & polish

- All buttons/inputs have labels or `aria-label`; visible focus ring (accent color).
- Respect `prefers-reduced-motion` (disable pops, FLIP, cross-fades).
- Tooltips on every hyperparameter (ⓘ icon) with one-sentence beginner explanations.
- Toasts bottom-center, auto-dismiss after 2.5s.
- Numbers: distances 3 decimals in formulas, 2 decimals in tables; loss 3 decimals; percentages 1 decimal.
- No console errors or warnings during normal use.

---

## 11. Code organization inside `index.html`

```
<style>   tokens (:root vars) → layout → panels → controls → tables → canvas → footer → responsive → reduced-motion
<script>
  // 0. CONFIG & MEMBERS (MEMBERS array, colors, limits)
  // 1. UTILITIES (mulberry32 PRNG, clamp, format, debounce, HiDPI canvas helper)
  // 2. DATA GENERATORS (blobs, moons, circles, xor, spiral, uniform) — seeded
  // 3. KNN CORE (pure: distance, knnPredict, evaluate, boundaryGrid, lossHistory)
  // 4. STATE & STATE MACHINE (stepOnce, nextEpoch, play/pause, resetModel, convergence)
  // 5. RENDER: DATA CANVAS (points, boundary offscreen, overlays per phase, probe card)
  // 6. RENDER: MODEL STATE (stepper, explanation + formulas, memory table, vectors, votes, confusion)
  // 7. RENDER: LOSS CHART (axes, series, ghost, hover tooltip)
  // 8. UI BINDINGS (tools, sliders, buttons, keyboard, resize observer)
  // 9. SELF-TESTS (run when URL has ?test=1; log PASS/FAIL to console)
  // 10. INIT
</script>
```

### 11.1 Self-tests (`?test=1`)
At minimum:
- Euclidean `d((0,0),(3,4)) = 5`; Manhattan = 7; Chebyshev = 4; Minkowski p=2 equals Euclidean.
- Empty memory gives `p_c = 1/C` and loss `ln C`.
- K=1 on a point that is itself in memory predicts its own class.
- Uniform vote with neighbors `[A, A, B]` → `p(A) = (2+α)/(3+αC)`.
- Tie-break picks the nearest neighbor's class.
- Loss history length = T + 1 (including epoch 0) after convergence.
- Same seed → identical generated dataset.

---

## 12. Implementation plan (do it in this order, verify each phase in the browser)

1. **Skeleton**: header, two floating scrollable columns, four numbered collapsible panels, footer, theme tokens, responsive stacking.
2. **Data canvas**: coordinate system, generators + seed, add/erase/right-click delete, hover tooltip, point encodings.
3. **KNN core + self-tests** (pure functions, `?test=1`).
4. **State machine + controls**: Step / Next epoch / Play / Pause / Reset, speed, convergence, metric chips.
5. **Model State panel**: stepper, explanation card with substituted numbers, memory table, vectors, vote bars, confusion matrix.
6. **Canvas phase overlays**: neighbor lines, K-circle, ranks, vote badge, ✓/✗ markers, pop animation.
7. **Decision boundary** heatmap + boundary lines + confidence shading + grid resolution.
8. **Loss chart**: baseline, val loss, error rate, hover, ghost line on hyperparameter change, convergence flag.
9. **Probe tool** with live floating prediction card.
10. **Polish**: tooltips, keyboard shortcuts, toasts, reduced motion, performance check with 300 points, cross-browser check.
11. **README.md** + deployment notes.
12. (Stretch) `Error vs K` tab.

---

## 13. Deployment (GitHub Pages)

1. Put `index.html` (and `README.md`) at the repository root on branch `main`.
2. GitHub → **Settings → Pages → Build and deployment → Source: Deploy from a branch → Branch: `main` / `(root)`** → Save.
3. Wait 1–2 minutes; the site is live at `https://<username>.github.io/<repo>/`.
4. Use only relative/no asset paths (everything is inline), so it works under the `/<repo>/` sub-path.

---

## 14. Acceptance checklist

- [ ] Single `index.html`, no external JS, opens via double-click and on GitHub Pages.
- [ ] Black, professional theme; header with title + principle; footer with group, members, CE KMITL.
- [ ] Two floating columns with independent vertical scrollbars; stacks on narrow screens.
- [ ] Data canvas: random generation (6 patterns, seeded) + click to add / erase / right-click delete.
- [ ] Hyperparameter panel with K, metric, weighting, B, order, spotlight, advanced α/early-stop.
- [ ] `Step` advances one sub-step; `Play until convergence` runs to completion and stops with a banner.
- [ ] Model State shows memory table, distance vector, vote vector, worked formulas with real numbers, confusion matrix.
- [ ] Loss graph starts at `ln C`, updates every epoch, shows error rate, ghost line on hyperparameter change.
- [ ] Canvas visibly changes on every sub-step; decision boundary re-renders after each evaluate.
- [ ] Changing K mid-run updates the boundary and recomputes the loss history without wiping memory.
- [ ] `?test=1` self-tests all pass; no console errors.
- [ ] All UI text in English.
