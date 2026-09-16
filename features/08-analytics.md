# 8. Analytics

**Status:** in-review

Added by QA (2026-08-27) — confirmed to exist as a feature, not yet
described in detail. Analyzed against Jira 2026-09-14 (QVD-73 epic,
QVD-81 Portal spec, QVD-69 backend API — see Related Jira Tickets below),
then **live-verified 2026-09-14 via Playwright MCP** against store
`test-vl-1` — both Portal (dashboard structure/data) and SDK (actual
network traffic from a real storefront view + Add to cart, captured via
Playwright's network inspector). Structure and data matched the ticket
closely; the SDK trace also surfaced a real, unresolved discrepancy
between events firing successfully and the dashboard not reflecting them
— see Business Rules and Edge Cases.

**Cross-reference:** [QVD-115](https://ownego.atlassian.net/browse/QVD-115)
adds an add-to-cart (ATC) click count (last 30 days) directly onto the
[Onboarding Dashboard](01-onboarding-dashboard.md) — a metric hosted there,
not necessarily part of a broader Analytics section/tab. Confirm during
live-testing whether the Dashboard's ATC count is the same underlying
metric as this page's "Add to cart clicks," or a separate/simplified one.

**Second cross-reference (found analyzing AI Offer Creation, 2026-08-27):**
[QVD-173](https://ownego.atlassian.net/browse/QVD-173) builds a backend
event/metrics pipeline (star distribution, response rate, copy rate, etc.)
for the [Ask for Review](09-ask-for-review.md) flow specifically — this is
an **AI-review funnel analytics** pipeline, narrower than and separate from
this "Analytics" feature (which QVD-81/QVD-69 confirm is about discount
offer performance, not review-funnel metrics). Confirmed as two genuinely
separate data sources, not shared infrastructure.

**Third cross-reference:** [QVD-179](https://ownego.atlassian.net/browse/QVD-179)
(Sidekick app extension, In Progress) surfaces offer performance data
(views, ATC, conversion, revenue) inside Shopify's own Sidekick assistant —
see [AI Offer Creation](02-ai-offer-creation.md). QVD-176/QVD-177 (Sidekick
data extension sub-tasks, still In Progress as of this analysis) are
literally named `get_offer_performance` / `get_discount_planning_suggestion`
— strongly suggests Sidekick reads from the **same** order-analytics API
this page uses (QVD-69's endpoints), rather than a separate pipeline.
`Need confirm` once QVD-176/177 ship.

**Fourth cross-reference, found analyzing this feature (2026-09-14):**
[QVD-164](https://ownego.atlassian.net/browse/QVD-164) "[BE] Trigger
analytics API" and [QVD-160](https://ownego.atlassian.net/browse/QVD-160)
"Implement Trigger Chat Scenarios for In-App Customer Support" are a
**completely separate, internal analytics system** — tracking how often
the app's own in-app chat/support trigger scenarios fire (`welcome`,
`creating_offer`, `inactive_user`, `customize_request`), viewable by
Ownego admins in an internal Filament tool ("Analytics tools → Trigger
Analytic Overview"), not by merchants at all. Not part of this
merchant-facing Analytics page — mentioned only so it isn't confused with
it later.

## Purpose

A **standalone, global Portal page** ("Analytics" in the app's left nav,
alongside Manage offers/Translations/Variant swatches) that lets a
merchant track their discount campaigns' performance and understand
customer behavior — how Quantity Break/Multibuy/Wholesale offers are
contributing to sales, with a date-range selector and a set of metrics
covering views, add-to-cart, conversion, orders, revenue, and top
performers.

## Business Rules

### Time range selector — confirmed live 2026-09-14, matches ticket plus extra detail

- Preset options — **confirmed live, exact match:** **Today**,
  **Yesterday**, **Last 7 days** (default), **Last 30 days**,
  **Last 90 days**. A 6th option, **"Custom"**, also exists in the
  dropdown but renders **disabled** (grayed out) — not mentioned in the
  ticket at all.
- **The dropdown itself doubles as a full custom date-range picker**,
  not documented in QVD-81: opening it reveals explicit **Starting**/
  **Ending** date text fields (`yyyy-mm-dd`) plus an inline calendar
  (month grid, prev/next month navigation), pre-filled to match whichever
  preset is selected. Dates beyond today are disabled on the calendar.
  The text **"Maximum range is 90 days"** is shown directly under the
  calendar — the backend's 90-day cap (QVD-69) **is** surfaced to the
  merchant in the UI, not just silently enforced server-side.
  `Cancel`/`Apply` buttons confirm or discard a manual range selection.

### Metrics — confirmed live 2026-09-14 with real data (`test-vl-1`, "Last 7 days")

| Metric (live label) | Live value | Notes |
|---|---|---|
| View | `8` | Ticket calls it "Views" (plural); live heading is singular "View" |
| Add to cart click | `2` | Ticket: "Add to cart clicks" (plural); live: singular |
| Conversion rate | `25.00%` | **Exactly matches the ticket's formula**: `2 / 8 × 100 = 25%` |
| Order with offers | *(no number shown)* | Ticket: "Orders with offers" (plural); heading rendered live but **no numeric value or chart appeared under it** — contrast with Additional revenue below, which did show a number. `Need confirm` whether this is a real bug (missing 0-state) or just this session's data happened to omit it |
| Additional revenue | `0.00 USD` | Shows a real number (zero, consistent with no completed orders in range) — chart itself not visible in the accessibility snapshot (likely a canvas/SVG chart library, expected) |
| Best-selling discounted items | `No data available` | Empty-state text, not documented in ticket |
| Most popular discount tiers | `No data available` | Same |

- All 4 top-row metric headings match the ticket exactly in meaning,
  just **singular vs. plural naming** differs live vs. ticket text
  throughout — a minor, consistent pattern (View/click/Order, not
  Views/clicks/Orders).
- "Views" and "Add to cart clicks" are logged as discrete **events** (see
  backend API below) — not derived after the fact from order data, so
  they should reflect customer interactions even when no purchase
  results (e.g. a view that never converts still counts as a view). The
  live 8 views / 2 ATC / 0 orders pattern is consistent with this (views
  and ATC clicks accumulated from this session's own testing, with no
  completed checkout yet).

### Backend API (QVD-69) — informs what's actually measurable, not a Portal UI spec

- **Timezone handling:** frontend must send `start`/`end` as ISO 8601
  with offset in the **merchant's own timezone** (e.g.
  `2025-09-30T00:00:00+07:00`); responses come back in that same
  timezone. Chart endpoints backfill **every day in the range with zero**
  when there's no data, rather than omitting empty days — so a "Last 30
  days" chart should always show 30 data points, not a sparse list.
- **Endpoints:**
  - `GET /api/order-analytics/summary` — revenue (store currency +
    USD-converted) + quantity, for the Views/ATC/Conversion/Orders/
    Revenue numbers.
  - `GET /api/order-analytics/graph` — time-series for the line charts;
    `summary` param picks `quantity`/`revenue`/`revenue_usd`; `type`
    param picks bucket granularity (`hourly`/`daily`/`weekly`/`monthly`/
    `yearly`, default `daily`).
  - `GET /api/top-discount-tiers` — powers "Most Popular Discount Tiers";
    sortable by `quantity` or `total_items`, default limit 10 (though the
    Portal UI only shows top 3 per QVD-81).
  - `GET /api/top-selling-variants` — powers "Best-Selling Discounted
    Items", keyed by `shopify_product_id`/`shopify_variant_id`.
  - `GET /api/events` — aggregate counts by `type` (`view` |
    `add_to_cart`).
  - `POST /api/events` — the SDK's own event-logging call — **confirmed
    live 2026-09-14** with real captured request bodies (form-encoded,
    not JSON) on product "Copper Light" (`test-vl-1`, both Quantity
    Break offer 815 and Multibuy offer 816 active on the same page —
    see [feature-map.md](../product/feature-map.md)'s overlap finding):
    ```
    shop=test-vl-1.myshopify.com&offer_id=815&type=view&shopify_discount_id=1917720723777
    shop=test-vl-1.myshopify.com&offer_id=816&type=view&shopify_discount_id=1917722329409&variants[0][shopify_product_id]=9966979350849&variants[0][shopify_variant_id]=50590217994561&variants[0][quantity]=1
    shop=test-vl-1.myshopify.com&offer_id=815&type=add_to_cart&shopify_discount_id=1917720723777
    shop=test-vl-1.myshopify.com&offer_id=816&type=add_to_cart&shopify_discount_id=1917722329409&variants[0][shopify_product_id]=9966979350849&variants[0][shopify_variant_id]=50590217994561&variants[0][quantity]=1
    ```
    All 4 calls returned **200**. This **resolves** the "does Views count
    once per page load or per widget" edge case: **1 page load with 2
    active offer widgets fired 2 separate `view` events** (one per
    `offer_id`), not 1. See Edge Cases for a second, more surprising
    finding from this same trace: **a single Add to cart click also
    fired 2 separate `add_to_cart` events**, one for each offer on the
    page — including the one (Multibuy) the click never actually
    interacted with.
- All endpoints accept a `filter` param (`test`, `currency`, …,
  underspecified in the ticket) — likely for excluding test-mode orders
  from real merchant totals; `Need confirm` exact semantics.

## Edge Cases

_Some resolved via this session's live Portal check (2026-09-14); others
still drafted from ticket specs only._

- **Resolved (live 2026-09-14):** the Time range dropdown supports a full
  custom range (Starting/Ending date fields + calendar), capped and
  labeled "Maximum range is 90 days" — not just the 5 fixed presets.
  See Business Rules above.
- **New, found live 2026-09-14, not in ticket:** "Order with offers"
  showed no visible number or chart while every other metric showed a
  value (including "0.00 USD" for Additional revenue, which was also
  zero) — `Need confirm` whether this is a real display bug (a 0-state
  that renders blank instead of "0") or specific to this session's data.
  Worth a retest once a real test order exists in range.
- **Resolved (live 2026-09-14, SDK network trace on "Copper Light"):**
  "Views" (and "Add to cart clicks") count **once per distinct offer
  widget on the page, not once per page load/click.** 1 page load with
  Quantity Break (815) + Multibuy (816) both active fired 2 `view`
  events. See Backend API above for the exact captured request bodies.
- **New, found live 2026-09-14, not in ticket — possible over-counting/
  attribution issue:** clicking the page's single "Add to cart" button
  (which is Quantity Break's own button, adding its currently-selected
  pack) fired **2** `add_to_cart` events — one correctly for Quantity
  Break (815), but **also one for Multibuy (816)**, the offer whose own
  "Pick" button was never clicked in this action. Either every
  add-to-cart action on a page with multiple active offer widgets gets
  attributed to *all* of them (which would inflate ATC/conversion
  numbers for offers the customer never actually engaged with), or
  there's a more specific reason tied to this particular overlap
  scenario not yet understood. `Need confirm` with Dev — retest on a
  page with only 1 active offer widget to see if this is overlap-
  specific or happens unconditionally.
- **New, found live 2026-09-14, unresolved:** despite all 4 events above
  returning **200 OK**, the Analytics dashboard's View/ATC counts (8/2
  respectively, "Last 7 days") **did not change** after the real
  view + add-to-cart above — checked immediately, then again after an
  explicit in-app "Refresh" click, then again after a further ~20s wait
  and a full page reload. Could be: (a) processing/aggregation lag longer
  than what this session waited out, (b) the backend deliberately
  excludes traffic from the "Preview on Online Store" bypass route (used
  throughout this session to view the storefront without the store's
  password — plausible and arguably correct behavior, to avoid polluting
  a merchant's real metrics with staff/QA preview views), or (c) a
  genuine dashboard bug. **Not enough evidence to conclude which** — this
  is reported as an open, unresolved finding, not a confirmed bug.
  `Need confirm`: retest on the real (non-preview) storefront domain, or
  wait significantly longer before rechecking.
- Conversion rate's formula (`ATC / Views × 100`) uses Add to cart, not
  actual completed orders — so "conversion" here really means
  "add-to-cart rate," not checkout conversion. Does the Portal UI label
  it clearly enough to avoid confusion with "Orders with offers"
  (the actual completed-purchase metric)?
- "Additional revenue" — the ticket doesn't define the calculation
  precisely ("additional revenue generated from app"). Does this mean
  total revenue from orders containing a discounted item, or specifically
  the discount-attributable delta (e.g. upsell from a higher tier)? Needs
  live confirmation of the actual number shown against a known test order.
- What happens to historical analytics data when an offer is deleted —
  do past Views/ATC/Orders for that offer remain queryable, or vanish
  with it?
- Interaction with the Onboarding Dashboard's own ATC count (QVD-115) —
  same underlying event data, or computed differently? Not yet compared
  side-by-side live.
- The `filter` param's exact semantics (excluding test orders?) — not
  tested; could matter for QA test orders skewing a merchant's real
  numbers if filtering doesn't work as expected.

## Known Issues

See [known-issues.md](../known-issues/known-issues.md) for entries tagged `analytics`.

## Related Jira Tickets

- [QVD-73](https://ownego.atlassian.net/browse/QVD-73) — Analytics, Epic — status **Done**
- [QVD-81](https://ownego.atlassian.net/browse/QVD-81) — [Portal] Analytics dashboard, Story — full metrics/date-range spec
- [QVD-69](https://ownego.atlassian.net/browse/QVD-69) — [BE] Analytic, Task — backend API endpoints, timezone handling, event logging
- [QVD-119](https://ownego.atlassian.net/browse/QVD-119) — [BE] Update multibuy tier analytic, Task — no description found; likely a data-model tweak for Multibuy's tier-level analytics specifically
- [QVD-115](https://ownego.atlassian.net/browse/QVD-115) — ATC count on Onboarding Dashboard — cross-reference, see above
- [QVD-173](https://ownego.atlassian.net/browse/QVD-173) — AI-review funnel analytics — cross-reference, confirmed separate from this feature
- [QVD-176](https://ownego.atlassian.net/browse/QVD-176), [QVD-177](https://ownego.atlassian.net/browse/QVD-177) — Sidekick data extensions — cross-reference, likely reuse this feature's API once shipped

## Source

- Jira (QVD project), via Atlassian MCP, analyzed 2026-09-14.
- Live Playwright MCP test, `test-vl-1`, 2026-09-14 (Portal: metrics with
  real data, Time range dropdown including the custom date picker; SDK:
  network trace of `POST /api/events` from a real view + Add to cart on
  product "Copper Light", captured via Playwright's network inspector —
  request bodies and response codes documented above).
- QA, direct confirmation in conversation (2026-08-27).
