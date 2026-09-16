# 5. Wholesale Pricing Offer

**Status:** in-review

Analyzed against Jira 2026-09-07 (QVD-142, QVD-143, QVD-150, QVD-156,
QVD-180 — see Related Jira Tickets below), then **live-verified 2026-09-07
through 2026-09-09 via Playwright MCP** against store `test-vl-1` (offer
"Offer 17"/ID 818, from the "Wholesale pricing (B2B)" template) — both
Portal settings and storefront/SDK behavior (PDP inline rendering, tier
math edge cases, table layout, cross-feature overlap) have now been
live-tested. Several ticket specs were found to not match shipped
behavior 1:1 and are called out inline as "corrected"/"confirmed live".

## Purpose

B2B/wholesale volume pricing: buyers get a **percentage discount that
scales with total quantity ordered**, configured as **min–max quantity
ranges** (not individual thresholds like [Quantity Break](03-quantity-break.md)).
The storefront centerpiece is a **Variant Selection Drawer** that lets a
buyer compose an order across multiple variant combinations (e.g. Color ×
Size × Material) in one session, see a running per-variant quantity
count, and watch the applicable pricing tier update live as they add
quantities — aimed at B2B buyers who order in bulk across many
variants at once, not a single-variant repeat purchase.

## Business Rules

### General information & discount setup (QVD-150)

- **Internal name** — merchant-only reference, not shown to customers.
- **Tier pricing title** — heading shown on the storefront tier pricing
  card (e.g. "TIER PRICING").
- **Variant title** — heading shown on the storefront variant preview
  card (e.g. "VARIANTS").
- **Discount method:** automatic discount is the default. Ticket note:
  "If users want to use discount code, contact support to add the config
  to show this setting" — same not-self-serve pattern documented for
  Quantity Break/Multibuy's 25-automatic-discount-limit workaround.
- **Discount name** — shown on cart/checkout; has a "Generate" button.

### Product targeting — live-verified 2026-09-07 (store `test-vl-1`, offer "Offer 17")

- **"Offer is triggered for"** radio group — **confirmed exactly 5
  radios, same as Quantity Break, not the odd combined wording in the
  ticket table:** **All products**, **Collections**, **Product tags**,
  **Specific products**, **Custom trigger**.
- **"Exclude products"** — confirmed live as a separate link/button below
  the radios (shown when "All products" is selected), same pattern as
  Quantity Break, not a checkbox. Same warning banner as Quantity Break
  also appeared here: *"Automatic discounts work even if the app block is
  disabled..."* — confirms that banner is **shared across all offer
  types**, not Quantity-Break-specific.
- Also live-confirmed: an automatic **trigger-overlap warning**
  ("Overlapping triggers may cause discount conflicts...") appeared
  immediately on this new offer, same non-blocking mechanism documented
  for Quantity Break/Multibuy in
  [feature-map.md](../product/feature-map.md).

### Tier settings — table, not threshold cards — live-verified 2026-09-07

Structurally different from Quantity Break/Multibuy's tier UI: a
**table** where each row is a **quantity range**, not a single threshold.

- **New fields, not in the ticket:** a **"Discount type"** radio —
  **Percentage** (default) / **Fixed Amount** — and a **"Show discount
  badge"** checkbox (checked by default). QVD-150's ticket only described
  a plain percentage column; the Percentage/Fixed Amount choice mirrors
  Quantity Break's tier discount type and wasn't documented in Wholesale's
  own ticket.
- Table columns — **live label is "Discount per item", not "Discount
  value" as the ticket said**: **Tier**, **Min quantity**, **Max
  quantity**, **Discount per item**, delete icon (per row).
- **"Add new tier"** appends a row.
- **Default state on first load — 3 pre-populated, continuous, no gaps:**
  Tier #1 = qty 1–10 / 0%; Tier #2 = qty 11–20 / 10%; Tier #3 = qty 21–30
  / 20%.
