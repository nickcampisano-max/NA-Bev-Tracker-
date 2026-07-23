# NA Bev Variance Tracker Changelog

All notable changes to the NA Bev Weekly Variance Tracker are documented here.

---

## [v1.1] — 2026-07-23
### Added
- **Dark theme redesign** — full visual overhaul from the original light theme to a dark, teal-accented palette (glowing buttons, gradient header text), matching the tool's primary usage context (GitHub Pages, not the Cowork sidebar).
- **Period 8 seeded for Central** — Period 8 (Jul 6 – Aug 2, 2026) pre-loaded with beginning on-hand counts pulled from the 7/5/26 physical inventory sheet (Period 7's ending count): Coke 43, Diet Coke 41, Sprite 33, Dr Pepper 57, Root Beer 5, Lemonade 3.4 gal. Enables count reconciliation from day one instead of starting at zero.
- **CSV Import** — each week now has an "Import CSV" control that reads a Toast product-mix export (Item / Qty sold / Discount amt columns), matches rows to tracked items by normalized name (case/punctuation-insensitive, so "Dr Pepper" and "Dr. Pepper" both match), and shows a preview of current → new Sold and Discount values before committing. Adds to the week's existing numbers rather than overwriting, so multiple channel exports can be imported into the same week without conflicting. Unmatched CSV rows are listed rather than silently dropped. Purchased quantities are still manual (that data only exists in the MarginEdge purchase report, not the sales export).

### Fixed
- **Import CSV button doing nothing on click** — the original implementation created a hidden file input and triggered it via a scripted `.click()`, which some browsers silently block for file inputs as a security measure. Replaced with a native `<label for>` + `<input type=file>` pair, the standard cross-browser-safe pattern for custom-styled upload buttons.

### Removed
- **Floating soda-can background animation** — added briefly as part of the dark theme redesign, then removed per request. Dark theme kept, decorative can graphics/animation fully stripped out.

---

## [v1.0] — 2026-07-22
### Added
- **Initial build.** Weekly purchased-vs-accounted-for reconciliation tool for NA Bev, built as a companion to the Fellow Bar Period Tracker but scoped narrower: this tracks whether purchased product is showing up as a transaction, not purchasing/ordering budgets.
- **Multi-store structure** — three stores configured: Clever Koi Central Phoenix (incl. ATP), Clever Koi Gilbert, Fellow Scottsdale. Central Phoenix pre-seeded with item list, pricing, and costs from initial variance investigation (Jun 8–Jul 5 period); Gilbert and Scottsdale ship empty, configurable in Settings.
- **Period / Week structure** — mirrors the bar tracker's 28-day period, 4-week layout. Each store tracks its own periods independently. "+ New Period" carries forward expected on-hand as next period's beginning inventory.
- **Per-item weekly entry** — Purchased, Sold (all channels combined), Discount $, Kitchen use (manual override, e.g. an item used in a recipe), and free-text Notes.
- **Auto-calculated columns** — Comped units (discount $ ÷ price, informational only — already included in Sold, not additive), Accounted for (Sold + Kitchen use), Unaccounted (Purchased − Accounted for), Unaccounted cost, % accounted.
- **Flagging** — 🟢 ≥85% accounted, 🟡 70–84%, 🔴 <70%. A "2+ wks" sustained-drift badge only appears when an item is red for two consecutive weeks, to avoid flagging single noisy weeks (small-sample items like Root Beer).
- **Physical count reconciliation** — optional "Count" field per item per week. Entering an actual cooler count resets the running expected-on-hand total for that item, so report-based drift can be checked against reality.
- **Period Summary** — per-item totals across all 4 weeks, plus KPI tiles: unaccounted units, unaccounted cost, retail value if those units had sold at menu price, and gross margin never captured (retail value minus product cost already spent).
- **Copy Email** button — generates the tightened variance-email format (item table, ruled-out list, bottom-line $ impact, comp-inclusion disclaimer) and copies it to clipboard.
- **Compare Stores view** — side-by-side table and bar chart of % accounted and unaccounted cost across all configured stores, for each store's active period. Surfaces whether a problem item is store-specific or systemic.
- **Settings** — per-store item configuration (name, unit type, price, cost, pour size for liquid items, default weekly kitchen-use qty). Reset option clears all weekly/period data while preserving item configuration.
- **localStorage persistence** — all data auto-saves under `nabev_tracker_v1` key, no backend required.

### Deliberately not included (see conversation for reasoning)
- No order-quantity forecasting, pour-cost purchasing targets, YoY comparison, or scenario planner — those solve "how much to buy to hit a cost %," which is the bar tracker's job. This tool answers "is what we bought showing up as a sale," a different question.

---

## Math Reference

### Purchased, in serving-equivalent units
```
Can/unit items:  purchasedServings = purchased (as entered)
Gallon items:    purchasedServings = (purchased gallons × 128oz) ÷ pour size (oz)
```

### Comped units (informational — already inside "Sold")
```
Comped = Discount $ ÷ Price
```

### Accounted For
```
Accounted For = Sold (all channels, includes comped drinks rung into POS) + Kitchen Use
```

### Unaccounted
```
Unaccounted = Purchased (serving-equivalent) − Accounted For
Unaccounted Cost = Unaccounted × Cost per serving
```
Cost per serving for gallon items = Cost per gallon ÷ (128 ÷ pour size in oz)

### % Accounted / Flag thresholds
```
% Accounted = Accounted For ÷ Purchased
🟢 green  ≥ 85%
🟡 yellow   70–84%
🔴 red    < 70%
```
Sustained-drift badge: current week AND prior week both red.

### Expected On-Hand (for count reconciliation)
```
Expected On-Hand = Beginning Inventory + Σ(Purchased − Accounted For) through the current week
```
Entering an Actual Count for a week resets Expected On-Hand to that value going forward.

### Retail Value / Margin Never Captured
```
Retail Value = Unaccounted units × menu price
Margin Never Captured = Retail Value − Unaccounted Cost
```
(Unaccounted Cost was already spent regardless of whether the product sold; Margin Never Captured is the actual P&L impact.)
