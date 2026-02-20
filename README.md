# Budget Application

A lightweight budgeting UI built with **plain HTML/CSS/JavaScript**. Users can add **income** and **expense** line items and see the budget summary update immediately.

Live preview: https://budgetapp.fcjamison.com/

This repo is intentionally framework-free and is designed as a clean reference implementation of:

- separation between **data/model**, **UI rendering**, and **application orchestration**
- DOM event wiring via **event delegation**
- deterministic, testable business math for budget totals

## What it does (product view)

- Add **Income (+)** items (description + numeric amount)
- Add **Expense (-)** items (description + numeric amount)
- Delete any item and see totals update instantly
- Show a top summary:
  - total income
  - total expenses
  - remaining budget (income − expenses)
  - overall expense percentage (expenses ÷ income)
- Display the **current month** from the user’s system clock

## Tech stack

- **HTML5** for structure
- **CSS3** for layout and styling (BEM-style class naming)
- **Vanilla JavaScript** using the module pattern (IIFEs)
- External assets:
  - Google Fonts (Open Sans)
  - Ionicons (v2) via CDN

## Getting started (developers)

### Prerequisites

You only need a modern web browser.

Optional (recommended) for local development:

- Node.js (to run a tiny static server)
- OR Python (built-in static server)
- OR a VS Code static server extension (for example, “Live Server”)

### Run locally

This is a static site. No build step.

#### Option A — open directly

- Open `index.html` in your browser.

Note: opening via `file://` works for most features, but running a local server tends to better match production behavior.

#### Option B — serve as a static site (recommended)

From the repo root:

- Node: `npx http-server`
- Python (3.x): `python -m http.server`

Then open the printed local URL.

### VS Code workflow

- Open the repo folder in VS Code.
- If your workspace includes a task that opens a local URL (for example `http://thebudgetapplication.localhost/`), run the VS Code task **“Open in Browser”**.

## Repository layout

- `index.html` — page structure and script entry
- `css/style.css` — styling
- `js/app.js` — application logic
- `images/` — background image asset

## Architecture (developer view)

All application logic lives in `js/app.js` and is intentionally split into three modules (module pattern via IIFEs):

1. **`budgetController` (model + calculations)**

- Owns all in-memory state in a single `data` object:

  ```js
  {
    allItems: { inc: [], exp: [] },
    totals: { inc: 0, exp: 0 },
    budget: 0,
    percentage: -1
  }
  ```

- Creates items via simple constructor functions (`Income`, `Expense`)
- Assigns deterministic IDs per type (last ID + 1)
- Performs all math in one place:
  - `totalInc = sum(inc)`
  - `totalExp = sum(exp)`
  - `budget = totalInc - totalExp`
  - `percentage = round((totalExp / totalInc) * 100)` or `-1` when income is 0

2. **`UIController` (DOM reads/writes)**

- Centralizes query selectors in a `DOMStrings` map
- Reads form data via `getInput()` and parses numeric values with `parseFloat`
- Renders list items by building HTML strings and inserting them with `insertAdjacentHTML`
- Updates the “budget header” UI in one method (`displayBudget`)
- Sets the month label via `displayMonth()` (uses `new Date()` + month name lookup)

3. **`controller` (wiring + orchestration)**

- Owns event listener registration
- Defines the two main user flows:
  - `ctrlAddItem()` — validate → add to model → render → clear fields → recompute summary
  - `ctrlDeleteItem()` — identify clicked row → delete from model → delete from DOM → recompute summary
- Initializes safely on `DOMContentLoaded` to ensure the month label exists

## Development notes

### External dependencies

This project pulls a couple of assets via CDN (fonts/icons). If you are developing offline, some icons/fonts may not render until you restore network access.

### Data lifecycle

- Data is stored **in memory only** (refresh resets state).
- DOM row IDs (`inc-0`, `exp-3`, …) are used as the bridge between the UI and the model.

### Browser support

The code is written for modern evergreen browsers. It uses classic DOM APIs and should work in current Chrome/Edge/Firefox/Safari.

### Troubleshooting

- **Icons/fonts not showing**: check your network connection (CDN assets) and your browser DevTools console/network tab.
- **Things behave differently than production**: prefer running a local server instead of opening `index.html` via `file://`.
- **Local URL doesn’t load** (e.g. `http://thebudgetapplication.localhost/`): ensure whatever is hosting/serving that domain is running; otherwise use one of the static server options above.

### Common modifications

- **Change styling/layout**: edit `css/style.css`.
- **Update calculations/business rules**: update `budgetController` in `js/app.js`.
- **Change markup/labels**: edit `index.html` and/or the HTML string templates in `UIController`.

## Key interaction flows

### Add item

1. Read UI input (type/description/value)
2. Validate basic constraints (non-empty description, numeric value > 0)
3. Add item to the data model
4. Render the new row into the appropriate list
5. Clear and focus the input fields
6. Recompute totals and repaint the summary

### Delete item (event delegation)

Instead of adding a click handler per row, the app attaches **one** handler to `.container` and uses bubbling to determine which item was targeted. The DOM row IDs (`inc-0`, `exp-3`, …) provide a stable bridge between UI and model.

## Design & implementation details (the “why”)

- **Separation of concerns**: The UI never “calculates” totals; it only displays values returned by `budgetController.getBudget()`.
- **Single source of truth**: All totals derive from the in-memory arrays (`data.allItems.inc/exp`). The DOM is treated as a view.
- **Deterministic IDs**: IDs are predictable, stable per list, and cheap to generate without external dependencies.
- **Composable update step**: `updateBudget()` isolates “recompute + repaint” so both add and delete flows stay consistent.
- **Graceful empty-income state**: When income is 0, percentage shows `---` to avoid misleading output.

## UX and visual design

- Two-column layout for Income vs Expenses
- Strong visual hierarchy in the top summary panel
- Color language:
  - teal for income
  - red for expenses
- Background image + overlay gradient for legibility

## Limitations / polish opportunities (intentional transparency)

This project keeps the implementation intentionally simple. A few areas are good candidates for follow-up work:

- **No persistence**: data lives in memory only; refresh resets the budget.
- **Number formatting**: values are displayed as raw numbers (no currency formatting or thousands separators).
- **Per-expense percentage**: the expense row markup currently includes a placeholder percentage value.
- **Delete targeting**: the delete handler relies on chained `parentNode` traversal, which is fragile if markup changes.
- **Keyboard handling**: it uses `keypress`/keyCode patterns that are considered legacy in modern browsers.

## Contributing

Small, focused improvements are welcome.

- Keep changes framework-free (no bundlers required).
- Preserve the three-module separation in `js/app.js`.
- Prefer readable, explicit DOM code over clever abstractions.

## Deployment

This app can be deployed to any static hosting provider (GitHub Pages, Netlify, Vercel static, S3, etc.).

- Upload `index.html` plus the `css/`, `js/`, and `images/` folders.
- No environment variables or server configuration are required.

## For recruiters / hiring teams

If you’re scanning for signal: this project emphasizes **clean modularization**, **DOM event patterns**, and **predictable state updates** without frameworks.
