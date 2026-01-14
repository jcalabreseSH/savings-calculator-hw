# Sighthound Savings Calculator (Homework Version)

This repository contains a static single-page web experience for the Sighthound hardware savings calculator. It is a lightweight HTML/CSS/JavaScript implementation with no build step or backend.

## Project Overview

- **Entry point:** `index.html` in the project root
- **Tech stack:**
  - Plain HTML for structure and content
  - CSS (inlined in `index.html`) for styling and layout
  - Vanilla JavaScript (inlined in `index.html`) for interactivity
- **Purpose:**
  - Present marketing content for Sighthound's hardware offering
  - Provide an interactive calculator so users can estimate savings based on the number of compute nodes and cameras

## Calculator Behavior

The savings calculator lives at the bottom of `index.html` and behaves as follows:

- Two product lines:
  - **Compute nodes** – unit price: **$2,000** USD
  - **Cameras** – unit price: **$2,500** USD
- Bulk discount rule (per line item):
  - If quantity > 5 (i.e. 6 or more units), a **5% discount** is applied to that line
- UI behavior:
  - Quantity inputs use plus/minus buttons and keyboard support
  - Line subtotals, discounts, and totals update live as quantities change
  - A receipt-style summary and a hero preview mirror the calculated values

All logic runs entirely in the browser; there is no server component.

## Running the Project Locally

You can open the static page directly in a browser or serve it via a simple HTTP server.

### Option 1: Open the file directly (macOS)

```bash
open index.html
```

### Option 2: Simple Python HTTP server

From the project root:

```bash
python3 -m http.server 8000
# then visit http://localhost:8000/ and open /index.html
```

## Development Notes

- There is **no** `package.json`, bundler, or build tooling configured.
- There are currently **no automated tests, linters, or formatters** set up in this repo.
- Any changes should be made directly in `index.html` and then tested manually in a browser.

If you later add tooling (e.g., ESLint, a test runner, or a bundler), update this `README.md` with installation and usage instructions.
