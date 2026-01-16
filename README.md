# 2020 Budget Application (Vanilla JS)

A lightweight budgeting UI built with **plain HTML/CSS/JavaScript**. Users can add **income** and **expense** line items and see the budget summary update immediately.

This repo is designed as a portfolio-friendly example of:

- clean separation between **data/model**, **UI rendering**, and **application orchestration**
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

## Architecture (developer view)

All application logic lives in `js/app.js` and is intentionally split into three modules:

1) **`budgetController` (model + calculations)**

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

2) **`UIController` (DOM reads/writes)**

- Centralizes query selectors in a `DOMStrings` map
- Reads form data via `getInput()` and parses numeric values with `parseFloat`
- Renders list items by building HTML strings and inserting them with `insertAdjacentHTML`
- Updates the “budget header” UI in one method (`displayBudget`)
- Sets the month label via `displayMonth()` (uses `new Date()` + month name lookup)

3) **`controller` (wiring + orchestration)**

- Owns event listener registration
- Defines the two main user flows:
  - `ctrlAddItem()` — validate → add to model → render → clear fields → recompute summary
  - `ctrlDeleteItem()` — identify clicked row → delete from model → delete from DOM → recompute summary
- Initializes safely on `DOMContentLoaded` to ensure the month label exists

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

## Run locally

### Option A — Open directly

- Open `index.html` in your browser.

### Option B — Serve as a static site (recommended)

- Node: `npx http-server`
- Python: `python -m http.server`

Then open the printed local URL.

## Repository layout

- `index.html` — page structure and script entry
- `css/style.css` — styling
- `js/app.js` — application logic
- `images/` — background image asset

## For recruiters / hiring teams

If you’re scanning for signal: this project emphasizes **clean modularization**, **DOM event patterns**, and **predictable state updates** without frameworks.
