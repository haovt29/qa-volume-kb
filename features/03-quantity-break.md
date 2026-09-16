# 3. Quantity Break

**Status:** in-review

Umbrella feature: reward customers for buying more units. What used to be
listed on the help center as three separate articles are variants of the
same underlying mechanic, applied to different contexts — merged here per
QA (2026-08-27) to avoid documenting shared business rules three times.
See [decisions/decision-log.md](../decisions/decision-log.md). Analyzed
against Jira 2026-08-28, updated 2026-09-03 (QVD-23, QVD-25,
QVD-26, QVD-28, QVD-41, QVD-72, QVD-82, QVD-85, QVD-90, QVD-92, QVD-94,
QVD-116, QVD-124, QVD-127, QVD-128, QVD-140, QVD-148, QVD-167 — see
Related Jira Tickets below), then **live-verified 2026-09-03 via Playwright
MCP** against store `test-vl-1` (offers "mi testing" [price-break variant],
"Free gift" [free-gift variant], "B collection - subscription" [subscription
variant]) — several ticket specs were found to not match shipped behavior
1:1 and are called out inline as "corrected"/"discovered live". Business
Rules and Edge Cases without a live-verification note are still
ticket-spec-only and not yet independently confirmed.

## Purpose

Reward customers for buying more, via tiered quantity/price rules (QVD-23).
The reward can be a price break (percentage/fixed discount — "Quantity
breaks for same product" template below), a free/discounted gift item
instead of a price cut ("Buy X, get Y free"), or the same tier
mechanic layered on top of a subscription purchase ("Volume discount for
subscription"). All three share one underlying tier-settings
engine (QVD-23) — they differ in what's configured *inside* a tier and,
for subscriptions, in extra purchase-option settings.

**Naming history:** the KB originally labeled these "Same-product tiers" /
"Buy X Get Y" / "Subscription products" (inherited from old help-center
article titles). That turned out to be misleading, because **trigger
scope** (how many/which products an offer applies to — see Trigger
Settings below) and **reward type** (price break vs. free gift vs.
subscription overlay) are two **independent** axes: an offer using the
"Quantity breaks for same product" reward template is **not** inherently
limited to one product — it can use any of the 7 triggers below, including
broad multi-product ones like "All products." QA confirmed the templates'
actual current names on 2026-09-03 (see Related Jira Tickets / decision-log
2026-09-03); the section headings below now use those exact names.

## Trigger Settings

**Structure corrected 2026-09-03 via live Portal check (store test-vl-1,
offers "mi testing" / "Free gift" / "B collection - subscription")** —
supersedes the "7 separate options" framing below it. The UI is actually
**5 radio buttons**, not 7:

- **All products**
- **Collections**
- **Product tags**
- **Specific products** (or variants — confirmed live: picking a product
  can be narrowed to specific variants within it, e.g. "6 of 6 selected")
- **Custom trigger** — "Custom triggers" modal (QA screenshot, 2026-09-03):
  "Only display the offer when: **All rules pass** / **Any rule pass**"
  (radio, controls how multiple trigger rows combine — AND vs OR). Each
  row: a trigger-type dropdown, a comparison operator (e.g. "in"), and a
  value picker (Browse/select). Multiple rows addable via "+ Add trigger",
  shown as a running count ("N trigger set"). **Trigger-type dropdown
  options, exact list from the screenshot:** Products, Product types,
  Product vendors, Product tags, Collections, Customer login, Customer
  tags, B2B Customer, Markets. (Corrects an earlier, less precise list in
  this file that included "new customer" — not present in the actual
  dropdown — and was missing Product types, Customer tags, Markets.)

**"Exclude" is not a 6th/7th radio option — corrected 2026-09-03:** it's a
separate **"Exclude products" link/button** that appears *underneath* the
radio list only when **All products** or **Collections** is selected
(confirmed live on both). Clicking it opens a product/variant picker to
exclude. Not available under Product tags, Specific products, or Custom
trigger. This file previously modeled exclude as 2 standalone trigger
choices ("All products – Exclude...", "Collections – Exclude...") — that
was an inaccurate simplification; the actual mechanism is trigger-radio +
optional exclude-link, not a distinct radio value.

**Collections trigger detail — Resolved (QA confirmed 2026-09-03):** shows
a "Select collections" search field plus a "N collection(s) selected"
summary. Unlike Multibuy (confirmed single-collection limit), **Quantity
Break's Collections trigger is not limited to 1** — merchants can select
multiple collections for a single offer.

- **Exclude behavior — Resolved (QA confirmed 2026-09-03):** excluding
  a product/variant does **both** — the offer is **not displayed** for
  that product/variant at all, **and** it never gets the discount applied
  even if its quantity would otherwise meet a tier threshold. It's a full
  exclusion, not just a discount-eligibility filter with the widget still
  showing.
- **Trigger-overlap warning — discovered live 2026-09-03, not in any
  ticket found:** if this offer's trigger includes a product already
  covered by another offer's trigger (any offer type, not just Quantity
  Break), Portal shows a warning: *"This trigger overlaps with offer(s):
  &lt;name&gt;. Overlapping triggers may cause discount conflicts. Please
  consider carefully whether to adjust this trigger."* **Non-blocking** —
  merchant can save anyway. Confirmed both widgets then render
  simultaneously on the same product page. See
  [feature-map.md](../product/feature-map.md) Known interactions for the
  full cross-feature writeup and the still-open question of what happens
  at checkout when both offers' conditions are met at once.

**Confirmed quantity-counting behavior — corrected 2026-09-03, supersedes
an earlier 2026-09-03 statement in this file that said "per line item":**
quantity is pooled **per product** — summed across that product's own
variants in the cart — but **not** pooled across *different* products.
**QA confirmed 2026-09-03 this applies to all 7 trigger types above, not
just "All products"** — same per-product pooling regardless of which
trigger scope selected the product into the offer in the first place (so
even under "Collections" or "Custom trigger", two different products in
the same collection still do **not** pool quantity with each other; only
a single product's own variants pool together). Verified against a real
cart screenshot (QA-provided, 2026-09-03) using an "All products" trigger:
Tier 1 = 5 packs/10% off; cart had product "Test swatch" in 3 variants
(qty 3 + 2 + 3 = 8, pooled within that one product) → all 3 variant lines
got 10% off, while a separate product "Chuột swatch" at qty 4 alone
(below the 5-unit threshold, and not pooled with "Test swatch" since it's
a different product) got no discount at all. The earlier verbal answer
("tính riêng từng product/variant") was imprecise — it's per-*product*,
not per-line/per-variant.

## Variants

**Section headings below are the 3 template cards' current exact names,
confirmed by QA 2026-09-03** (QVD-116's screenshot is outdated/mixed in a
Multibuy card instead of the subscription one; QVD-140 added the
subscription template separately — treat the headings below as current
ground truth, not QVD-116's screenshot). Clicking "Create" on any of them
opens the **same** tier settings page;
each template just pre-fills the reward side differently (e.g. the
subscription template pre-enables the "Subscriptions" toggle).
[Trigger Settings](#trigger-settings) is a separate, independent setting
available inside any of the 3 templates — picking "Free Gift" doesn't
restrict which trigger you can choose, and vice versa.

### Quantity breaks for same product

(KB called this "Same-product tiers" before QA gave the exact template
name on 2026-09-03; formerly "Quantity Breaks for Same Product Offer" on
the old help center) — tiered pricing, reward scales with quantity
purchased per the Trigger Settings scope (see above). This is the base
tier engine (QVD-23) that the other two variants build on.

- **Multiple tiers — field list corrected/expanded 2026-09-03 via live
  Portal check (offer "mi testing"):** merchant adds multiple discount
  tiers (e.g. Buy 3 → 10% off, Buy 5 → 15% off). Each tier has: **Title**,
  **Subtitle (optional)**, **Badge**, **Discount type** (dropdown, exact
  labels **"Percentage" / "Fixed Amount"**, default Percentage),
  **Discount value (per item)** (positive number > 0; percentage capped
  at ≤100% — field label explicitly says "per item," confirming the
  calculated rate applies per unit, not as one lump sum), minimum
  **Quantity** (required, positive integer, must be greater than the
  previous tier's quantity), **Label type** (dropdown — live-confirmed
  exact list: **Classic, Oval, Tag, Callout**; earlier "label shape"
  wording in this file was vague, this is the real option set), **Label
  text**, an **Image** upload (per-tier, via "Choose File"/"Add image" —
  not previously documented), and **"Selected by default"** (checkbox).
- **Default state on first load:** 3 pre-populated tiers to preview live:
  Tier 1 = qty 1 / 0% / title "1 pack"; Tier 2 = qty 2 / 10% / title
  "2 packs" / subtitle "Buy 2 save 10% off" / label "Flat" text "Most
  popular" / badge "Free shipping" / **selected by default**; Tier 3 =
  qty 3 / 20% / title "3 packs" / subtitle "Buy 3 save 20% off" / badge
  "Free shipping". Only **one tier** can be marked "selected by default"
  at a time — checking it on one tier auto-unchecks any other.
- **Expand/collapse:** Tier 1 expanded by default, Tiers 2–3 collapsed.
  Only one tier can be expanded at a time — clicking a collapsed tier (or
  its Edit button) expands it and auto-collapses whichever tier was open.
  Collapse/Edit/Delete buttons are mutually exclusive by state (Collapse
  shows only while expanded; Edit and Delete show only while collapsed).
- **"+ Add new tier":** appends a new tier below the existing ones,
  auto-expanded, auto-assigned defaults (next tier number, quantity =
  previous + 1, blank/empty discount fields); collapses whatever tier was
  previously open.
- **Guardrail:** at least one tier must always exist — merchants cannot
  delete the last remaining tier. If somehow no tier is configured, no
  discount is applied.
- **Discount calculation — Resolved (QA confirmed 2026-09-03), not
  documented in any ticket found so far:** tiers are **not**
  progressive/incremental bands. The rule is: find the **highest tier
  whose quantity threshold the purchased quantity meets or exceeds**, and
  apply that tier's discount rate to the **entire quantity purchased** —
  never a split across bands. This applies both mid-range and above the
  top tier. Example with Tier 1 = qty 5 / 10% off, Tier 2 = qty 10 / 25%
  off:
  - Buying **7** units (meets Tier 1, not Tier 2) → all 7 units get 10%
    off (Tier 1's rate), not 0% and not a partial split.
  - Buying **20** units (exceeds Tier 2, the highest) → all 20 units get
    25% off (Tier 2's rate) — no tier beyond Tier 2 to step up to.
  - Buying **4** units (below Tier 1) → no discount at all.
- **End-to-end storefront test — confirmed live 2026-09-03:** created a
  real offer ("Offer 15") on store `test-vl-1`, trigger = Specific
  products (single-variant product "Copper Light," $60 base price),
  Tier 1 = qty 1/10%, Tier 2 = qty 2/10% (default), Tier 3 = qty 3/20%
  (default), Automatic discount method. On the live storefront preview:
  - Widget rendered correctly with live pricing exactly matching the
    Portal preview (1 pack $54.00, 2 packs $108.00/Free shipping/Most
    popular, 3 packs $144.00).
  - Selected "2 packs," clicked Add to cart → cart showed 2× at **$54.00
    each** (10% off $60, Tier 2's rate) = **$108.00** — automatic
    discount applied immediately in cart, no checkout step needed to see
    it, discount code `QIKIFY-5WBE1R` shown as applied.
  - **Increasing quantity directly in the cart (2 → 3) auto-recalculated
    to Tier 3's rate live** — price dropped to $48.00/unit (20% off) ×
    3 = **$144.00**, with no re-visit to the product page needed. This
    confirms the discount Function evaluates cart state dynamically in
    real time, not a rate "locked in" at whichever tier was selected at
    Add-to-cart time.
  - **Known issue, now logged centrally:** a raw, unrendered-looking text
    string `{ [Default Title]: "", }` appeared on the product page
    directly below the Add to cart/Buy it now buttons. Confirmed
    2026-09-07 as the same bug pattern also present on Wholesale Pricing
    — see [known-issues.md](../known-issues/known-issues.md)
    (`quantity-break, wholesale-pricing-offer`) for the full writeup;
    likely a shared SDK-layer bug, not Quantity-Break-specific.
  - **Not yet tested from this session:** whether disabling the theme
    app embed actually blocks the auto-discount (the QVD-167 regression
    test) — this session only verified the discount fires correctly with
    the embed presumably enabled; didn't toggle it off and repeat.
  - **Unresolved tension flagged, not yet reconciled:** the offer-creation
    screen shows a warning — *"Automatic discounts work even if the app
    block is disabled. For testing, we recommend using Discount code to
    prevent customers from receiving unintended discounts."* — worded
    live, 2026-09-03. This reads as contradicting the QVD-167 fix
    documented below (auto-discount gated on theme-embed status,
    previously QA-confirmed via regression test). Possible explanations,
    none confirmed: (a) "app block" (a specific theme section block) and
    "theme embed" (the whole app's embed toggle) are different Shopify
    concepts and this warning is about a narrower case QVD-167 doesn't
    cover; (b) this banner is stale copy predating the QVD-167 fix; (c)
    genuine gap. `Need confirm PO/Dev` before writing regression test
    cases against either claim as ground truth.
- **Quantity counting under the "All products" trigger — corrected
  2026-09-03 (supersedes a wrong 2026-09-03 statement below):** see
  [Trigger Settings](#trigger-settings) above for the full list of trigger
  options and the naming-conflict flag. Under "All products", quantity is
  pooled **per product** — summed across that product's own variants in
  the cart — but **not** pooled across *different* products. (An earlier
  version of this file said "per line item," based on a verbal answer that
  turned out to be imprecise; corrected against an actual cart screenshot,
  see below.)
  - **Verified example (real screenshot, QA-provided 2026-09-03):** Tier 1
    = qty 5 / 10% off. Cart has product "Test swatch" in 3 variants (Pink
    ×3, Brown ×2, "hướng dương" ×3 — none individually ≥5) plus an
    unrelated product "Chuột swatch" ×4. "Test swatch" variants pool to
    3+2+3 = **8** (≥ Tier 1's 5) → **all 3 "Test swatch" lines get 10% off
    their own price** (e.g. Pink €35.00 → €31.50, Brown €20.00 → €18.00,
    "hướng dương" €10.00 → €9.00 — each line discounted on its own price,
    not a blended price across variants). "Chuột swatch" ×4 stays at **0%
    off** — its quantity is not pooled with "Test swatch" since it's a
    different product, and 4 alone doesn't reach Tier 1.
  - **Hypothetical, corrected:** Tier 1 = qty 5/10%, Tier 2 = qty 10/25%,
    Tier 3 = qty 15/$70 off per item. Cart has Product A/Medium ×2,
    Product A/Small ×4, Product B ×4. Product A pools to 2+4 = **6** (≥
    Tier 1, < Tier 2) → **both Product A lines get 10% off**, each on its
    own price. Product B ×4 alone stays **below Tier 1** (not pooled with
    Product A) → **0% off**.
- **Validation:** real-time inline errors for rule violations; success
  toast/inline message shown after Save.
- **Storefront widget design (QVD-41):** no custom app "Add to Cart"
  button — integrates with the **theme's native ATC button**. Trigger
  logic is per product **variant**, not whole product. Merchant toggles
  control whether the widget shows/hides its own quantity selector and
  variant selector (if hidden, the widget relies on the theme's native
  selectors instead). If the widget's variant selector is shown, it
  **syncs dynamically** with the theme's native variant selector (e.g.
  shopper picks Size=S in the theme dropdown → tier widget reflects that
  variant/pricing).
- **Dynamic product slots (QVD-127):** storefront shows "slots"
  representing units needed to reach the next tier, revealed incrementally
  as items are added to cart. Example with tiers at 3/5/8 units: starts
  with 3 empty slots; on reaching 3, shows 2 more (to reach 5); on
  reaching 5, shows 3 more (to reach 8); on reaching 8 (max tier), no more
  slots shown and the top tier is highlighted as unlocked.
- **Tier settings panel layout fix (QVD-72):** the tier panel previously
  had a "card-in-card" layout issue (nested cards, referenced against
  Shopify's Checkout > Custom Field pattern as the target look) — fixed to
  a flatter layout. Also fixed: at narrower/minimized browser widths, the
  Setting Offer page had inconsistent spacing between input fields and
  oversized default-value font (e.g. "Offer 1", "Buy more, save more")
  compared to full width. **Resolved requirement:** spacing and font size
  must stay consistent and the layout fully responsive across viewport
  widths — worth a visual regression check at narrow browser widths.

### Buy X, get Y free

(KB called this "Free gift reward (Buy X Get Y)" before QA gave the exact
template name on 2026-09-03; formerly "Buy X Get Y" on the old help
center) — not a separate offer-creation flow: it's the same tier engine
above, with a free gift attached to a tier instead of (or alongside) a
price cut (QVD-82 epic, QVD-92 portal story). Any tier can have a gift
attached, not just a dedicated "top" tier.

- **"Add free gift" button** inside a tier's settings reveals the gift
  settings card.
- **Gift picker:** search-and-multi-select. Typing in the search box opens
  a **Gift Selection Modal** with the search term prefilled and live
  results; clicking "Browse" opens the same modal without a prefilled
  term. Merchant can select multiple product **variants** as gifts from
  either path.
- **Selected gifts list** shows per gift: product image, product name, an
  editable required quantity field (default 1), and a delete button. If a
  multi-variant product is added without picking specific variants, only
  the product name is shown (no variant breakdown) — the configured
  quantity applies uniformly to **whichever variant the shopper ends up
  receiving** (e.g. gift set to qty 2 on a shirt with S/M/L → shopper gets
  2 units of whatever size they qualify for).
- **Promotion message:** optional free-text field shown on the storefront
  widget (e.g. "+ Free Gift Bottle").
- **"Show gift's price" toggle:** checked → the gift's original price
  shows next to it on the storefront; unchecked → price is hidden.
- **"Remove" button:** clears all gifts from that tier, reverting the UI
  back to just the "Add free gift" button.
- **Storefront preview confirmed live 2026-09-03 (offer "Free gift," 2
  products trigger):** each tier with a gift shows a gift count badge on
  the tier itself (e.g. "1 gift," "2 Gifts," "3 gifts" — casing was
  inconsistent between tiers in the live example, worth a copy-consistency
  check) plus a separate line below the tier listing the gift's own
  price struck through with **"Free"** (e.g. "1 gift — $10.00 Free," "3
  gifts — $125.00 Free"). Tier subtitles also appended the gift count
  automatically, e.g. "Buy 2 save 10% off - 1 gift." Multiple gifts in
  one tier (tier showing "3 gifts") displayed as a combined price/Free
  line rather than 3 separate lines — exact per-gift breakdown when a
  tier has multiple different gift products not yet inspected in detail.

### Volume discount for subscription

(KB called this "Subscription products" before QA gave the exact template
name on 2026-09-03; formerly "Volume Discounts for Subscription Products"
on the old help center) — same tier mechanic applied where the purchased
item is a subscription, layered with subscription purchase-option settings
(QVD-94 SDK, QVD-128 Portal, QVD-140 entry point).

- **Eligibility:** the subscription widget/quantity-break-for-subscription
  UI is shown **only for products eligible for subscription** (i.e. the
  product supports subscriptions via a subscription app); hidden
  otherwise.
- **"Subscriptions" toggle** in the quantity-break setup, **default
  OFF/disabled**: OFF → subscription widget hidden on storefront, tiers
  apply to one-time purchase only. ON → subscription UI/logic activated
  for that offer.
- **Info banner (when enabled) — verbatim confirmed live 2026-09-03
  (offer "B collection - subscription"), exact match to the ticket:**
  "Subscription widget will only be shown in products that are eligible
  for subscription. Subscription app discounts are applied first,
  followed by other discounts. This may slightly affect the final price."
- **"Recommended app" card — discovered live 2026-09-03, not in any
  ticket found so far:** directly below the info banner, a card
  recommending **Appstle℠ Subscriptions App** ("Easily create, manage
  subscriptions and subscription boxes...") with a "Free to install"
  button. Confirms the app doesn't provide its own subscription
  infrastructure — it depends on a separate subscription app (Appstle
  suggested, but presumably any compatible subscription app works, per
  the generic "subscription app discount" wording elsewhere). The live
  example's Preview showed a working purchase-option dropdown pulling
  real plan data from Appstle ("10% off," "10$ off," "fixed price 10$"),
  confirming this integration is live/functional in this test store.
- **Purchase option title** — field, live default "Purchase options";
  this is the section heading shown to shoppers above the
  Subscribe/One-time toggle (distinct from the Subscription/One-time
  *titles* below).
- **Position** (renamed from "Widget position" — live field label is
  just "Position"): merchant chooses **Above volume tiers** or **Below
  volume tiers** for where the Subscribe/One-time purchase picker sits
  relative to the tier widget. Ticket/earlier draft of this file said
  default is "Above" — the live example had it set to **Below**, so true
  out-of-the-box default is unconfirmed from this one configured offer.
- **Customizable labels — exact live field labels confirmed:**
  **Subscription title** (default "Subscribe & save"), **Subscription
  subtitle** (empty by default), **One-time purchase title** (default
  "One time purchase"), **One-time purchase subtitle** (empty by
  default).
- **Default purchase selection — exact live field label: "Selected
  subscription plan by default"** (checkbox, checked in the live
  example) — preselects "Subscribe & Save" when the product page loads;
  shopper can still switch to one-time purchase manually. Quantity-break
  tiers stay selectable under either purchase type.
- **Discount stacking order (business rule, affects pricing
  correctness):** the **subscription app's own discount** (e.g. a
  subscription app's "Subscribe & save 20%") is applied **first**; this
  app's **quantity-break discount** is applied **on top of that**.
  **Formula:** `Final price = (Base price − Subscription discount) −
  Quantity break discount`. Subscription discounts always take priority
  over this app's discount, so the final price on a subscription tier can
  legitimately differ from the same tier on a one-time purchase.
- **Entry point (QVD-140):** the "Volume discount for
  subscription" card on the offer type list opens the **same**
  quantity-break settings page used for "Quantity breaks for same
  product," with the "Subscriptions" toggle **pre-enabled by default** —
  it is not a separate settings screen.
- **Parent epic (corrected 2026-09-03):** all of the above (QVD-94, 128,
  140) are children of [QVD-124](https://ownego.atlassian.net/browse/QVD-124)
  "Subscription integration" (Epic). Earlier draft of this file
  mischaracterized QVD-124 as "not yet built" because the **epic itself**
  still shows status **To Do** — but 3 of its 4 children are actually
  **Done** and already shipped (the behavior described above). Epic status
  in this project doesn't appear to auto-roll-up from children, so don't
  read an epic's own status as proof nothing under it shipped — check its
  children.
- **[QVD-148](https://ownego.atlassian.net/browse/QVD-148) "New UI for
  Subscription (SDK)" — Resolved (QA confirmed 2026-09-03): shipped**,
  even though the Jira ticket itself still shows status **To Do** with no
  description (same stale-status pattern as the QVD-124 epic above — the
  ticket metadata hasn't caught up to reality). **Gap:** since the ticket
  has no description, this file has no detail on what specifically
  changed in the new SDK UI — `Need input from QA/Dev` on what's actually
  different from the QVD-94 spec above before writing test cases against
  it.

## Shared / cross-variant rules

Rules below live on the same tier-settings page as all 3 variants above.
**Flagging rather than deciding unilaterally:** the tickets describe these
as generic "tier settings page" behavior, and Multibuy has its own
near-identical "Tier settings" ticket (QVD-105) — these may be shared
across *all* offer types, not exclusive to Quantity Break. Confirm with QA
before assuming identical behavior on
[Multibuy](04-multibuy-mix-match-combo.md) /
[Wholesale Pricing](05-wholesale-pricing-offer.md); if confirmed identical,
consider moving to `product/` per the Custom CSS precedent (see
decision-log 2026-08-27) and cross-linking from `feature-map.md` instead.

- **Display settings (QVD-28) — corrected 2026-09-03 via live Portal
  check (all 3 variants: "mi testing", "Free gift", "B collection -
  subscription" show the identical structure):** heading "Display
  settings" with **2** visible sub-groups, **not 3** — **Variant display**
  and **Pricing display**. No "Content" group was visible in any of the 3
  live offers checked; QVD-28's "Content" accordion section is either not
  shipped, not applicable to this offer type, or gated behind a condition
  not yet found — don't assume it exists until confirmed otherwise.
  - **Accordion behavior claim not confirmed live:** the 2 groups
    rendered as plain always-visible sections with their checkboxes shown
    directly underneath — no expand/collapse arrows or "only one open at
    a time" behavior were visible in the accessibility snapshot. QVD-28's
    "collapsed by default, one at a time" spec may not match shipped
    behavior; needs a visual/interactive check (this session read the
    accessibility tree, not a live click-to-expand test).
  - **Variant display:** **"Allow customers to choose different variants
    for each item"** (checkbox, confirmed **unchecked by default** live)
    and **"Hide theme variant picker"** (checkbox, confirmed **unchecked
    by default** live, independent of the first). Neither "Allow customer
    to choose variant for single-item offer" nor "Variant swatch
    settings" (both documented in QVD-28 as conditionally shown only when
    the first checkbox is on) were visible in any of the 3 live examples
    — consistent with the ticket's stated condition (all 3 had the first
    checkbox off), not necessarily wrong; still unconfirmed since not
    directly toggled on and observed.
  - **Pricing display — new field names confirmed live, not in QVD-28's
    original ticket text:** **"Show price per unit"** and **"Use product
    compare-at price"** (both checkboxes, both confirmed unchecked by
    default live).
  - **Cross-reference, still flagging:** the (unconfirmed) "Variant
    swatch settings" button may overlap with the standalone [Variant
    Swatches](06-variant-swatches.md) feature — still needs direct
    confirmation once someone enables the parent checkbox to make it
    visible.
- **Appearance & style — discovered/confirmed live 2026-09-03, not
  documented at this level of detail in any ticket found, identical
  across all 3 variants checked:** an accordion with 5 groups —
  **Layout** (Card border radius slider, live default 8px), **Color**
  (**Primary color**, **Secondary color**, **Label background color**,
  **Label text color** — 4 color pickers; live defaults #2196F3 / #262626
  / #FC81C7 / #FFFFFF), **Typography** (4 sub-groups — **Heading**,
  **Body**, **Badge**, **Label** — each with a Font weight dropdown +
  Font size number field; live defaults: Heading Bold/18px, Body
  Regular/14px, Badge Regular/12px, Label Bold/12px), **Badge** (Badge
  text — live default `UP TO {discount} OFF`, single-brace variable
  syntax, shown on product image on collection/recommendation pages;
  Badge position dropdown, live default "Top left"; Badge border radius
  slider, live default 6px), and **Custom CSS** (textarea gated behind
  "contact our support team to enable" — **not self-serve**, same
  support-gated pattern confirmed for Multibuy — see
  [Custom CSS](../product/custom-css.md)). **Note the color/typography
  field set differs from Multibuy's** (Multibuy has Primary/Secondary/
  Accent colors and Heading/Body/Badge typography — no "Label" group,
  since Multibuy doesn't have the same per-tier Label feature) — these
  are offer-type-specific, not a shared component.
- **Discount combination (QVD-25):** checkboxes to let this offer's
  discount combine with Order discount / Product discount / Shipping
  discount — all **default unchecked (off)**. Located under "Combination"
  on the tier settings page.
- **Offer scheduling (QVD-26) — "Enable countdown timer" confirmed live
  2026-09-03, not in QVD-26's ticket:** start date + time (required,
  defaults to offer-creation time, cannot be in the past, uses the
  store's timezone) and an optional end date + time (behind an "Enable
  end date" checkbox, default off = runs indefinitely). If enabled: end
  date must be ≥ start date; if start date = end date, end time defaults
  to 11:59 PM (start and end time can't be equal on the same day). A
  separate **"Enable countdown timer"** checkbox also sits in this
  section (unchecked by default in all 3 live examples) — same field
  confirmed to exist for Multibuy too; likely ties to
  [QVD-147](https://ownego.atlassian.net/browse/QVD-147) "Countdown timer
  to offer," not yet analyzed in depth for either file.
- **Product option / personalization app integration (QVD-90):**
  quantity-break discounts must calculate correctly alongside
  personalization/customization surcharges from supported product-option
  apps (Tepo, Globo, OPTIS, Easify, Avis, Variant Option, Infinite, Hulk,
  LPO) — no double-counting or missed discount, both charges shown
  clearly at cart/checkout.
- **Portal-wide label/breadcrumb polish (QVD-85, sub-task of QVD-72,
  portal-wide — not Quantity Break-specific):** confirms exact field
  label casing on the tier settings UI: **"Discount type"** and
  **"Discount value"** (sentence case, not Title Case as earlier
  mockups showed). Also added breadcrumbs to Dashboard, Manage offers,
  Setting page, Translations, and Variant swatches pages, and fixed
  casing elsewhere (Manage offers, Create offer, Internal name, Preview
  section, Connected products). Listed here because it touches the Tier
  setting labels directly; the breadcrumb/casing work otherwise applies
  across the whole portal, not just Quantity Break.
- **Auto-discount gated on theme-embed status (QVD-167, bug fix —
  cross-ref [Onboarding Dashboard](01-onboarding-dashboard.md) Theme
  Extension Status):** previously, the discount Function applied based on
  cart contents alone with **no check** on whether the app's theme embed
  was actually enabled — this caused a real incident (merchant
  "Supersport" had 77 live orders discounted while believing the app was
  still inactive/in testing). Now gated: the discount Function checks
  theme-embed status and **skips applying the discount** if the app block
  isn't enabled in the theme editor, even when cart contents match a
  qualifying tier. This is the mechanism behind whether the Theme
  Extension Status block (Active/Inactive) can be trusted — worth a
  dedicated regression test given there's a real production incident
  behind it.

## Edge Cases

_Drafted from ticket specs — not yet QA-confirmed, treat as open questions
the same way 01/02's edge cases started._

- Setting a tier's quantity ≤ the previous tier's quantity — **Resolved
  (QA confirmed 2026-09-03):** shows an inline error message and focuses
  the offending quantity field directly — not an auto-adjust.
- Attempting to delete the last remaining tier — **Resolved (QA confirmed
  2026-09-03):** the Delete button is **hidden** when only one tier
  remains — the guardrail is enforced by removing the affordance, not by
  showing an error after the fact.
- Discount value at the boundary — **Resolved (QA confirmed 2026-09-03),
  4 sub-cases:**
  - **0%:** allowed, not blocked. Valid use case — a merchant may
    deliberately want a tier with no discount, e.g. "buy 1 = no discount"
    as the baseline before higher tiers kick in.
  - **Negative:** not blocked via keyboard/paste — **known issue**, see
    [known-issues.md](../known-issues/known-issues.md) (`quantity-break`
    tag) for details rather than duplicating here.
  - **Blank:** shows an inline "field required" error.
  - **Exactly 100%:** allowed.
- A tier at 100% off **and** with a free gift attached — **Resolved (QA
  confirmed 2026-09-03):** yes, the gift still applies on top of a 100%-off
  tier. Not blocked — it's a legitimate merchant setting choice (some
  merchants intentionally want to give the product free **and** throw in
  a gift on top, e.g. as a loyalty/promo push).
- Free gift: selected gift product/variant goes out of stock before
  checkout — **Resolved (QA confirmed 2026-09-03):** the offer **hides**
  the free gift (rather than blocking checkout or showing an error).
- Free gift: multi-variant product added as gift without picking specific
  variants — **Resolved (QA confirmed 2026-09-03, verified with
  screenshot):** the configured quantity applies uniformly across all
  selected variants (as the ticket said), but only **one** variant
  actually gets added to cart. On the storefront widget, the gift banner
  ("+1 FREE GIFT") has a "Details" link that reveals exactly which variant
  will be added (e.g. "Mexico Organic Single Origin / 2lb Bag Whole
  B...", x1). **If the shopper doesn't explicitly pick a variant, it
  defaults to the first available variant** of that gift product.
- Subscription: "Subscriptions" toggle OFF — **Resolved (QA confirmed
  2026-09-03):** if the shopper buys via **another app's** subscription
  widget on the same product, this app's own quantity-break widget
  **auto-hides**. It only (re)shows when the shopper is on the **one-time
  purchase** path — no subscription UI leaks through from this app while
  another app's subscription flow is active.
- Subscription: shopper switches between "Subscribe & Save" and "One-time
  purchase" after already selecting a quantity-break tier — **Resolved
  (QA confirmed 2026-09-03): the tier selection resets** — it does not
  persist across the purchase-type switch.
- Subscription: the subscription app's own discount changes or is removed
  mid-session — **Resolved (QA confirmed 2026-09-03): does not recompute
  live** — the quantity-break price stays stale until the shopper reloads
  the page or navigates to a new page.
- Product-option/personalization app integration (QVD-90) — **On hold
  (QA, 2026-09-03)**, same status as the Sidekick note in
  [02-ai-offer-creation.md](02-ai-offer-creation.md) — not being
  tested/analyzed further for now; revisit when unblocked.
- Widget variant selector on a single-variant product / theme with no
  native variant selector — **Resolved (QA confirmed 2026-09-03):** no
  error, no blank screen — the widget simply **doesn't show the variant
  dropdown** at all in that case (degrades gracefully by omission).
- Theme-embed gating (QVD-167) regression — **Resolved/confirmed (QA,
  2026-09-03):** verified — with the app/embed disabled, a cart that
  otherwise qualifies for a tier does **not** get auto-discounted.
- Discount combination + Offer scheduling precedence — **Resolved (QA
  confirmed 2026-09-03):** Offer scheduling is evaluated **first** — the
  offer must be within its scheduled window to display/apply at all.
  Discount combination (Order/Product/Shipping discount stacking) only
  comes into play **after** that gate is passed; an offer outside its
  scheduled window never reaches the combination logic.

## Advanced Settings

Shares the [Custom CSS](../product/custom-css.md) input field with other
offer types — **confirmed live 2026-09-03:** gated behind "contact our
support team to enable," same as Multibuy — not self-serve by default.

## Known Issues

See [known-issues.md](../known-issues/known-issues.md) for entries tagged
`quantity-break`, `quantity-break/same-product`, `quantity-break/buy-x-get-y`,
or `quantity-break/subscription`.

## Related Jira Tickets

- [QVD-23](https://ownego.atlassian.net/browse/QVD-23) — Discount tier settings, Story — core tier engine shared by all 3 variants
- [QVD-25](https://ownego.atlassian.net/browse/QVD-25) — Discount combination, Story — Order/Product/Shipping discount toggles on the tier settings page
- [QVD-26](https://ownego.atlassian.net/browse/QVD-26) — Offer scheduling, Story — start/end date+time on the tier settings page
- [QVD-28](https://ownego.atlassian.net/browse/QVD-28) — [Portal] Variant eligibility settings, Sub-task — Display settings card (Variant/Pricing/Content); moved here from Multibuy file 2026-09-03, confirmed this feature not Multibuy's
- [QVD-41](https://ownego.atlassian.net/browse/QVD-41) — [SDK] show offer volume on product page, Sub-task — storefront widget design (native ATC, variant-based trigger)
- [QVD-72](https://ownego.atlassian.net/browse/QVD-72) — [Portal] Improve portal's discount settings, Improvement — tier settings panel layout/responsiveness fix
- [QVD-82](https://ownego.atlassian.net/browse/QVD-82) — Free gift, Epic — Buy X Get Y variant, high-level spec
- [QVD-85](https://ownego.atlassian.net/browse/QVD-85) — [Portal] Improve portal's UI, Sub-task (of QVD-72) — label casing + breadcrumb polish, portal-wide
- [QVD-90](https://ownego.atlassian.net/browse/QVD-90) — Product option integration, Story — compatibility with personalization/product-option apps
- [QVD-92](https://ownego.atlassian.net/browse/QVD-92) — [Portal] Free gift, Story — gift picker UI, detailed spec
- [QVD-94](https://ownego.atlassian.net/browse/QVD-94) — [SDK] Subscription purchase for quantity breaks, Story — discount stacking order, purchase option switching; child of QVD-124
- [QVD-116](https://ownego.atlassian.net/browse/QVD-116) — [Portal] Offer type list, Task — entry-point cards (context only)
- [QVD-124](https://ownego.atlassian.net/browse/QVD-124) — Subscription integration, Epic — status shows **To Do**, but 3 of its 4 children (QVD-94, 128, 140) are **Done**/shipped; only QVD-148 is genuinely pending — see note in Business Rules
- [QVD-127](https://ownego.atlassian.net/browse/QVD-127) — [SDK] Show product slots based on tier required quantity, Improvement — dynamic slot UI for same-product tiers
- [QVD-128](https://ownego.atlassian.net/browse/QVD-128) — [Portal] Subscription purchase for quantity breaks, Story — Subscription Settings section spec; child of QVD-124
- [QVD-140](https://ownego.atlassian.net/browse/QVD-140) — [Portal] Subscription template on offer type list, Task — entry point pre-enables Subscriptions toggle; child of QVD-124
- [QVD-148](https://ownego.atlassian.net/browse/QVD-148) — New UI for Subscription (SDK), Story — child of QVD-124, **To Do, not started**, no description
- [QVD-167](https://ownego.atlassian.net/browse/QVD-167) — Bug: Volume Discount auto-discount applies even when app is not enabled in theme editor, Improvement — theme-embed gating fix, real production incident

## Source

- Jira (QVD project), via Atlassian MCP, analyzed 2026-08-28, updated 2026-09-03.
- Help center (may be outdated, originally 3 separate articles — kept for
  reference only, superseded by the Jira analysis above):
  - https://help.qikify.com/en/article/quantity-breaks-for-same-product-offer-1fr75vy/
  - https://help.qikify.com/en/article/buy-x-get-y-17jb4y7/
  - https://help.qikify.com/en/article/volume-discounts-for-subscription-products-e1sghp/
