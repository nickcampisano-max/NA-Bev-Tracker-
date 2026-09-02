# NA Bev Variance Tracker Changelog

All notable changes to the NA Bev Weekly Variance Tracker are documented here.

---

## [v2.2] — 2026-09-02
### Fixed
- **Lemonade's stray Kitchen use values (Period 9, Weeks 2–3) — root cause confirmed and corrected.** These were real Arnold Palmer conversions (Week 2: 4 sold; Week 3: at least 5, possibly imported in two passes) that got routed to Kitchen use back when that recipe link's `addsTo` was still `'kitchen'` and unrounded — before the v1.2 fix. The saved item config already said `addsTo:'sold'` by the time this was investigated, confirming the fix never touches historical week data, only new imports going forward. Corrected by moving each week's Kitchen value into Sold (rounded to match how the current code would have recorded it) and zeroing Kitchen use. Kitchen use is now genuinely $0/0 units across every period — this was a one-time data correction in Nick's saved state, not a code change (no source file involved).

### Added
- **Staff comp rate & Revenue realized** — two new metrics on the Period Summary, prompted by Nick wanting to see profit (guest-paid sales) vs. perk (staff comps) separately, not blended into one "accounted for" number. Built entirely from existing fields, no new data collection: comp units = discount $ ÷ menu price (already computed as `comped`); revenue realized = billed $ ÷ what the same units would bill at full price. Deliberately two separate percentages rather than one, since a comped unit is a full unit but zero revenue — they diverge on purpose. Two new period-level KPI cards, plus Comp % / Revenue % columns added to the existing per-item Period Summary table (not the weekly Variance table, to avoid re-adding the column clutter just removed). Confirmed with Nick beforehand that the "Discount $" field is staff-comp-only for these items (no other discount type touches NA bev), so no separate Toast reason-code report is needed for this to be accurate.

### Investigated (resolved)
- Kitchen use mystery from v2.1 — see Fixed, above.

---

## [v2.0] — 2026-08-31
### Changed
- **Variance table redesigned for clarity**, after Nick flagged that a green "OK" flag sitting right next to an orange "⚠ LARGE" badge read as a flat contradiction, and that "% acct." (104% for Coke) didn't explain what it was measuring. Root issue: the table was mixing two different time windows — This Week (the flow check: did what got bought this week get sold) and Running Total (the accumulated pile since period start) — with no visual distinction between them.
  - Columns now grouped under two header bands: teal "This week" (Purchased, Sold, Discount $, Comped, Kitchen use, Accounted for, % of purchases, Flag) and orange "Running total (since period start)" (Unaccounted, Unacct. cost).
  - The "⚠ Large" badge moved off the Flag cell (This Week) onto the Unacct. cost cell (Running Total) — it's describing that number, not contradicting the flag next to it.
  - "% acct." renamed to "% of purchases†" with a tooltip spelling out the formula (this week's Accounted for ÷ this week's Purchased, excluding carried-over stock).
  - Footnote rewritten to explain plainly why the two groups can legitimately disagree — a green flag with a "⚠ Large" badge means this week's buying-and-selling looks fine, but there's still a real backlog sitting unaccounted overall.

---

## [v1.9] — 2026-08-31
### Added
- **"⚠ LARGE" alert badge**, independent of the weekly OK/WATCH/FLAG. The weekly flag only ever checks this week's purchased-vs-sold ratio (by design, so a zero-purchase week with real sales isn't a false alarm) — which means a large real pile of unaccounted product can sit there indefinitely, green the whole time, as long as purchasing keeps roughly pace with sales each week. Caught live: Coke Week 3 of Period 9 showed 82 units / $78.72 unaccounted with a green "OK" flag, because 50 sold against 48 purchased that week is 104%. The new badge is a second, independent check — any week where an item's unaccounted *cost* is at or above a configurable threshold (default $50) gets "⚠ LARGE" next to whatever the weekly flag already says. Editable in Settings under a new "Alerts" card. Migrates automatically for existing saved data (defaults to $50 if missing).

---

## [v1.8] — 2026-08-31
### Changed
- **Store tab renamed to "Sales vs Purchasing."** With only Central Phoenix left, a tab labeled with the store's name told you nothing about what was on it — every other tab (Order Guide, Compare Periods, Settings) already describes its function, so this one now matches. Label only; the store's actual name is unchanged everywhere else (Order Guide header, email subject line, Compare Periods).
- **"Compare Stores" → "Compare Periods."** The old view showed one row per store's latest period — with one store, that's a single row and no longer a comparison of anything. Repurposed to show one row per *period* for Central Phoenix instead (Period 8, 9, 10...), so % accounted and unaccounted cost can be tracked trending over time. Verified live: Period 8 → Period 9 shows % accounted improving from 51% to 69%.

