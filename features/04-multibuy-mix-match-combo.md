# 4. Multibuy – Mix & Match Combo

**Status:** in-review

**Freshly pulled from Jira, not yet QA-confirmed** — Business Rules below
are ticket specs (may not match shipped behavior 1:1), Edge Cases are open
questions, same as 01/02/03 were before QA review. Analyzed against Jira
2026-09-03 (QVD-28, QVD-99, QVD-100, QVD-102 through QVD-109, QVD-118,
QVD-121, QVD-129, QVD-132, QVD-151, QVD-155, QVD-162, QVD-166, QVD-178 —
see Related Jira Tickets below).

## Purpose

Lets merchants create promotions where customers **mix and match across
multiple different products** (not just quantity of the same product, see
[Quantity Break](03-quantity-break.md) for that) to unlock a discount —
e.g. "Any 3 products for $10," or "Buy any 2 items from this collection,
get 20% off." Customers pick which eligible products go into their bundle,
either via an in-page popup or a merchant-defined redirect page (QVD-106).

## Business Rules

- **Internal name & Offer title (QVD-103):** Internal name — text field,
  auto-defaulted to "Offer #1", "Offer #2", etc.; merchant-only reference,
  not shown to customers. Offer title — text field, default template
  `{{quantity_tier}} for {{total_price}}`, supports inserting dynamic
  variables (quantity of tier, total price) either by typing them manually
  or via a `{}` button that opens a variable picker.
- **Discount method (QVD-103):** automatic discount is the default. If a
  merchant hits Shopify's limit of **25 automatic discounts** or wants a
  discount **code** instead, they're directed to contact support — not
  self-serve in the UI.
- **Trigger settings — corrected 2026-09-03, supersedes QVD-104's written
  spec:** radio button, one selection only — **"All products"** (default),
  **"Collection"**, or **"Specific products"**. Runtime behavior: the
  discount is only **displayed** on the product/cart/checkout page when
  the trigger condition is met.
  - **"All products" — Resolved (QA confirmed 2026-09-03):** no exclude
    option. QVD-104's ticket described a "Select excluded products"
    option for this trigger; that is **not** the current actual behavior
    — treat the ticket's exclude-for-All-products spec as outdated.
  - **"Collection" (singular — QVD-104's ticket/UI mockup says
    "Collections," plural, but the label and behavior are single-select,
    so this KB uses the singular form to avoid implying multi-select) —
    Resolved (QA confirmed 2026-09-03), 2 corrections to QVD-104:** (1)
    merchant can select **only 1 collection** — not a multi-collection
    picker despite the ticket showing a generic search field; (2) **no
    exclude option** here either, same correction as "All products"
    above.
  - **"Specific products" — Resolved (QA confirmed 2026-09-03):** lets the
    merchant pick both **products and individual variants** (matches
    QVD-104's "Browse" flow — select an entire product or expand to
    choose specific variants). No exclude option, consistent with the
    original ticket (doesn't need one, since the merchant is already
    hand-picking the eligible set).
  - **Net effect: "Select excluded products" does not exist anywhere in
    current Multibuy Trigger settings**, despite QVD-104 documenting it
    for 2 of the 3 trigger types. Don't write test cases against the
    ticket's exclude flow for Multibuy.
  - **Cross-reference — Resolved (QA confirmed 2026-09-03): confirmed
    NOT identical to Quantity Break's implementation.** This trigger model
    (All products / Collection [1 only] / Specific products+variants) is
    a separate, simpler system from Quantity Break's 7-option Trigger
    Settings (see [03-quantity-break.md](03-quantity-break.md)) — Multibuy
    has no "Product tags" or "Custom trigger" condition builder, no
    exclude variants, and Collection is single-select here vs. Quantity
    Break's broader scope. Confirmed by QA as an obvious/expected
    difference, not something requiring further investigation. **Radio
    list itself re-confirmed live 2026-09-03** (offer created on store
    `test-vl-1`): exactly 3 radios, "All products" / "Collection" /
    "Specific products", matching this section.
  - **Trigger-overlap warning — discovered live 2026-09-03, shared
    mechanism with Quantity Break, not in any ticket found:** if this
    offer's trigger includes a product already covered by another offer's
    trigger (any offer type), Portal shows a **non-blocking** warning:
    *"This trigger overlaps with offer(s): &lt;name&gt;. Overlapping
    triggers may cause discount conflicts. Please consider carefully
    whether to adjust this trigger."* Confirmed live: creating a Multibuy
    offer on the same product as an existing Quantity Break offer
    triggered the warning, saved anyway, and both widgets rendered
    simultaneously on that product page. See
    [feature-map.md](../product/feature-map.md) Known interactions for
    the full writeup and the still-open checkout-math question.
