# Sighthound Hardware Savings Calculator – Specification

## 1. Project Summary

A static, single-page marketing site for Sighthound’s edge AI hardware, featuring an **interactive savings calculator** that models one-time hardware costs for:

- **Compute nodes** (GPU-ready edge compute)
- **Cameras** (smart security cameras)

All markup, styles, and behavior live directly in `index.html` with a small amount of inline JavaScript. There is **no backend** and no build pipeline; the page is served as plain HTML/CSS/JS.

This specification documents the **current behavior** of the existing `index.html` experience. It does **not** introduce a new version or roadmap of changes.

## 2. Scope

### 2.1 In Scope

- A single HTML page (`index.html`) containing:
  - Hero section with marketing copy and a live estimate preview.
  - Pricing spotlight for compute nodes and cameras.
  - Bulk discount explainer.
  - Interactive calculator widget (quantities, line items, discounts, receipt summary).
  - Features section.
  - FAQ section.
  - Final call-to-action.
- Client-side calculator logic implemented via inline `<script>`:
  - Read and sanitize quantities for compute nodes and cameras.
  - Apply hard-coded unit prices and bulk-discount rules.
  - Compute and display per-line and overall totals and savings.
  - Keep hero preview and receipt summary in sync.
- JSON-LD metadata for SEO.

### 2.2 Out of Scope

- Any backend service or API.
- Authentication, user accounts, or persistence of scenarios.
- Multi-page flows, wizards, or dashboards.
- Support for additional product SKUs beyond the existing **Compute Node** and **Camera**.
- New pricing logic beyond what is currently implemented.

## 3. Functional Specification – Calculator

### 3.1 Inputs

The calculator exposes two numeric inputs:

- **Compute nodes quantity**
  - Element: `input#qty-compute[type="number"]`
  - Allowed range: `0`–`999` (inclusive)
  - Default value: `0`
- **Cameras quantity**
  - Element: `input#qty-camera[type="number"]`
  - Allowed range: `0`–`999` (inclusive)
  - Default value: `0`

Input methods supported:

- Direct typing into each `<input type="number">`.
- Plus/Minus **stepper buttons** (`.qty-btn`) with `data-target` and `data-step` attributes.
- Keyboard shortcuts when an input is focused:
  - `ArrowUp` or `+` → increment by 1.
  - `ArrowDown` or `-` → decrement by 1.

### 3.2 Pricing Model

Hard-coded constants (inline script):

- `UNIT_COMPUTE = 2000` (USD)
- `UNIT_CAMERA = 2500` (USD)
- `DISCOUNT_RATE = 0.05` (5%)
- `DISCOUNT_THRESHOLD = 5`

**Bulk discount rule:**

- For each **line item** (compute or camera) independently:
  - If `quantity > DISCOUNT_THRESHOLD` (i.e., `>= 6` units), apply a **5% discount** to that line’s subtotal.
  - Otherwise, discount for that line is `0`.

### 3.3 Computation Flow (`syncTotals`)

On each relevant user interaction (button click, input change, blur, keydown), `syncTotals()` recomputes all derived values.

#### 3.3.1 Quantity Sanitization

A helper `clampQty(value)` ensures inputs are in the valid range:

- Parse the string as base-10 integer.
- If `NaN` or `< 0` → return `0`.
- If `> 999` → return `999`.
- Otherwise → return the parsed integer.

`syncTotals()`:

1. Reads raw `.value` from `qty-compute` and `qty-camera`.
2. Clamps them via `clampQty`.
3. Writes the clamped values back to the inputs if they changed.

#### 3.3.2 Line-Level Calculations

For compute nodes:

- `computeSubtotal = qtyCompute * UNIT_COMPUTE`
- `computeDiscount = (qtyCompute > DISCOUNT_THRESHOLD) ? (computeSubtotal * DISCOUNT_RATE) : 0`
- `computeTotal = computeSubtotal - computeDiscount`

For cameras:

- `cameraSubtotal = qtyCamera * UNIT_CAMERA`
- `cameraDiscount = (qtyCamera > DISCOUNT_THRESHOLD) ? (cameraSubtotal * DISCOUNT_RATE) : 0`
- `cameraTotal = cameraSubtotal - cameraDiscount`

#### 3.3.3 Aggregate Calculations

- `subtotal = computeSubtotal + cameraSubtotal`
- `totalDiscount = computeDiscount + cameraDiscount`
- `finalTotal = subtotal - totalDiscount`

“Savings” is presented as **`totalDiscount`**.

### 3.4 UI Updates

All currency values are represented as numeric `data-value` attributes and rendered using `Intl.NumberFormat("en-US", { style: "currency", currency: "USD" })`.

#### 3.4.1 Per-Line UI