- **Resolved — top-of-range behavior, live cart-tested 2026-09-09 (store
  `test-vl-1`, product "Clay Plant Pot"):** the "leave Max empty for
  unlimited" hint turned out to be **misleading** — it implies quantity
  beyond an explicitly-capped last tier gets no coverage, but that's
  **not what happens**. With the default tiers unchanged (Tier 3 = 21–30
  / 20% off, Max = 30 *not* emptied), setting quantity to **35** (beyond
  Max) still applied Tier 3's rate to the **entire** quantity: unit price
  showed $16.00/item (Tier 3's discounted rate) and Subtotal was
  ~~$700.00~~ **$560.00** = `35 × $16.00`. So an explicit Max on the last
  tier does **not** actually cap or block the discount for quantity
  beyond it — the highest defined tier's rate keeps applying regardless.
  Leaving Max blank may just avoid a display quirk (e.g. showing "21-30"
  instead of "21+") rather than changing the actual discount math — not
  independently tested since this store's last tier already behaves as
  if unlimited.
- **Resolved — tier continuity is NOT enforced, live-tested 2026-09-09:**
  changed Tier #1's Max quantity from 10 to **8** (creating a gap: 9-10
  falls in neither Tier 1 [1-8] nor Tier 2 [11-20]) and saved — **no
  validation error, save succeeded**, and the gap persisted after a page
  reload. The ticket's "must be validated for continuity (no gaps or
  overlaps)" AC is **not actually implemented** as a blocking rule.
  - **Gap behavior, live cart-tested:** with quantity set to **9** (inside
    the gap), the storefront showed **no discount at all** — unit price
    stayed at the full $20.00/item, Subtotal = $180.00 = `9 × $20.00`, no
    tier card highlighted. A quantity that matches no tier's range simply
    gets full price, silently — no error shown to the shopper either.
  - Test tier was restored to Max=10 afterward to keep the offer's
    default 3-tier example accurate for future reference/testing.
- This tier table is what drives both the PDP tier cards and the
  in-drawer tier display (QVD-143).

### Appearance & Style (QVD-150, extended by QVD-156) — colors/CSS gate live-verified 2026-09-07

- **Layout:** carousel of card-layout presets, **plus a table layout**
  added later by QVD-156 (see Storefront section below) — selectable in
  the same Layout carousel. **Card border radius** slider.
- **Color — corrected 2026-09-09: actually 4 pickers, not 3.** An earlier
  pass of this file said "exactly 3 pickers, matching the ticket" after
  only searching for "Accent color" text; a fuller live snapshot
  (2026-09-09) shows a 4th picker: **Primary color**, **Secondary color**,
  **Accent color**, and **Badge background color** (live defaults seen:
  Primary `#000000`, Secondary `#000000`, Accent `#FFFFFF`, Badge
  background `#D75E60`). Quantity Break's equivalent is a different
  4-color set (Primary/Secondary/Label background/Label text) — still not
  a shared color-field component, but both offer types happen to have 4
  pickers, not 3-vs-4 as previously stated.
- **Typography:** Heading / Body / Badge / **Normal text** — 4 groups,
  each with font weight + font size. "Normal text" here vs. "Label" in
  Quantity Break's equivalent — still not individually re-verified live,
  but consistent with the Color field set also being offer-type-specific
  rather than shared.
- **Badge — confirmed live 2026-09-09, not previously documented:** a
  dedicated settings group ("Badge will be shown on product image (on
  collection page, recommendation) to highlight the deal") with **Badge
  text** (live default `"BULK SAVING"`), **Badge position** (default
  "Top left"), and **Badge border radius** slider (default `6px`).
  Distinct from the "Show discount badge" checkbox in Tier settings above.
- **Custom CSS — Resolved (QA confirmed live 2026-09-07): gated behind
  "contact our support team to enable," same as Quantity Break and
  Multibuy.** The ticket's silence on this wasn't evidence of being
  self-serve — all 3 offer types share the same support-gated Custom CSS
  pattern.

### Combination (QVD-150) — confirmed live 2026-09-07

- Order discount / Product discount / Shipping discount checkboxes —
  **confirmed live: all 3 checked by default** on a freshly-created
  offer, matching the ticket. Combined with the same live-checked default
  seen on Quantity Break's and Multibuy's example offers, the true
  shipped default across all 3 offer types is **on** — the "unchecked by
  default" wording in Quantity Break's/Multibuy's own older tickets was
  outdated, not Wholesale being the odd one out.

### Campaign scheduling (QVD-150)

Start date/time, optional end date/time behind "Enable end date" — same
shape as Quantity Break's QVD-26 and Multibuy's QVD-109. Not yet
individually re-verified live for Wholesale specifically, but no reason
expected to differ.

## Storefront (QVD-143, extended by QVD-156) — live-verified 2026-09-07