- **Tier settings (QVD-105, extended by QVD-155) — variable syntax
  corrected 2026-09-03 via live Portal check (store test-vl-1, offer
  "Offer 13"):** merchant adds multiple tiers. Each tier has: a per-tier
  **Title** (live default `{quantity} for {tier_price}` — **single**
  curly braces, not the `{{double}}` notation QVD-103's ticket text used),
  minimum quantity ("Quantity of product required," numeric), a reward
  configured as either **Fixed total price** (QVD-105's original spec,
  e.g. "Buy 3 → $10 total") or **Percentage off** (added later by QVD-155,
  e.g. "Buy any 3 items → 30% off," applied across any combination of
  eligible products), plus a **Badge** (live default `Saved
  {saved_price}`, single braces). Live example seen: 3 tiers "2 for $10,"
  "3 for $20," "4 for $30" (Fixed total price mode).
  - **Default state on first load:** 1 pre-populated tier — qty 1 / $10
    total / badge "Saved {saved_price}".
  - **Guardrail:** at least one tier must always exist — cannot delete the
    last remaining tier. No tier configured → no discount applied.
  - **Expand/collapse & tier actions:** same pattern as Quantity Break
    (QVD-23/QVD-105 share near-identical wording) — Tier 1 expanded by
    default, only one tier expanded at a time, Collapse/Edit/Delete
    buttons mutually exclusive by expanded/collapsed state, "+ Add new
    tier" auto-expands the new tier and collapses whichever was open,
    auto-assigns next tier number/quantity.
  - **Real-time validation** on rule violations; success toast/inline
    message on Save.
- **"Maximum discount uses" section — discovered 2026-09-03 via live
  Portal check, description confirmed by QA same day, not in any ticket
  found so far:** a checkbox **"Allow to apply multiplier discounts per
  order"** (checked in the live example seen), helper text "Apply the
  discount for every pack that meets the required quantity." **Resolved
  (QA confirmed 2026-09-03):** when enabled, the discount applies to
  **each qualifying pack in a single order** — e.g. buying 6 items on a
  "3 for $20" tier applies the discount **twice** → $40 total (not $20 +
  full price for the other 3). This is the **Fixed-total-price
  equivalent** of the set-repeating behavior QVD-178 documents for
  Percentage mode's "Apply per combo product." **Not yet confirmed:**
  exact behavior when this checkbox is **unchecked** (does the tier then
  apply only once, with the remainder at full price, similar to
  Percentage mode's remainder handling?), or whether this setting is
  Fixed-price-only or also affects Percentage mode alongside QVD-178's
  separate toggle.
- **"Remaining quantity message" field — discovered 2026-09-03 via live
  Portal check, not in any ticket found so far:** a merchant-editable text
  field, live default `Add {remaining_quantity} more items to get this
  deal`, shown on the storefront widget for customers who haven't yet
  reached the **first** qualifying tier (progress-toward-a-deal
  messaging — confirmed live in the offer's Preview section, e.g. "Add 2
  more items to get this deal"). This is the **configurable version** of
  QVD-121's "Add X more items to get the deal at $Y" spec — it's a real
  Portal field, not hardcoded SDK copy. **Distinct from** the already
  QA-confirmed "no nudge message" edge case below, which is specifically
  about a *leftover* unit after a full set is already complete — this
  field only covers progress *before* reaching a tier at all.
- **Quantity pooling across different products — Resolved (QA confirmed
  2026-09-03), not documented in any ticket found so far, and the
  opposite of Quantity Break's behavior:** under an "All products"
  trigger (and, by the feature's own purpose, likely the other trigger
  types too — not yet separately confirmed), Multibuy **pools quantity
  across every eligible product/variant in the cart**, unlike Quantity
  Break which explicitly does **not** pool across different products (see
  [03-quantity-break.md](03-quantity-break.md)). This is the core
  difference in mechanic between the two features — Multibuy exists
  specifically to let customers mix products into one combo.
  - **Remainder/SKU-selection rule for "Fixed total price" tiers
    (Resolved, QA confirmed 2026-09-03):** when pooled quantity exceeds a
    tier's threshold with a remainder that doesn't reach the next tier,
    the discounted set is filled with the **highest-priced** eligible
    units first, leaving the **cheapest** unit(s) as the remainder at
    full price. Example: Tier "5 items → $25 total." Cart has A/Small
    ($10)×1, A/Medium ($15)×3, B ($10)×2 — 6 units total, pooled. The
    5-item set = the 3 A/Medium units ($15 each) + 2 of the $10 units;
    the 1 remaining $10 unit (A/Small or B) is charged at full price
    ($10). **Not yet confirmed:** tie-breaking when multiple units share
    the same "cheapest" price (here, both A/Small and B are $10) — which
    specific line gets left as the remainder.
  - **Exact-match case (no remainder) — verified live 2026-09-03, real
    storefront test (store `test-vl-1`):** when pooled quantity lands
    exactly on a tier's threshold (no leftover), **every** unit in the
    set gets discounted, and the tier's fixed total price is allocated
    **proportionally by each unit's original price** — not evenly split,
    not highest-price-first. Verified: tier "3 items → $20," cart had
    Copper Light ($60)×2 + Pink Armchair ($75)×1 (3 units, exact match).
    Result: Copper Light charged $6.16/unit ($12.31 total), Pink Armchair
    charged $7.69. Check: `120/(120+75) × 20 = 12.31` and
    `75/195 × 20 = 7.69` — confirms proportional allocation by original
    line value across the whole set, consistent with the "highest-priced
    first" remainder rule above (same underlying proportional-allocation
    engine, just no remainder to carve off when the match is exact).
- **"How discount is applied" setting (QVD-178) — only shown when
  Percentage is selected as the discount type**, positioned directly below
  Discount type, above the tier list:
  - **"Apply per combo product" (default, preserves original behavior):**
    quantity is grouped into fixed **sets** matching each tier's quantity;
    sets repeat if total quantity is a multiple of the tier quantity; any
    **remainder** that doesn't complete a full set is charged at **full
    price**. Example (tiers 2=10%, 3=20%, 4=30%; cart has 4 eligible
    items): 3 items get 20% off (the 3-tier), the remaining 1 item stays
    full price.
  - **"Apply highest tier to entire quantity" (new):** finds the
    **highest tier** whose threshold the total eligible quantity still
    meets, and applies that tier's rate to the **entire** quantity — no
    set grouping, no remainder. Same example → all 4 items get 30% off
    (the 4-tier, since 4 qualifies for it).
  - **Quantity below the lowest tier threshold → no discount, in both
    modes.**
  - Setting is **fully hidden** when discount type = Fixed total price (no
    clean way to scale a fixed $ amount across a partial/uneven quantity).
  - **Switching Percentage → Fixed total price** after "Apply highest
    tier to entire quantity" was selected: the setting hides and its
    value **resets to default** ("Apply per combo product") on Save.
  - Backend: `tier_application_method` field (`per_group` |
    `highest_tier_all`) on the discount config, handled in the
    `multibuy-discount` Function at the tier-matching step.
  - **Note:** this is the same underlying concept QA independently
    confirmed for Quantity Break (tier-vs-entire-quantity calculation, see
    03-quantity-break.md) — except for Multibuy it's an explicit
    **merchant-facing toggle** with a documented default, not a fixed,
    undocumented behavior.
- **Customer product selection behavior (QVD-106):** radio choice —
  **"Open selection popup"** (default) or **"Redirect to a specific
  page/URL"** (reveals a "Destination URL" input field when chosen).
  - **Popup mode detail (QVD-121):** triggered by the "+" button or "View
    more products." For products **with variants**: "+" expands a
    **variant picker** sliding up from the bottom of the product card
    (select variant options, adjust quantity, add-to-cart button); a
    semi-transparent overlay appears, tapping outside the picker collapses
    it back to default. For products **without variants**: "+" instantly
    adds to cart and the "+" icon changes to a cart/added state. A
    persistent **cart summary** within the popup shows thumbnails of added
    products, empty slots for remaining suggested quantity, current total
    price, and a "Buy now" CTA. A dynamic title + remaining-quantity
    message updates live, e.g. "Add X more items to get the deal at $Y"
    (configurable — see "Remaining quantity message" field above).
  - **Confirmed live 2026-09-03 (real storefront, not just the ticket
    spec):** clicking **"Pick"** on a no-variant recommended product adds
    it to the **real cart immediately** (not a staging/preview area) —
    confirmed by checking the actual `/cart` page right after. The
    progress bar/count ("Add N more items...") reflects **actual live
    cart contents**, not a separate in-widget counter: reloading the
    product page after a Pick showed the updated progress correctly. Also
    confirmed the widget's empty "+" combo slots exist in the DOM
    (`qvl-multibuy-tier_add-more`) but did not respond to a scripted
    (non-trusted) click event in this session — if automating this in
    Playwright/browser tests, use a real `click()` action, not
    `element.click()` via page evaluate, or it may silently no-op.
  - **New fields, discovered 2026-09-03 via live Portal check (not in
    QVD-106's ticket):** below the redirect-URL input, a checkbox **"Show
    product recommendations"** (checked in the live example) and, when
    checked, a **"Max number of recommended products"** number field
    (max 10, live example set to 5). Live example also showed the
    redirect radio selected with URL
    `https://test-vl-1.myshopify.com/products/wooden-outdoor-slats` —
    confirms "recommendations" apply to **both** selection modes (popup
    and redirect), not just the popup.
- **Appearance & style (QVD-107) — expanded 2026-09-03 via live Portal
  check, far more granular than the ticket's high-level Figma spec:** an
  accordion with 6 sections — **Layout** (card border radius, live
  default 12px), **Color** (Primary/Secondary/Accent color pickers, live
  hex values shown), **Typography** (Heading/Body/Badge — each with font
  weight dropdown + font size number field, live defaults: Heading
  Bold/20px, Body Regular/14px, Badge Regular/12px), **Image** (image
  size slider, live default 60px; image border radius slider, live
  default 12px), **Badge** (Badge text — live example "MULTIBUY" in caps,
  vs. QVD-132's documented default "Multi buy"; not yet confirmed which
  is the true out-of-the-box default vs. this being a merchant-edited
  value; Badge position dropdown, live default "Top left"; Badge border
  radius slider, live default 6px — QVD-132 said default 10px, another
  discrepancy not yet resolved), and **Custom CSS** (textarea gated
  behind "contact our support team to enable" — confirms Custom CSS is
  **not self-serve**, merchants must ask support to turn it on before the
  field becomes usable; see [Custom CSS](../product/custom-css.md)).
- **Discount combination (QVD-108):** checkboxes to combine with Order /
  Product / Shipping discount, all default **off** — identical structure
  to Quantity Break's QVD-25. Same shared-vs-exclusive caveat applies (see
  Quantity Break's file) — not confirmed whether this is one shared
  component across offer types or a per-offer-type copy.
- **Offer scheduling (QVD-109):** start date/time (required, defaults to
  creation time, can't be in the past, store timezone) + optional end
  date/time (behind "Enable end date," default off = runs indefinitely;
  end ≥ start; same-day start/end defaults end time to 11:59 PM) —
  identical structure to Quantity Break's QVD-26.
  - **"Enable countdown timer" checkbox — discovered 2026-09-03 via live
    Portal check, not in QVD-109's ticket:** sits at the bottom of the
    Campaign scheduling section (unchecked in the live example). Likely
    ties to [QVD-147](https://ownego.atlassian.net/browse/QVD-147)
    "Countdown timer to offer" (New Feature, found during ticket search
    but not yet analyzed in depth for this file) — showing a countdown to
    the offer's end date on the storefront widget. `Need confirm` exact
    behavior/placement; worth a follow-up analysis pass on QVD-147.
- **Multibuy discount badge (QVD-132):** optional badge shown on product
  images on **collection pages, recommendation sections**, and optionally
  the **cart page**, to flag products that have an active multibuy offer.
  Settings: Badge text (default "Multi buy", **blank hides the badge**,
  ~20–25 char recommended limit), Position (dropdown: Top-left [default],
  Top-right, Bottom-left, Bottom-right), Badge border radius (slider,
  default 10px).
- **Compare-at-Price discount logic (QVD-166) — corrected 2026-09-03,
  supersedes the ticket's framing:** QVD-166's ticket text describes this
  as a one-off custom build for a single merchant ("Supersport," Buy 4 Get
  50% Off campaign) — **QA confirmed this is actually a global feature**,
  not merchant-specific, so it's in scope for standard Multibuy
  regression, not carved out. Logic: if `compare_at_price > 0`, the
  discount % is calculated off `compare_at_price` (original price), not
  `price` — e.g. compare_at_price 1,000,000₫, price 800,000₫, 50% off →
  final 500,000₫ (half of *original*, not half of current selling price).
  Falls back to standard `price`-based calculation if no compare_at_price
  is set. **Known, accepted limitation:** if a customer changes quantity
  directly in the cart (rather than re-adding from the product page), the
  displayed discount/compare price can look inconsistent — this is a
  Shopify cart-display constraint, not an app bug, and was explicitly
  accepted rather than fixed. Also supports an optional "silent" mode
  (discount applies automatically, offer widget UI hidden via custom CSS)
  — feasible, but merchants using it should be told about Shopify's
  native 25-automatic-discount limit.
- **Store-specific SDK config, not general/default behavior:**
  `showTriggerProductInRecommendation` (QVD-129) and
  `numberOfRecommendationProductPerLine` (QVD-162) — SDK-level custom
  config flags (`QikifyVolume = {...}`), not merchant-facing portal
  settings — set per-store by Dev, not something QA should expect to see
  in the Portal UI by default.

## Edge Cases

_Drafted from ticket specs — not yet QA-confirmed, treat as open questions
the same way 01/02/03's edge cases started._

- Fixed total price tier, remainder tie-breaking: when 2+ eligible units
  share the same lowest price (e.g. two different $10 line items), which
  specific one gets left as the full-price remainder — cart order, product
  ID, or something else? `Need confirm`.
- Redirect-to-URL mode (QVD-106) — **Resolved (QA confirmed 2026-09-03):**
  not a concern — the redirect **opens in a new tab**, so the original
  page (and any in-progress selection) is untouched; there's no
  navigate-away to reset anything.
- Percentage mode + "Apply per combo product," remainder handling —
  **Resolved (QA confirmed 2026-09-03):** example confirmed — tiers "2
  items→10%," "3 items→20%," customer has 4 eligible items → the 3-item
  set is charged at 20% off, the 1 remaining item at full price. **No
  nudge/prompt message** for the leftover item — just the itemized price
  breakdown (discounted items priced differently from the full-price one),
  with nothing extra like "buy 1 more to save."
- ~~"Apply highest tier to entire quantity" + Discount combination
  interaction~~ — **corrected framing (QA, 2026-09-03):** these operate on
  different layers, not really an "interaction" to test as originally
  framed. "Apply highest tier to entire quantity" only controls **how this
  offer's own Percentage discount amount is calculated** (per-group vs.
  whole-quantity). "Discount combination" only controls whether **this
  offer's resulting discount** is allowed to stack with unrelated
  Order/Product/Shipping discounts. They don't conflict by construction —
  combination just takes whatever discount amount the tier-application
  method already computed. No longer treating this as an open edge case.
- Switching discount type Fixed ↔ Percentage repeatedly — **Resolved (QA
  confirmed 2026-09-03):** tier values are **preserved**, not lost or
  corrupted, when switching back and forth (only the documented "Apply
  highest tier" setting itself resets, per QVD-178).
- Multibuy discount badge (QVD-132) + Offer scheduling (QVD-109) —
  **Resolved (QA confirmed 2026-09-03):** badge is gated by scheduling —
  it **hides** automatically when the offer is outside its scheduled
  window, and only shows while the offer is active.
- Compare-at-Price discount logic (QVD-166) — **Corrected (QA, 2026-09-03):
  this is actually a global feature**, not scoped to one merchant as
  QVD-166's ticket text implied ("custom, single-merchant build" for
  Supersport). Business Rules above have been corrected accordingly — see
  that section. In scope for standard Multibuy QA coverage.

## Advanced Settings

Shares the [Custom CSS](../product/custom-css.md) input field with other
offer types — **confirmed live 2026-09-03:** the Custom CSS field in
Multibuy's Appearance & style section is gated behind "contact our
support team to enable," i.e. **not self-serve by default**; a merchant
has to ask support to turn it on before the field is usable.

## Known Issues

See [known-issues.md](../known-issues/known-issues.md) for entries tagged `multibuy-mix-match-combo`.

## Related Jira Tickets

- [QVD-99](https://ownego.atlassian.net/browse/QVD-99) — [SDK] Multibuy discount UI, Story — no description found
- [QVD-100](https://ownego.atlassian.net/browse/QVD-100) — Multibuy Offer, Epic — no description found, parent epic
- [QVD-102](https://ownego.atlassian.net/browse/QVD-102) — [Portal] Multibuy offers, Story — settings interface overview, Figma-referenced
- [QVD-103](https://ownego.atlassian.net/browse/QVD-103) — [Portal-Multibuy] Internal name & Discount type, Sub-task — naming fields, automatic discount default, 25-limit note
- [QVD-104](https://ownego.atlassian.net/browse/QVD-104) — [Portal-Multibuy] Trigger settings, Sub-task — All products/Collection (single-select)/Specific products
- [QVD-105](https://ownego.atlassian.net/browse/QVD-105) — [Portal-Multibuy] Tier settings, Sub-task — core tier engine, fixed-price reward
- [QVD-106](https://ownego.atlassian.net/browse/QVD-106) — [Portal-Multibuy] Customer product selection behavior, Sub-task — popup vs redirect
- [QVD-107](https://ownego.atlassian.net/browse/QVD-107) — [Portal-Multibuy] Appearance & style, Sub-task — widget visual customization
- [QVD-108](https://ownego.atlassian.net/browse/QVD-108) — [Portal-Multibuy] Discount combination, Sub-task — Order/Product/Shipping toggles
- [QVD-109](https://ownego.atlassian.net/browse/QVD-109) — [Portal-Multibuy] Scheduling, Sub-task — start/end date+time
- [QVD-118](https://ownego.atlassian.net/browse/QVD-118) — [SDK] Multibuy function logic, Customize — no description found
- [QVD-121](https://ownego.atlassian.net/browse/QVD-121) — [SDK] Product selection popup, Story — popup UX detail
- [QVD-129](https://ownego.atlassian.net/browse/QVD-129) — [SDK] Improvement multibuy, Customize — custom config flag, store-specific
- [QVD-132](https://ownego.atlassian.net/browse/QVD-132) — Multibuy discount badge, Story — collection/recommendation badge spec
- [QVD-151](https://ownego.atlassian.net/browse/QVD-151) — Multibuy with percentage off, Epic — no description found, parent of QVD-155
- [QVD-155](https://ownego.atlassian.net/browse/QVD-155) — Multibuy - Tiered Percentage Discount, New Feature — adds Percentage discount type
- [QVD-162](https://ownego.atlassian.net/browse/QVD-162) — [SDK] Multibuy - expand image on recommendation product, Customize — custom config flag, store-specific
- [QVD-166](https://ownego.atlassian.net/browse/QVD-166) — [Custom-SSP] Multibuy discount logic based on Compare-at-Price, Story — ticket text frames it as a single-merchant build (Supersport), but **QA confirmed 2026-09-03 this is actually a global feature**
- [QVD-178](https://ownego.atlassian.net/browse/QVD-178) — Multibuy: "Apply highest tier to entire quantity" setting, Story — per-group vs highest-tier-all discount application

## Source

- Jira (QVD project), via Atlassian MCP, analyzed 2026-09-03.
- Help center (may be outdated): https://help.qikify.com/en/article/multibuy-mix-match-combo-1iht99r/