For **compute**:

- Subtotal: `#compute-subtotal`
- Discount: `#compute-discount` (shown as negative value)
- Line total: `#compute-total`
- Badge: `#bulk-badge-compute` (class `active` when discount > 0)
- Discount note: `#compute-discount-note` (class `active` when discount > 0)

For **cameras**:

- Subtotal: `#camera-subtotal`
- Discount: `#camera-discount` (shown as negative value)
- Line total: `#camera-total`
- Badge: `#bulk-badge-camera` (class `active` when discount > 0)
- Discount note: `#camera-discount-note` (class `active` when discount > 0)

#### 3.4.2 Summary Receipt UI

Receipt section elements:

- Subtotal before discounts: `#summary-subtotal`
- Total discount: `#summary-discount` (negative value)
- Final total: `#summary-total`
- Savings (“You save”): `#summary-save`

Visual state:

- Container `#summary-subtotal-row` toggles class `subtotal-strike` when `totalDiscount > 0`.

#### 3.4.3 Hero Preview UI

Hero card mirrors summary values:

- Subtotal: `#hero-subtotal`
- Discount: `#hero-discount`
- Total: `#hero-total`
- Savings: `#hero-save`

Behavior:

- The element containing `#hero-subtotal` toggles class `subtotal-strike` when `totalDiscount > 0`.

### 3.5 Animation & Formatting

A helper `animateValue(el, newValue)`:

- Reads previous numeric value from `el.dataset.value` (default `0`).
- If unchanged, directly formats and sets the currency text.
- Otherwise, runs a short (≈180 ms) `requestAnimationFrame` loop:
  - Computes eased interpolation (`easeOutQuad`) between old and new values.
  - Updates `textContent` with formatted currency each frame.
  - At the end, sets:
    - `el.textContent` to final formatted value.
    - `el.dataset.value = String(newValue)`.
    - Removes CSS class `updated`.
- Before animating, applies CSS class `updated` to the element.

Wrapper `updateNumber(el, value)` simply calls `animateValue` when the element exists.

### 3.6 Initialization

On `DOMContentLoaded`:

1. `initNumberDefaults()`
   - For all `.number-animate` elements:
     - Ensure `el.dataset.value` is set (default "0").
     - Set `textContent` to the formatted currency for `Number(el.dataset.value)`.
2. `attachQtyControls()`
   - Attaches click handlers to all `.qty-btn` elements.
   - Attaches `input`, `blur`, and `keydown` handlers to `#qty-compute` and `#qty-camera`.
3. `syncTotals()`
   - Runs once to ensure visible UI matches initial state.

## 4. Non-Functional Requirements

### 4.1 Performance

- All logic runs client-side with lightweight, inline JavaScript.
- No external JS dependencies or bundlers.
- Calculator updates should feel instantaneous on modern browsers.

### 4.2 Accessibility

- Use semantic HTML sections (`<section>`, `<aside>`, `<header>`, `<main>`).
- Provide ARIA labels for regions (e.g., calculator, receipt summary).
- Use `aria-live="polite"` on dynamic savings areas for screen readers.
- Preserve or enhance:
  - Skip link to the calculator.
  - Screen-reader-only text where present (e.g., `sr-only` classes).

### 4.3 Browser Support

- Modern evergreen desktop and mobile browsers.
- Relies on `Intl.NumberFormat` and `requestAnimationFrame`.
- No requirement to support legacy browsers without these APIs.

## 5. Implementation Notes

- All behavior stays contained in a self-invoking function in an inline `<script>` tag at the bottom of `index.html`.
- Any future refactors (if desired) could split JavaScript into a separate `.js` file, but that is **not** part of this specification.
- Pricing constants and discount rules are currently hard-coded and must be updated directly in the script if business rules change.

## 6. Testing Expectations (High-Level)

Although no automated test tooling is currently configured, the following behaviors should be manually validated when making changes:

1. **Quantity handling**
   - Typing values, using +/- buttons, and arrow keys all update totals correctly.
   - Values are clamped between 0 and 999; negative, non-numeric, or extreme values are normalized.

2. **Discount activation**
   - When either line exceeds 5 units, that line gets a 5% discount and the corresponding bulk badge and discount note become active.
   - When quantities drop back to 5 or below, discounts and badges deactivate.

3. **Summary & hero sync**
   - Receipt and hero preview always match the per-line calculations.
   - Subtotal rows strike-through only when there is a non-zero discount.

4. **Animation & formatting**
   - All currency fields display properly formatted USD values.
   - Quick successive changes (rapid clicking or typing) do not lead to NaN, Infinity, or obviously incorrect values.

This specification captures the **current intended behavior** of the existing `index.html` page without proposing any redesign or new feature work.
