# WARP.md

This file provides guidance to WARP (warp.dev) when working with code in this repository.

## Rule sources and precedence

This repo uses the generic **Warping** rules directory at `Warping/`.

When working in this project, follow these rule files in order of precedence (highest first):

1. `Warping/user.md` – user-specific preferences (address the user as **Vis**).
2. `Warping/project.md` – project-level guidelines (note that parts of this file reference a separate CLI project and Taskfile; treat those sections as aspirational unless matching tooling exists in this repo).
3. `Warping/python.md` / `Warping/go.md` – language-specific standards, if and when you introduce those languages here.
4. `Warping/taskfile.md` – Taskfile usage patterns, if a `Taskfile.yml` is added for this project.
5. `Warping/main.md` – general Warp AI behavior (documentation locations, secrets handling, commit conventions, and quality expectations).
6. `Warping/specification.md` – pointer to higher-level specs (currently references `../SPECIFICATION.md`, which does not exist in this repo).

Key global expectations from these files that apply here:

- Prefer **Conventional Commits** for commit messages.
- Keep secrets in a `secrets/` directory (as `.env`-style files), never in code.
- Prefer placing new documentation in `docs/` rather than the repo root (this `WARP.md` is a deliberate exception).
- When you introduce tooling, prefer a `Taskfile.yml`-based workflow (`task check`, `task test:coverage`, etc.).

## Project overview

This repository currently hosts a **static single-page web experience** for the Sighthound hardware savings calculator:

- Entry point: `index.html` in the repo root.
- Everything (markup, styling, and behavior) is inlined in this file; there is no build system, bundler, or framework.
- The page includes marketing content (hero, pricing spotlight, features, FAQ, and final CTA) plus an **interactive savings calculator**.
- There is no backend component; all behavior runs client-side in the browser.

### Calculator architecture

The savings calculator is implemented as a small vanilla JavaScript widget embedded at the bottom of `index.html`:

- **Data model**
  - Two product types, with hard-coded unit prices and a simple bulk-discount rule:
    - Compute nodes – `UNIT_COMPUTE = 2000` (USD).
    - Cameras – `UNIT_CAMERA = 2500` (USD).
    - Bulk discount – `DISCOUNT_RATE = 0.05` (5%) applied per line when quantity is greater than `DISCOUNT_THRESHOLD = 5` (i.e., 6+ units).
- **DOM structure**
  - Quantity inputs live in the calculator section as `input[type="number"]` elements with IDs `qty-compute` and `qty-camera`, each wrapped in plus/minus "stepper" controls using `.qty-btn` buttons and `data-target` / `data-step` attributes.
  - Per-line subtotals, discounts, and totals are updated via IDs such as `compute-subtotal`, `compute-discount`, `compute-total`, `camera-*` equivalents, and line-level containers like `compute-subtotal-row`.
  - Badges and explanatory text for discounts use elements like `#bulk-badge-compute`, `#bulk-badge-camera`, `#compute-discount-note`, and `#camera-discount-note`, with an `active` class toggled based on whether the discount is applied.
  - A **receipt-style summary** and a **hero preview** mirror the per-line values via IDs such as `summary-subtotal`, `summary-discount`, `summary-total`, `summary-save`, `hero-subtotal`, `hero-discount`, `hero-total`, and `hero-save`.
- **Computation & update flow**
  - Core helpers:
    - `clampQty(value)` – sanitizes and clamps quantities to the inclusive range `0–999`.
    - `animateValue(el, newValue)` – interpolates from the previous numeric value to the new one over ~180ms and formats it as a USD currency string using `Intl.NumberFormat`.
    - `updateNumber(el, value)` – thin wrapper that calls `animateValue`.
  - `syncTotals()` – the main recomputation routine:
    - Reads and clamps quantities from the compute and camera inputs.
    - Derives per-line subtotals, discounts, and totals.
    - Aggregates into a global subtotal, total discount, final total, and "you save" figure.
    - Updates all DOM outputs and toggles relevant CSS classes (`active`, `subtotal-strike`) to visually distinguish pre-discount amounts.
  - `attachQtyControls()` – wires event handlers:
    - Click handlers for `.qty-btn` plus/minus buttons (driven by `data-target` and `data-step`).
    - `input` and `blur` handlers on numeric fields to clamp and resync.
    - Keyboard support: `ArrowUp` / `+` increment, `ArrowDown` / `-` decrement.
  - `initNumberDefaults()` – initializes all `.number-animate` elements with a `data-value` attribute and formats their initial display.
  - On `DOMContentLoaded`, the script runs `initNumberDefaults()`, `attachQtyControls()`, and `syncTotals()` once to ensure the UI and data are in sync at page load.

The rest of `index.html` is primarily **semantic HTML and CSS** for layout, visual design, accessibility (skip links, ARIA labels, live regions), and SEO (Open Graph tags and a JSON-LD `WebPage`/`Product` schema block).

## Local development and commands

There is currently **no task runner, package.json, or build/test tooling** configured in this repository. All behavior is implemented in `index.html` and runs directly in the browser.

### Run the calculator locally

On macOS or Linux, you can either open the file directly or serve it via a simple static server:

```bash
open index.html
```

or:

```bash
python3 -m http.server 8000
# then visit http://localhost:8000/ in your browser and open /index.html
```

### Linting, formatting, and tests

- **JavaScript/HTML tooling is not yet configured.** There is no `Taskfile.yml`, `package.json`, or test configuration in this repo at the moment.
- If you introduce a tooling stack (e.g., ESLint, Jest/Vitest, Playwright, or a bundler), update this `WARP.md` with:
  - How to install dependencies.
  - How to run the main test suite.
  - How to run a single test or a focused spec.
  - Any `task` aliases (e.g., `task check`, `task test:coverage`) that wrap those commands.

Until such tooling exists, there are **no canonical build, lint, or test commands** beyond opening `index.html` and manually exercising the UI in a browser.