**Major structural correction — the "drawer" doesn't exist as a separate
step, at least not on this theme/template:** QVD-143 specifies a 2-stage
design (PDP shows a collapsed VARIANTS *preview* card → clicking it opens
a slide-in **Variant Selection Drawer** on the right edge). **Live
behavior on store `test-vl-1` (product "Clay Plant Pot," offer "Offer
17") shows the full variant-composition UI rendered directly inline on
the PDP** — no preview card, no "Select now" link, no slide-in
panel, no backdrop. Everything the ticket describes as living *inside the
drawer* (variant rows, badges, quantity steppers, subtotal) is just...
on the page. **Every other mechanic in the ticket checks out exactly**,
just without the drawer wrapper — see below. Not yet confirmed whether
this is a theme-specific difference (inline vs. drawer could be a
configurable/theme-dependent rendering mode) or the drawer genuinely
isn't shipped.
- **Checked 2026-09-09 for a second theme to test against — none
  available on `test-vl-1`:** the store's only *published* theme is Dawn
  (where the inline rendering above was observed); the 4 other themes
  installed (Horizon, Savor, Crave, Vessel) are unmodified Shopify
  starter themes added from the Theme Store, not duplicates of this
  store's live Dawn setup, so they almost certainly don't have the
  Qikify app embed block placed/configured on them at all — testing on
  one wouldn't isolate a theme-rendering difference, it'd just show
  "block missing because never added." A real test would need either a
  duplicate of the live Dawn theme (to get a true second same-app-config
  instance) or manually adding+configuring the app embed on a starter
  theme first — out of scope for a quick check. `Need confirm`: still
  open, same as before.

- **PDP tier cards are always visible** and purely informational — they
  highlight whichever tier matches the quantity the buyer has composed via
  the variant rows, and the cards themselves are **never clickable**
  (display-only, unlike Quantity Break's tier cards which are themselves
  the selection mechanism). **Confirmed live:** the active tier card gets
  a highlighted background and bold price — verified going from 1 unit
  (Tier "1-10 Items" highlighted, $20.00) to 12 units (Tier "11-20 Items"
  highlighted instead, $18.00).
- **Variant ordering is merchant-defined — confirmed live:** this
  product's 3 variant types rendered in the order **Size → Material →
  Color** (not a fixed Color→Size→Material sequence), matching whatever
  order the merchant set for the product's option types.
- **The LAST variant type is special-cased — confirmed live:** Size and
  Material rendered as ordinary single-select rows ("Size: Regular" with
  swatch images to switch; "Material: Clay" with text pills), while
  **Color (the last type) rendered as a quantity row per value** — Brown
  and Black each got their own independent +/− stepper, letting the buyer
  order both simultaneously.
- **Per-first-variant badge counters — confirmed live, exactly as
  specified:** the first variant type's currently-selected value (here,
  "Size: Regular") displayed a live count badge. Setting Brown's quantity
  to 12 made the Regular swatch show badge **"12"** — confirms the badge
  sums quantity across the last-variant's values, keyed to the first
  variant's current value, live-updating on every stepper change.
- **Per-unit price on each last-variant value updates live with the
  active tier — not explicitly called out in the ticket, but confirmed
  live:** Brown's and Black's displayed price (e.g. "$20.00/Item") changed
  to "$18.00/Item" the moment total quantity crossed into the 11-20 tier
  — the *rate shown next to each option*, not just the subtotal, tracks
  the active tier in real time.
- **Subtotal — confirmed live, with one addition:** shows current total
  quantity and a subtotal that recalculates live; when a discount is
  active, the **original (undiscounted) subtotal shows struck through**
  next to the discounted figure (e.g. ~~$240.00~~ **$216.00** for 12 units
  at the 11-20 tier's $18/unit rate — matches `12 × $18 = $216`,
  `12 × $20 = $240`).
- **"Reset selection" link** — appears once any quantity is set; not
  documented in the ticket.