---

## [v1.7] — 2026-08-31
### Added
- **"Ending on-hand" card** on the Variance view, always visible (not gated behind clicking into the Week 4 tab) — Nick went looking for P9's ending count after closing it out and couldn't find it, since it previously only existed as Week 4's Count column. Not a new/separate field: it reads and writes the exact same `period.weeks[3][item.id].actualCount` data as Week 4's Count column, so editing either place updates both — there's one number, just two places to see and fix it. Verified live: editing a value here updates Week 4's Count column instantly, and vice versa.

---

## [v1.6] — 2026-08-31
### Fixed
- **Typing a multi-digit number into the Variance week table silently dropped every character after the first** (e.g. typing "49" into Count only ever saved "4"). Root cause: that table's inputs re-rendered the entire page on every keystroke (to recompute Accounted for/Unaccounted/Flag/Total live), which destroyed and rebuilt the very input being typed into, losing focus and the rest of the keystrokes. Switched from committing on `input` (every keystroke) to `change` (blur/Enter) — computed columns still update, just once you leave the field instead of mid-keystroke. Caught while entering Period 9's Week 4 physical count; the real counts (Coke 49, Diet Coke 43, Sprite 52, Dr Pepper 68, Root Beer 10, Lemonade 0) had to be entered directly rather than through the (at-the-time-still-buggy) UI.
- **"+ New Period" carried a meaningless computed beginning inventory forward for order-only items.** Passport coffee/tea items never participate in the purchased-vs-accounted flow, but a purchase CSV can still incidentally match one of them (e.g. a broader MarginEdge report with a "Coffee, Whole Bean" line lands on the item now that it's tracked) — carrying that through `expectedOnHand` produced a stray, meaningless "beginning inventory" for an item that was never actually reconciled. `+ New Period` now skips order-only items when carrying forward. Period 10 (created same day) had this stray value stripped from its already-created `beginInv` directly, without touching the underlying Week 4 purchase record it came from — that's real purchase data (Nick did buy the coffee), just not one this tool tracks usage for.
- Note for future CSV imports: since the 8 Passport items now exist in the tracked item list, a MarginEdge purchase report that includes coffee/tea lines will now match and fill their `purchased` field (previously "unmatched"). Harmless — order-only items never appear in the Variance table or feed any calculation — but worth knowing if a purchased number shows up somewhere unexpected.

---

## [v1.5] — 2026-08-24
### Changed
- **Removed the Gilbert and Scottsdale stores.** Both were always empty (no items, no data) — the tracker now covers Clever Koi Central Phoenix only. A migration (`pruneRemovedStores`) drops them from any browser's already-saved localStorage too, so old tabs don't linger.
- **Order Guide reworked to match Gary's real ordering process**, after reviewing his live "CKC + ATP NA BEVERAGE ORDER GUIDE" sheet — it surfaced that the first version's weekly-cadence assumption was wrong:
  - Items are grouped by vendor (Shamrock, Passport), each with its own table.
  - Order logging is now per real order window, not per calendar week — Shamrock (sodas/lemonade) gets three columns (Tue-for-Thu, Thu-for-Sat, Sat-for-Mon), Passport (coffee/tea) gets one (Tue-for-the-week). The `orders` data shape changed from one `{qty,date}` per item per week to an array of them, sized to the item's vendor.
  - Added a Size column (`sizeLabel` on each item, e.g. "12 oz (24/cs)") mirroring his sheet's SIZE column.
  - Added 8 new order-only items for Passport (Coffee Decaf/Espresso/Whole Bean, 4 teas, tea filters) — no sales data exists for these, so they show "—" for avg usage/par/suggested (same as blank cells on his sheet) but still get order-window logging.
  - The "Suggested" quantity stays a whole-week figure (par − on-hand), not split per order window — real sales data only arrives at weekly granularity, so a per-window forecast would be false precision. Logging is per-window; the suggestion is a weekly directional guide.
  - Corrected 3 item costs to match his sheet, which is his live vendor-pricing reference: Dr Pepper 0.99→0.96, Root Beer 0.68→0.71, Lemonade 8.08→8.18. This only affects newly-seeded browsers — an existing browser's saved cost isn't overwritten by migration, so already-deployed data needs the same correction applied once via Settings (or directly, if done for Nick's live tab already).
  - `migrateOrders()` now also upgrades the very first Order Guide version's single-slot order format into the new per-window array format, padding new slots blank rather than guessing.

---

## [v1.4] — 2026-08-24
### Added
- **Order Guide view** — a new tab, separate from the Variance view on purpose (see reasoning below). Built for Gary (area director), who's now taking over NA Bev ordering from Nick. Per item, per week: trailing average usage (Sold + Kitchen use, last up to 4 weeks with real data, spanning prior periods so a new period doesn't reset to nothing), a suggested par target (avg usage × a per-item multiplier, default 1.3x, editable in Settings), estimated current on-hand (the same running-balance math as the Variance view's carryIn, but through-and-including the current week rather than entering it — the freshest snapshot available), and a suggested order quantity (par − on-hand, floored at zero). Also logs what was actually ordered (qty + date) per item per week, shown as "Last order" starting the following week — kept separate from `purchased` (what MarginEdge shows arrived) since an order and what shows up can legitimately differ.
- Shares item config (name, unit, price, cost, par multiplier) and period/week structure with the Variance view — one config to maintain, not two. Deliberately kept as a separate view rather than merged into the Variance table: the Variance view is backward-looking accountability (did purchased match sold), the Order Guide is forward-looking decision support (how much to buy next) — the tracker's own v1.0 changelog entry ruled out mixing these into one tool, and that's still the right call.
- "Real time" here means "as of whatever's been imported/entered so far" — there's no live POS feed, so the on-hand figure is a snapshot, not literally live. Labeled as such in the view.

### Data model
- Added `parMultiplier` to item config (default 1.3, per-item, editable in Settings) and a new `orders` array on each period (mirrors the `weeks` array shape — one entry per week, per item: `{qty, date}`). Both are backfilled automatically for existing saved data via `migrateOrders()`, following the same pattern as `migrateItemConfig()`.

---

## [v1.3] — 2026-08-13
### Fixed
- **"+ New Period" corrupted gallon items' carried-forward beginning inventory.** It carried forward `expectedOnHand` (a servings-equivalent number) directly into the new period's `beginInv` field, which is stored in each item's native unit — fine for cans (1 unit = 1 serving), but wrong for Lemonade, where it wrote a servings number into a gallons field and understated the true carried-forward stock by ~21x. Added `servingsToNative(item, servings)`, the inverse of the existing `purchasedServings()`, and routed the "+ New Period" handler through it.
- Period 9 for Central was started directly from Period 8's verified 8/2/2026 ending inventory (Coke 14, Diet Coke 27, Sprite 43, Dr Pepper 22, Root Beer 11, Lemonade 0.7 gal) rather than a fresh physical count, per Nick's call that the just-closed P8 count was already accurate enough to carry forward.

---

## [v1.2] — 2026-07-23
### Changed
- **Arnold Palmer's lemonade use now counts as Sold, not Kitchen use.** It's a drink guests order, not an internal recipe like the apple dish — Kitchen use is for non-guest-facing consumption. The CSV import preview and confirm logic now route a recipe link to either Sold or Kitchen use per-item (`addsTo` on the recipe config), defaulting to Sold since most recipe links will be guest drinks like this one.
- **Unaccounted now factors in stock already on hand, not just this week's Purchased.** Previously, an item you didn't restock that week (but still had plenty of standing inventory for) always showed a negative, alarming-looking Unaccounted number — even though nothing was actually wrong. It's now Beginning On-Hand (or the running balance carried out of prior weeks) + this week's Purchased, minus Accounted For. The % Accounted and red/yellow/green flag are unchanged — they still key off this week's Purchased alone, which already correctly treats a zero-purchase week with real sales as 100%/green. Period Summary, Compare Stores, and the email export still total each week's own Purchased-vs-Accounted flow rather than this running balance, since that's the correct period-level total either way.
- **Recipe-conversion amounts (like Arnold Palmer → Lemonade servings) round to 1 decimal** instead of carrying full floating-point precision (e.g. 4.666666666666667 → 4.7).

### Fixed
- **Beginning inventory and physical Count for gallon items (Lemonade) weren't being unit-converted** before being used in the running on-hand calculation — they were entered in gallons but combined directly with servings-based Purchased/Sold math, understating the real carried-forward balance. Both now convert through the same gallons→servings math as Purchased.

---

## [v1.1] — 2026-07-23
### Added
- **Dark theme redesign** — full visual overhaul from the original light theme to a dark, teal-accented palette (glowing buttons, gradient header text), matching the tool's primary usage context (GitHub Pages, not the Cowork sidebar).
- **Period 8 seeded for Central** — Period 8 (Jul 6 – Aug 2, 2026) pre-loaded with beginning on-hand counts pulled from the 7/5/26 physical inventory sheet (Period 7's ending count): Coke 43, Diet Coke 41, Sprite 33, Dr Pepper 57, Root Beer 5, Lemonade 3.4 gal. Enables count reconciliation from day one instead of starting at zero.
- **CSV Import** — each week has a drag-and-drop dropzone (drop a file on it, or click to browse) that reads either a Toast sales export (Item / Qty sold / Discount amt → fills Sold and Discount) or a MarginEdge purchase report (Product / Purchased Units → fills Purchased). It sniffs the header row to tell them apart automatically, so there's one drop target for both file types rather than two to keep straight. Matches rows to tracked items by normalized name (case/punctuation-insensitive), with a substring fallback for MarginEdge's more verbose naming ("Soda, Diet Coke 12oz Can" still resolves to "Diet Coke," not "Coke," by preferring the longest/most specific match). Shows a preview of current → new values before committing, and adds to the week's existing numbers rather than overwriting, so multiple files (e.g. two channel exports, or a mid-week invoice on top of an earlier one) can be imported into the same week without conflicting. Unmatched CSV rows are listed rather than silently dropped.

- **Arnold Palmer → Lemonade recipe link** — Arnold Palmer isn't purchased or sold as its own tracked item, but it uses 4oz of lemonade per drink. Sales CSV import now recognizes "Arnold Palmer" rows, converts the quantity sold to a servings-equivalent (4oz ÷ Lemonade's 6oz pour size), and adds it to Lemonade's Kitchen use for that week automatically instead of leaving it as an unmatched row. Shown as a distinct "Recipe use detected" section in the import preview so it's clear where the number came from. Configured per-item via a `recipeUsage` list on Lemonade, so more links (e.g. other mixed drinks) can be added the same way later.

### Fixed
- **Import CSV button doing nothing on click** — the original implementation created a hidden file input and triggered it via a scripted `.click()`, which some browsers silently block for file inputs as a security measure. Replaced with a native `<label for>` + `<input type=file>` pair, the standard cross-browser-safe pattern for custom-styled upload buttons.
- **Week tabs — and every other button in the app — not responding to clicks.** Took three passes to actually fix.
  - First pass converted all 14 `.onclick =` assignments to `.addEventListener('click', ...)`. Didn't fix it.
  - Second pass introduced a `wireClick(el, fn)` helper (stores the handler as a plain `__clickHandler` property) backed by a single delegated `click` listener on `document`. This looked right and passed an isolated test, but still didn't fix it on the real page.
  - **Actual root cause**, found by comparing a button's state immediately after `wireClick` ran versus after `render()` finished: `wcard.innerHTML += '<div class="info-note">...</div>'`, used to append a footnote after the week tabs were already built and wired, was the problem. `innerHTML +=` re-serializes an element's *entire* existing content to a string and reparses it from scratch — every child, including the week tab buttons, was being destroyed and rebuilt on every single render, silently wiping out whatever click-wiring mechanism was attached to them (this is true regardless of whether that mechanism is `.onclick`, `addEventListener`, or a custom property — which is why the first two passes didn't help). Fixed by replacing both `innerHTML +=` call sites in the app with `insertAdjacentHTML('beforeend', ...)`, which appends new HTML without touching existing children. Confirmed with a targeted reproduction of the app's exact click-wiring pattern showing the old code replaces the button node (handler lost) and the new code leaves it untouched (handler and a real dispatched click both survive).
- **Drag-and-drop not registering** — the drop target was only the small dashed box, so a slightly imprecise drop landed outside it and the browser's default behavior (navigate away to display the raw file) fired instead of the importer. The whole week card is now the drop target, and a window-level dragover/drop guard prevents that default navigation anywhere on the page.

### Fixed (cont'd)
- **Sprite's auto-filled Kitchen use (7/week, "apple dish") was double-counting.** Turns out the manager already comps those units at POS, so they were already showing up in Sold via the discount — adding another 7 as Kitchen use on top counted the same product twice. Removed the default; Sprite's Kitchen use is now 0 unless entered manually.
- **Arnold Palmer → Lemonade recipe link silently breaking on already-saved browser data.** The link was added to the item's config in code, but a browser that had already used the app before that point had its own saved copy of the Lemonade item without it — and the app never backfilled new config fields into old saved data, so Arnold Palmer rows fell into "unmatched" instead of adding to Lemonade's Kitchen use. Added a migration step in `load()` that backfills any item-config field present in the current code's defaults but missing from a saved item, matched by id, without touching fields the saved item already has. This is a general fix — it'll also cover the next config field added to an item down the road, not just this one.

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
