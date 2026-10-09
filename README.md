# KNN Interactive Algorithm Visualizer

An interactive, single-page visualizer that teaches the **K-Nearest Neighbors (KNN)** classifier step by step, with the math of every step shown using real numbers. Built for complete beginners — *learning by remembering*.

> KNN has no weights to learn. Its "model" is simply the memory of training examples. Training = storing points. Prediction = finding the K closest stored points and letting them vote.

<!-- TODO: replace with a real screenshot, e.g. docs/screenshot.png -->
![Screenshot placeholder](docs/screenshot.png)

## How KNN is mapped onto "training"

KNN is a lazy, non-parametric algorithm, so there is no gradient descent. The app uses an honest framing instead:

| Concept | In this app |
|---|---|
| Training / model update | **Memorization** — each epoch stores the next `B` training points |
| Model parameters | The **Memory table** (stored points and labels) — the data *is* the model |
| One step | One sub-step: Memorize → Distance → Sort & Select K → Vote → Evaluate |
| Loss | Validation **cross-entropy** of the memory-based classifier (epoch 0 = ln C) |
| Convergence | All training points memorized, or optional early stopping |

The loss generally decreases as more examples are stored, but small bumps are normal.

## Features

- Seeded dataset generator (Gaussian blobs, two moons, concentric circles, XOR, spiral, uniform) plus click-to-add, erase brush and right-click delete.
- `Step`, `Next epoch`, `Play until convergence`, `Reset model`, speed control.
- Hyperparameters: K, distance metric (Euclidean / Manhattan / Chebyshev / Minkowski), vote weighting, smoothing α, points stored per epoch, memorization order, spotlight query, early stopping. Changing K / metric / weighting mid-run keeps the memory and recomputes the loss curve (the previous curve stays as a faint ghost line).
- Model State panel: pipeline stepper, worked formulas with the real numbers of the spotlight query, memory table, distance vector, vote bars, confusion matrix.
- Decision-boundary heatmap with confidence shading, neighbor lines, K-radius shape (circle, diamond, square, … depending on the metric) and ✓/✗ markers.
- Probe tool: drop a free query point and see its K nearest neighbors and class probabilities live.
- Hand-drawn canvas loss chart with validation loss, error rate, hover tooltip, ghost line and convergence flag.

Keyboard: `Space` play/pause · `→` step · `E` next epoch · `R` reset · `G` generate · `1–4` class · `A` / `X` / `P` tools · `Esc` clear probe.

## Run locally

No build step and no dependencies. Open `index.html` in a browser (double-click), or serve the folder:

```bash
python -m http.server 8000   # then open http://localhost:8000
```

Self-tests: open `index.html?test=1` and check the browser console (and the green badge in the top-right corner). All tests should report `PASS`.

## Deploy to GitHub Pages

1. Put `index.html` and `README.md` at the repository root on branch `main`.
2. GitHub → **Settings → Pages → Build and deployment → Source: Deploy from a branch → Branch: `main` / `(root)`** → Save.
3. After 1–2 minutes the site is live at `https://<username>.github.io/<repo>/`.

Everything is inline and uses no asset paths, so it works under the `/<repo>/` sub-path. Google Fonts (Inter, JetBrains Mono) are optional; the app falls back to system fonts offline.

## Group

**Group: ManchesterLiver** — Computer Engineering, KMITL (CE KMITL)

| Student ID | Name |
|---|---|
| 66010064 | กาลัญญ จินดา |
| 66010089 | ขจรยศ จูเลิศ |
| 66010324 | ธนกฤต สุไชยชิต |
| 66011463 | รัตนพล มาสูตร |
| 66011520 | อาณา สุขอุดม |
| 66011471 | วรพัฒน์ คำศรี |

> TODO: the page is meant to be in English — replace the Thai names with the members' official English names in the `MEMBERS` array at the top of the script in `index.html` (and in this table).