- **Recurring debug artifact, same bug family as Quantity Break's
  `{ [Default Title]: "", }` (see that file's Business Rules):** a raw
  text string appeared on this PDP too — `{ [Regular]: "", }, {
  [Large]: "", }` — literally listing this product's Size variant values.
  Seeing the same pattern on 2 different offer types (Quantity Break and
  Wholesale) on 2 different products suggests this is a **shared/systemic
  bug in the SDK layer**, not an isolated one-offer-type issue — worth
  logging centrally rather than re-discovering per offer type. See
  [known-issues.md](../known-issues/known-issues.md).

### Table layout (QVD-156)

Alternative to the default card layout, selected from the same Appearance
& Style → Layout carousel (QVD-150).

- **PDP table:** bordered table, header row **Quantity | Discount | Unit
  price**; each tier row shows its quantity range (e.g. "1 – 20"),
  discount %, and calculated unit price.
- **Active tier row highlight:** info-colored background, info-colored
  medium-weight text, and a check icon before the quantity range on the
  active row; inactive rows are plain text, no icon, no highlight.
- **Same table renders inside the Variant Selection Drawer**, more
  compact (reduced font size/padding) to fit the drawer width; active-row
  highlight behavior identical and still updates live off the stepper
  controls.
- **Switching between card layout and table layout in the portal is
  claimed to apply live on the storefront with no page reload required**
  (ticket AC) — not yet independently verified.

## Custom config, piloting with select merchants (not a general Portal feature)

- **[QVD-180](https://ownego.atlassian.net/browse/QVD-180) "Minimum
  quantity enforcement" — corrected 2026-09-07, supersedes the "To Do, not
  started" status below: ticket has since moved to status "Passed" and
  shipped as a per-store custom config, currently being piloted with a
  handful of merchants — not yet a general, merchant-facing Portal
  setting.** Config flag (per QVD-180's latest comment, 2026-09-07):
  ```
  customConfig = {
      ENABLE_WHOLESALE_QTY_ENFORCE_OPTION: true,
  }
  ```
  Only stores Dev has explicitly turned this flag on for get the
  behavior — for every other store, minimum-quantity enforcement does
  **not** exist, same as before. **If the pilot goes well, this is
  expected to graduate into a real, Portal-configurable feature** — don't
  assume the flag name/shape above is final.
  - **Confirmed live 2026-09-08/09 (store `test-vl-1`):** flag is **off**
    for this store, as expected — tested quantity = 1 (well below any
    plausible minimum) via the storefront Add to cart, and it succeeded
    with no blocking, no error. `test-vl-1` is not one of the pilot
    stores, so this only confirms the flag is off here, not the
    enforcement behavior itself when on.
  - **Original problem + proposed mechanics (still the best description
    of what the flag actually does, per the ticket's design discussion
    before implementation):** 2 real merchant requests to block
    add-to-cart below a configured minimum quantity (e.g. 25 units, or 6
    units) when using Wholesale/volume pricing. Mechanism: adds a setting
    "Restrict ATC when minimum quantity is not met"; listens for cart
    changes and auto-removes a line item if its quantity drops below the
    configured minimum (reuses the existing gift-with-purchase removal
    logic pattern).
  - **Documented limitations from the design discussion — not yet
    re-confirmed against the shipped pilot, but likely still apply:**
    only catches adds through the product page's own ATC button (relies
    on the app's own line-item properties); **cannot** prevent adds from
    collection/home page quick-add, and **cannot** prevent a buyer from
    directly increasing quantity in the cart past the minimum then back
    down. `Need confirm` whether the shipped pilot behaves exactly per
    this original proposal or diverged during implementation — worth
    checking directly on one of the piloting stores if reachable, rather
    than assuming the pre-implementation design doc is still 1:1 accurate.

## Edge Cases

_Drafted from ticket specs — not yet QA-confirmed, treat as open questions
the same way 01/02/03/04's edge cases started._

- Tier range continuity validation — **Resolved (QA-tested live
  2026-09-09):** NOT enforced. A gap can be saved with no error; a
  quantity landing in the gap silently gets full price (no discount, no
  error shown). See Business Rules for the exact test.
- Total quantity **below** the lowest tier's min — **Resolved (QA-tested
  live 2026-09-09):** confirmed "no discount, same as any other gap."
  Changed Tier #1's Min quantity from 1 to **5** (creating a sub-Tier-1
  gap: qty 1-4 falls in no tier), tested with quantity **3** → full price
  $60.00 (`3 × $20.00`, no discount applied) — same pattern as the
  mid-table gap test above. Tier #1's Min was restored to 1 afterward.
- Total quantity **above** the highest tier's max, with Max explicitly
  set — **Resolved (QA-tested live 2026-09-09):** the highest tier's
  rate keeps applying to the entire quantity regardless — an explicit Max
  on the last tier does **not** cap or block the discount. The "leave Max
  empty for unlimited" hint text is misleading — see Business Rules.
- Whether the drawer (QVD-143) exists on any theme, or the inline
  rendering seen on Dawn is universal — **checked 2026-09-09: no usable
  second theme on `test-vl-1`** to test against (see Storefront section
  above for why). Still open.
- Drawer behavior with only **one** variant type configured (e.g. just
  "Size", no Color/Material) — the "last variant gets quantity steppers"
  rule would also make it the *first* variant; does the badge-counter
  logic still make sense, or degrade to something simpler?
- Drawer behavior with variants that have **many** values (e.g. 20+
  colors) — does the badge/stepper UI scale/scroll reasonably, or was it
  designed assuming a small variant count?
- Table layout (QVD-156) — **partially resolved (QA-tested live
  2026-09-09):** switching the Appearance & Style → Layout carousel to
  the 2nd preset (table layout) and saving **does render correctly on the
  storefront** — bordered table with header row "QUANTITY | DISCOUNT |
  ITEM PRICE", tier rows like "5-10 Items | — | From $20.00/Item". Layout
  was reverted back to the 1st (card) preset afterward to keep the offer
  in its default state. **Still open:** whether the switch truly applies
  without a full page reload (not specifically isolated in this test) and
  whether the table layout reflects the same Color/Typography/border-radius
  settings as the card layout — not yet independently verified.
- Interaction with [Onboarding Dashboard](01-onboarding-dashboard.md)'s
  Theme Extension Status / auto-discount gating (QVD-167, confirmed for
  Quantity Break) — does Wholesale's discount Function share the same
  theme-embed gating, or was that fix scoped only to Quantity Break?
  `Need confirm`, given it was a targeted incident fix, not obviously a
  shared library change.
- Interaction with [Multibuy](04-multibuy-mix-match-combo.md) /
  [Quantity Break](03-quantity-break.md) when a product's trigger overlaps
  across offer types — **partially resolved (QA-tested live 2026-09-09,
  product "Copper Light", all 3 offer types triggered simultaneously):**
  Wholesale does participate in the same "not both apply" pattern —
  see the 3-way overlap test in
  [feature-map.md](../product/feature-map.md). Added Copper Light to
  Wholesale offer 818's "Specific products" trigger (already covered by
  Quantity Break's Offer 15 and Multibuy's Offer 16); all 3 widgets
  rendered together on the PDP. Setting **quantity = 3 via Wholesale's own
  quantity stepper + Add to cart** resulted in the cart line getting
  **Quantity Break's** "Buy 3 save 20%" discount ($48.00/item, discount
  code `QIKIFY-5WBE1R`) instead of Wholesale's own Tier 1 rate. **Caveat:
  not a fully clean priority test** — at qty=3, Wholesale's own Tier 1
  (1-10 items) is 0% off anyway, so there was no real conflict between two
  *active* discounts, just Quantity Break's discount applying on top of
  what would've been Wholesale's full-price tier. `Need confirm PO/Dev`:
  a cleaner test (quantity landing in a Wholesale tier with a *nonzero*
  %, e.g. 12-20 units, compared against a Quantity Break/Multibuy tier
  that also qualifies) is still needed to properly resolve priority
  between Wholesale and the other two offer types.

