# 2020 Budget Application

A lightweight, browser-based budgeting app that lets you add income and expenses and see your remaining budget update instantly.

## Highlights

- **Add transactions**: Create **Income (+)** or **Expense (-)** items with a description and numeric value.
- **Live budget totals**: Automatically recalculates and displays:
  - Total income
  - Total expenses
  - Remaining budget (income − expenses)
  - Expense percentage (expenses ÷ income)
- **Delete transactions**: Remove any income/expense item and immediately see totals refresh.
- **Current month label**: The header month is set automatically from the user’s system date on page load.

## Technical Highlights

- **Module pattern (IIFEs)**: The app is split into three self-contained modules in `js/app.js`:
  - `budgetController` (data + math)
  - `UIController` (DOM reads/writes)
  - `controller` (event wiring + orchestration)
- **Single source of truth for calculations**: Totals and budget are computed from the internal `data` structure (not from the DOM).
- **Deterministic IDs**: Each new item is assigned an ID based on the last item in its category (`inc` / `exp`).
- **Event delegation for deletes**: A single click handler on the `.container` detects delete clicks and removes the right item.
- **Safe init timing**: App initialization runs on `DOMContentLoaded` to ensure the month label and budget labels exist before updates.

## Key Functions (quick scan)

| Area | Function | Responsibility |
|------|----------|----------------|
| Data | `budgetController.addItem(type, des, val)` | Creates an `Income` / `Expense`, assigns an ID, stores it in the data model |
| Data | `budgetController.deleteItem(type, id)` | Removes an item from the data model by type + ID |
| Data | `budgetController.calculateBudget()` | Computes totals, remaining budget, and overall expense percentage |
| Data | `budgetController.getBudget()` | Returns a snapshot of computed values for the UI |
| UI | `UIController.getInput()` | Reads user input from the form controls |
| UI | `UIController.addListItem(obj, type)` | Renders a new income/expense row in the correct list |
| UI | `UIController.deleteListItem(selectorID)` | Removes a rendered row from the DOM |
| UI | `UIController.displayBudget(obj)` | Updates the top summary numbers (budget/income/expenses/percentage) |
| UI | `UIController.displayMonth()` | Sets the month label based on the system date |
| App | `controller.init()` | Initializes month + default totals and wires event listeners |
| App | `updateBudget()` | Orchestrates recompute + re-render after changes |
| App | `ctrlAddItem()` | Handles add-item flow (read input → store → render → clear → update totals) |
| App | `ctrlDeleteItem(event)` | Handles delete flow (find clicked item → remove data → remove UI → update totals) |

## How It Works (at a glance)

- Built with vanilla JavaScript using a simple module pattern:
  - **Budget Controller**: Stores items and computes totals/percentages.
  - **UI Controller**: Reads input, renders list items, and updates totals/month label.
  - **App Controller**: Wires event listeners and coordinates updates.

## Code Walkthrough (How the app works)

### 1) Data model (Budget Controller)

The internal state lives in a `data` object:

```js
data = {
  allItems: { inc: [], exp: [] },
  totals:   { inc: 0,  exp: 0  },
  budget: 0,
  percentage: -1
}
```

- `addItem(type, description, value)`
  - Creates a new `Income` or `Expense` instance
  - Assigns a numeric `id`
  - Pushes it into `data.allItems[type]`
- `deleteItem(type, id)`
  - Finds the matching item by ID and removes it from the array
- `calculateBudget()`
  - Recomputes totals, then:
    - `budget = totalInc - totalExp`
    - `percentage = Math.round((totalExp / totalInc) * 100)` (or `-1` if no income)
- `getBudget()`
  - Returns a snapshot object for the UI to display

### 2) Rendering + DOM updates (UI Controller)

- `getInput()` reads:
  - `.add__type` (inc/exp)
  - `.add__description`
  - `.add__value` (parsed to a number)
- `addListItem(obj, type)`
  - Builds an HTML string for the list item
  - Replaces placeholders (ID/description/value)
  - Inserts it into either `.income__list` or `.expenses__list`
- `displayBudget(budget)`
  - Updates the labels at the top of the page (`budget`, `income`, `expenses`, `percentage`)
- `displayMonth()`
  - Uses `new Date()` + a month-name array
  - Writes the current month into `.budget__title--month`

### 3) App flow (Global Controller)

On page load:

1. `controller.init()` runs (after `DOMContentLoaded`)
2. Calls `UICtrl.displayMonth()` to set the month label
3. Calls `UICtrl.displayBudget(...)` with zeros for the initial state
4. Calls `setupEventListeners()`

When you add an item (button click or Enter):

1. Read form input (`UICtrl.getInput()`)
2. Create and store the item (`budgetCtrl.addItem(...)`)
3. Render the row (`UICtrl.addListItem(...)`)
4. Clear the input fields (`UICtrl.clearFields()`)
5. Recalculate + redraw totals (`updateBudget()` → `budgetCtrl.calculateBudget()` → `UICtrl.displayBudget(...)`)

When you delete an item:

1. The click bubbles to the container handler
2. The code reads the row `id` like `inc-0` / `exp-3`
3. Removes from data (`budgetCtrl.deleteItem(type, id)`)
4. Removes from DOM (`UICtrl.deleteListItem(rowId)`)
5. Recalculates + redraws totals (`updateBudget()`)

## Run Locally

### Option A: Open directly
1. Open `index.html` in your browser.

### Option B: Serve as a static site (recommended)

Use any simple static server so asset paths behave consistently.

- Node: `npx http-server` (run from the project folder)
- Python: `python -m http.server` (run from the project folder)

## Tech

- HTML / CSS
- Vanilla JavaScript (no frameworks)

## Project Structure

- `index.html` — App layout and script entry
- `css/style.css` — Styling
- `js/app.js` — Application logic

## Notes

- Values are entered as numbers and calculations update immediately after adding or deleting items.
- The expense percentage is shown only when income is greater than zero.
