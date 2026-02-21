# Budget Application

A lightweight budgeting UI built with plain HTML/CSS/JavaScript. Users add income and expense line items and the budget summary recalculates immediately.

Live preview: https://budgetapp.fcjamison.com/

This repo is intentionally framework-free. It’s a good reference for:

- separation between model (business math), UI rendering, and orchestration
- DOM event wiring via event delegation
- small, deterministic state management without dependencies

## Features (user-facing)

- Add Income (+) and Expense (-) items (description + numeric amount)
- Delete any item and see totals update instantly
- Top summary:
  - total income
  - total expenses
  - remaining budget (income − expenses)
  - overall expense percentage (expenses ÷ income)
- Displays the current month from the user’s system clock

## Tech stack

- HTML5 + CSS3 (BEM-style class naming)
- Vanilla JavaScript (module pattern via IIFEs)
- CDN assets:
  - Google Fonts (Open Sans)
  - Ionicons v2

## Quickstart (developers)

This is a static site: no build step, no package manager required.

### Option A — open the file (fastest)

- Open `index.html` in your browser.

Notes:

- Running via `file://` works, but serving over HTTP is closer to real hosting.
- The icon font and Google font are loaded from CDNs, so offline development will look different.

### Option B — run a local static server (recommended)

From the repo root:

- Node: `npx http-server` (or any static server)
- Python 3: `python -m http.server`

Then open the URL printed in the terminal.

### Option C — use the workspace URL task

This workspace includes a VS Code task named “Open in Browser” that opens:

- `http://thebudgetapplication.localhost/`

That URL assumes you already have a local web server + hosts/DNS configuration for that domain. If you don’t, use Option A or B.

## Project layout

- `index.html` — markup + script entry
- `css/style.css` — styles
- `js/app.js` — all application logic
- `images/back.png` — hero background

## Architecture (how the code is organized)

All logic lives in `js/app.js` and is split into three IIFE modules:

### 1) `budgetController` (model + calculations)

Responsibilities:

- Holds all in-memory state in a single `data` object:

  ```js
  {
    allItems: { exp: [], inc: [] },
    totals: { exp: 0, inc: 0 },
    budget: 0,
    percentage: -1
  }
  ```

- Creates items via the `Income` and `Expense` constructor functions
- Generates IDs per type as `lastId + 1` (or `0` when empty)
- Owns all business math:
  - totals are sums of `value`
  - `budget = totalInc - totalExp`
  - `percentage = round(totalExp / totalInc * 100)`, or `-1` when income is `0`

Key API:

- `addItem(type, description, value)`
- `deleteItem(type, id)`
- `calculateBudget()`
- `getBudget()`

### 2) `UIController` (DOM reads/writes)

Responsibilities:

- Centralizes selectors in `DOMStrings`
- Reads form state via `getInput()`:
  - `type` is `inc` or `exp`
  - `value` is parsed via `parseFloat(...)`
- Writes list rows via `insertAdjacentHTML(...)`
- Updates the header totals in `displayBudget(budget)`
- Sets the month label in `displayMonth()`

DOM contract:

- The app relies on specific class names like `.add__description`, `.income__list`, `.expenses__list`, `.budget__value`, etc.
- List row element IDs are rendered as `inc-<id>` and `exp-<id>`.

### 3) `controller` (wiring + orchestration)

Responsibilities:

- Registers event listeners
- Implements the two user flows:
  - `ctrlAddItem()` — validate → add → render → clear fields → update totals
  - `ctrlDeleteItem()` — identify clicked row → delete model → delete DOM → update totals
- Initializes on `DOMContentLoaded`:
  - writes initial “0” budget state
  - writes current month label

## Runtime behavior (important implementation details)

### Data lifecycle

- State is in-memory only (refresh resets everything)
- The DOM row IDs (`inc-0`, `exp-3`, …) are the bridge between UI and model

### Events

- Add item:
  - click `.add__btn`
  - press Enter (document-level `keypress` handler)
- Delete item:
  - single delegated click handler on `.container`

### Cache busting

The script is included as `js/app.js?v=...` in `index.html`. If you’re fighting browser caching during development, bump the `v=` value.

## Common changes

- Styling/layout: edit `css/style.css`
- Business rules (totals/percentages): edit `budgetController` in `js/app.js`
- Markup/labels: edit `index.html` and the HTML templates in `UIController.addListItem(...)`

## Troubleshooting

- Icons not showing
  - Ionicons is loaded from an `http://` URL in `index.html`. If you serve the app over HTTPS, browsers may block it as mixed content.
  - Fix by switching the Ionicons link to HTTPS (or host the icon CSS locally).

- Delete doesn’t work after changing markup
  - `ctrlDeleteItem` uses chained `parentNode` traversal to find the row ID. If you change the nesting, update the traversal or use `closest('.item')`.

- Numbers look “raw”
  - Values are displayed as plain JS numbers (no currency formatting, rounding, or thousands separators).

- “thebudgetapplication.localhost” doesn’t load
  - That URL is workspace-specific and depends on your local web server setup. Use `index.html` directly or run a static server.

## Known limitations (by design)

- No persistence (no LocalStorage / backend)
- Expense-row percentage is currently a hard-coded placeholder in the template
- Uses legacy `keypress`/`keyCode` patterns for Enter detection

## Deployment

Deploy to any static host (GitHub Pages, Netlify, S3, IIS static, etc.):

- Publish `index.html` plus `css/`, `js/`, and `images/`
- No environment variables or server-side logic required

## Debugging tips

- Open DevTools Console; the modules are global (`budgetController`, `UIController`, `controller`).
- Inspect in-memory state by calling `budgetController.testing()`.
- If you refactor selectors, note the accessor is named `getDOMStrrings()` (spelling as in code).