## Advanced Settings

Shares the [Custom CSS](../product/custom-css.md) input field with other offer types.

## Known Issues

See [known-issues.md](../known-issues/known-issues.md) for entries tagged `wholesale-pricing-offer`.

## Related Jira Tickets

- [QVD-142](https://ownego.atlassian.net/browse/QVD-142) — Wholesale pricing tier, Epic — problem statement / goals
- [QVD-143](https://ownego.atlassian.net/browse/QVD-143) — [SDK] Wholesale pricing tier, Story — PDP tier cards + Variant Selection Drawer spec
- [QVD-150](https://ownego.atlassian.net/browse/QVD-150) — [Portal] Wholesale Pricing Tier — Offer Settings, Story — full portal settings spec (tier table, appearance, combination, scheduling)
- [QVD-156](https://ownego.atlassian.net/browse/QVD-156) — [SDK] Wholesale - Table layout, Story — adds table layout alternative to card layout
- [QVD-180](https://ownego.atlassian.net/browse/QVD-180) — [Support] Wholesale - Minimum quantity enforcement, Task — status **Passed**; shipped as per-store custom config (`ENABLE_WHOLESALE_QTY_ENFORCE_OPTION`), piloting with select merchants, not yet a general Portal feature

## Source

- Jira (QVD project), via Atlassian MCP, analyzed 2026-09-07.
- Help center (may be outdated): https://help.qikify.com/en/article/wholesale-pricing-offer-18jsbca/
