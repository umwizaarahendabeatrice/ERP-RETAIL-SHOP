# Duka ERP: Retail Prototype

A lightweight ERP prototype for a neighbourhood grocery shop, built for the **Midterm Project: Generative AI-Assisted ERP System Design & Reflection**. It integrates two core ERP modules on a single database:

- **Sales & Inventory** (Materials Management): products, customers, sales, purchases, stock movements
- **Financials** (General Ledger): chart of accounts, double-entry journal, trial balance

A single business event updates both modules. For example, completing a sale reduces stock *and* posts revenue and cost-of-goods-sold entries to the ledger in one atomic transaction.

**Author:** _[your name]_  **Course:** _[course name and code]_  **Date:** _[date]_

---

## Contents

| File | Purpose |
|---|---|
| `retail-erp.html` | The runnable prototype (single file, no install). Contains the UI, business logic and data store. |
| `schema.sql` | Production-style PostgreSQL / Supabase schema with constraints and triggers. |
| `DOCUMENTATION.md` | System architecture, ER diagram, data flow, validation controls, CRUD matrix, AI Support Statement, Prompt Log. |
| `tests/run-tests.js` | Automated test of the business logic (31 checks). |
| `README.md` | This file. |

## Quick start

**Option A: run the prototype (recommended)**

1. Download `retail-erp.html`.
2. Double-click it to open in any modern browser (Chrome, Edge, Firefox, Safari). No server, install or internet connection is needed.
3. The app loads with sample data. Data is saved in your browser (localStorage) and survives refreshes. Use **Dashboard → Reset to sample data** to start over.

**Option B: deploy the schema on Supabase**

1. Create a free project at supabase.com.
2. Open **SQL Editor**, paste the contents of `schema.sql` and run it.
3. Insert into `purchases`, then `sales` and `sale_items`, and watch `products`, `stock_movements` and `journal_lines` update through triggers.

> Status: `schema.sql` is the database-level version of the same design. _[Update this line after you run it: e.g. "Tested on Supabase on (date), ran without errors" or list what you had to fix.]_

**Run the automated tests (optional, needs Node.js)**

```
node tests/run-tests.js
```

Expected result: `31 passed, 0 failed`.

## Features

| Area | What you can do |
|---|---|
| Dashboard | Cash, revenue, gross profit, stock value; reorder alerts; integrity checks |
| Inventory | Create, edit and delete products; view recent stock movements |
| Customers | Create, edit and delete customers; see lifetime sales |
| Sales | Build a multi-item sale (cash or mobile money); view history; **void** a sale |
| Purchasing | Receive stock at a unit cost (average cost is recalculated) |
| Ledger | Trial balance and the latest journal lines (read-only) |

## How the modules are integrated

| Business event | Inventory effect | Financial effect |
|---|---|---|
| Sale completed | Stock down, `sale` movement | Dr Cash / Cr Sales Revenue; Dr COGS / Cr Inventory |
| Sale voided | Stock restored, `void` movement | Both entries reversed |
| Stock received | Stock up, weighted-average cost updated | Dr Inventory / Cr Cash |

Each event is all-or-nothing: if any step fails (for example, insufficient stock), no part of it is saved.

## Data integrity (single source of truth)

- SKUs and customer phone numbers are unique.
- Stock can never go negative; overselling is rejected.
- Stock quantity and average cost cannot be typed in; they change only through purchases and sales.
- Every journal entry must balance (debits equal credits).
- Records with history cannot be deleted; sales are voided with reversing entries instead.
- Dashboard check: the Inventory account in the ledger matches stock on hand valued at average cost.

Full detail is in `DOCUMENTATION.md`.

## Sample data

| Item | Price (UGX) | Avg cost (UGX) | Opening stock | Reorder level |
|---|---|---|---|---|
| Rice 5kg | 28,000 | 22,000 | 40 | 10 |
| Cooking oil 3L | 24,000 | 19,000 | 25 | 10 |
| Sugar 1kg | 5,500 | 4,200 | 60 | 20 |

Opening capital is UGX 5,000,000. Starting cash is therefore UGX 3,393,000 and starting stock value is UGX 1,607,000. Two sample customers are included.

## Manual test script

1. Sell 5 Rice 5kg. Rice stock becomes 35, revenue 140,000, gross profit 30,000.
2. Try to sell 999 Cooking oil. It is rejected.
3. Void the sale. Stock and ledger return to the starting values.
4. Receive 40 Rice at 26,000. Average cost becomes 24,000 and stock 80.
5. Add a product with an existing SKU. It is rejected.
6. Delete a customer who has a sale. It is rejected.
7. Refresh the page. Data is still there.

## Known limitations

- The browser version has **no login, roles or multi-user concurrency**, and data lives in one browser only.
- Sales are paid in full at the time of sale (no credit or receivables); no tax or returns of damaged goods.
- COGS is rounded to whole shillings, which can cause a tiny difference between ledger and stock value.
- Deleting a seeded product with no sales is blocked, and the error message says "sales history" even though the cause is its opening purchase record.
- The SQL version posts one journal entry pair per sale line rather than per sale.
- Prototype scale only; see `DOCUMENTATION.md` for the enterprise considerations.

## Technology

Plain HTML, CSS and JavaScript (no frameworks or build step); browser localStorage for persistence; PostgreSQL (Supabase) for the production-style schema. Chosen so the prototype is free, installs nothing and can be reviewed by opening one file.

## AI Support Statement

Generative AI (Claude) was used for architecture, schema and code generation, and for testing support, under the course policy. The full statement and prompt log are in `DOCUMENTATION.md`. The reflection paper was written by the author without AI assistance.
