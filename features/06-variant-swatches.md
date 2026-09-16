# 6. Variant Swatches

**Status:** in-review

Analyzed against Jira 2026-09-09 (QVD-13 epic, QVD-46, QVD-49 — see Related
Jira Tickets below), then **live-verified 2026-09-09 and extended
2026-09-11 via Playwright MCP** against store `test-vl-1` (both Portal
and storefront/SDK). Several ticket specs were found to not match shipped
behavior 1:1 and are called out inline as "corrected"/"confirmed live".
**Most significant finding: the configured swatch (color/image) does not
actually render on the storefront — confirmed for 2 of the 5 swatch types
with real data, confirmed both on the theme's native variant picker and
inside the app's own offer widgets. See Storefront rendering below and
[known-issues.md](../known-issues/known-issues.md).**

## Purpose

A **standalone, global Portal page** ("Variant swatches" in the app's left
nav, alongside Manage offers/Translations/Analytics) that lets a merchant
configure how each **variant option** (e.g. "Color", "Size", "Material" —
the option *name*, not a specific offer) is displayed across the storefront
widget: default dropdown, or a color/image swatch (custom-configured or
auto-pulled from Shopify's own variant data). This is a store-wide,
per-variant-option setting — it is not scoped to one offer or one product.

Distinct from, but directly referenced by, [Quantity Break](03-quantity-break.md)'s
own per-offer "Variant display" settings (QVD-28) — see Cross-Feature
Interaction below.

## Business Rules

### Empty state (QVD-46) — confirmed live 2026-09-09

- Shown only when the merchant hasn't yet synced any product variants.
- Card: title **"Variant swatches"** (plural — ticket's mockup text said
  "Variant swatch," singular; live nav link, breadcrumb, and page heading
  are all plural), body text confirmed live verbatim: "To customize how
  your product variants appear, you'll first need to sync variant data
  from your Shopify store.", a primary **"Sync variants"** button.
- Clicking "Sync variants" pulls **all** available variants and variant
  values from Shopify into the app; a loading state shows while syncing
  (confirmed: button goes disabled + shows a loading spinner, and a
  separate "Loading variant swatches…" status line appears above the
  table area).

### Variant option table — live-verified 2026-09-09, several name corrections

- Columns — **live labels differ from the ticket:** **"Variant options"**
  (plural, not "Variant option"), **"Connected products"**, **"Swatch
  type"** (not "Display type" as QVD-46 said), **"Customize"**.
- `test-vl-1` had **13 already-synced variant options** at time of
  testing (e.g. Size/9 products, Material/3, Color/11, Variant/1,
  Shape/1, Type/1, Band color/2, plus several clearly test/junk option
  names like "Test Color", "Test", "test variant", "option" — real store
  data is messy, worth keeping in mind when writing test cases that
  assume clean option names).
- Swatch type values seen live on real rows: **"Default dropdown"**
  (most rows, i.e. never configured), **"Custom color dropdown"** (Type,
  Test Color), **"Custom image dropdown"** (Color) — see the 5-option
  list below; "Variant color dropdown" and "Variant image dropdown"
  exist as selectable options but weren't in use by any row on this
  store.
- Pagination: max 20 variant options per page (not exercised — only 13
  rows existed, all fit on one page; "Previous"/"Next" both rendered
  disabled).
- **Clicking "Customize" does NOT open a modal** — see next section.

### "Setting Variant Option" page — corrected 2026-09-09: NOT a modal

**Major correction from QVD-46's spec:** the ticket describes a
"Variant swatch settings" **modal**, with a close/cancel button and an
unsaved-changes confirmation dialog. **Live behavior: clicking
"Customize" instead navigates to a full dedicated page** at
`/apps/staging-bk-volume/variant-swatches/setting/<OptionName>` (e.g.
`.../setting/Color`), with its own breadcrumb ("Staging BK Volume >
Variant Swatches > Setting Variant Option") and page heading matching
the option name (e.g. "Color"). No modal, no overlay, no
close/cancel/unsaved-changes-confirmation UI observed.

- **"Connected products"** — shown again at the top of this page as a
  clickable count button (e.g. "11 products"), matching the table row.
- **"Swatch type" dropdown — confirmed live: 5 options, not 4, and
  renamed from the ticket's wording:**
  1. **Default dropdown** (ticket: "Theme default dropdown")
  2. **Custom color dropdown** (ticket: "Custom color variant dropdown")
  3. **Custom image dropdown** (ticket: "Custom image variant dropdown")
  4. **Variant color dropdown** — **not in the ticket at all.**
  5. **Variant image dropdown** (ticket: "Image variant dropdown" —
     auto-assigns Shopify's own variant images) — renamed, and now sits
     as the 5th/last option instead of 2nd.

  **Resolved (live-tested 2026-09-11):** selecting **"Variant color
  dropdown"** shows the live tooltip/helper text **"Automatically assigns
  the existing variant colors"** — confirming it's an auto-source option
  parallel to "Variant image dropdown" ("Automatically assigns the
  existing variant images", inferred from the matching pattern), just for
  Shopify-native variant colors instead of images. Neither has a
  customization table (no per-value rows) — both are pure "pull from
  Shopify, no manual config" modes, unlike the 2 "Custom ..." options.
- **Per-variant-value rows, shown below the Swatch type dropdown —
  content depends on the selected Swatch type:**
  - **"Custom image dropdown" (tested live on "Color", 11 products, 15
    variant values):** each row = a disabled "Variant option" textbox
    (the value name, e.g. "Brown", "Black", "beige") + either an
    **existing image thumbnail with a "Remove image" button** (values
    that already had one set: Brown, Black, White, Green, "stock = 2")
    or a **"Choose File" + "Add image" button pair** (values with no
    image yet: Beige, beige, brown, clear, gold, "Rose gold", Bronze,
    Pink, "Green stock = 2", "Out of stock").
  - **"Custom color dropdown" (tested live on "Type", 1 product, 3
    variant values — BK1, BK2, "Sold out"):** each row = disabled
    "Variant option" textbox + a color-picker swatch icon + a HEX
    textbox (placeholder `#FFFFFF`, empty on all 3 values at time of
    testing — none had been configured yet).
  - **Real-world edge case observed, not in the ticket:** the "Color"
    option's 15 rows included **case-only duplicates** — "Beige" is not
    listed but both "beige" (lowercase) and "Brown"/"brown" (both
    cases) appear as **separate rows** — confirming variant values are
    treated as distinct by exact string match, not normalized/merged by
    case. Also several rows look like leftover test data ("Green stock
    = 2", "Out of stock", "stock = 2") rather than real color names —
    a reminder that this store's product catalog has accumulated messy
    test variants over time.
- **"Preview section"** — live on both the "Color" and "Type" pages: a
  mocked **tier-card-style widget preview** (e.g. "Most popular / Pack 1
  / Free shipping / Saved 10% / $18.00 ~~$20.00~~"), with the currently
  configured variant option's value shown as a label under it (e.g.
  "Brown", "BK1"). Not documented in QVD-46 at all — the fact this
  reuses a Quantity-Break-style tier/pack card mockup (rather than a
  generic swatch preview) is a live-observed hint at how tightly this
  feature's UI is coupled to the offer-widget rendering, consistent with
  the QVD-28 cross-feature link below.

### Storefront rendering (QVD-49) — live-tested 2026-09-09, major bug found

- Ticket lists only **3** SDK-rendered swatch types: "Theme default
  dropdown", "Image variant dropdown", "Color swatch dropdown" — already
  known to be stale naming/counting relative to the Portal's actual
  **5**-option live list above (QVD-49 predates QVD-46/the live Portal
  build).
- **Major bug, live-confirmed 2026-09-09 on product "Color swatch"**
  (`test-vl-1`, handle `/products/color-swatch` — a product with **no**
  Quantity Break/Multibuy/Wholesale offer at all, so the theme's native
  variant picker renders unobstructed by any app widget): the "Color"
  option is configured as **"Custom color dropdown"** with real HEX
  values set (Beige `#eaabab`, Pink `#FFC0CB`, "stock = 2" `#000000`,
  Green `#05AA3D` — confirmed via the page's own debug-artifact text, see
  below) — **but the native variant picker still renders as plain text
  radio pills** ("Beige", "Pink", "stock = 2", "Green"), **not as color
  swatches**. Visually indistinguishable from an unconfigured option.
- **This reframes the recurring debug-artifact bug** (previously
  documented in [known-issues.md](../known-issues/known-issues.md) as
  "quantity-break, wholesale-pricing-offer") **as the swatch feature's
  own root cause, not a cosmetic side issue:** the raw text leaking below
  Add to cart on this product read `#eaabab { [Beige]: "#eaabab", } , {
  [Pink]: "#FFC0CB", } , { [stock = 2]: "#000000", } , { [Green]:
  "#05AA3D", }` — i.e. **the exact configured HEX values, computed
  correctly, but dumped as unrendered text instead of being turned into
  a visual swatch.** The swatch config data pipeline works; only the
  final "render as a colored circle/thumbnail" step is broken. See the
  updated, broadened known-issues.md entry (now tagged `variant-swatches`
  too, not just the 2 offer types where it was first spotted).
- **"Custom image dropdown" — resolved (live-tested 2026-09-11): same bug
  confirmed.** Product "White Ceramic Pot" (`test-vl-1`, no app offer at
  all — Size/Color options, "Color" globally configured as "Custom image
  dropdown" with real images already uploaded for values "White" and
  "Green") — storefront still rendered plain text radio pills ("White",
  "Green", "Beige"), not image swatches. Confirms the rendering failure
  is **not specific to "Custom color dropdown"** — it reproduces
  identically with real image data too. 2 of 5 swatch types now
  independently confirmed broken with real configured data.
- **"Variant color dropdown" / "Variant image dropdown" — tested
  2026-09-11, inconclusive (not confirmed broken or working).** Switched
  option "Type" (product "Test product") to each in turn and checked the
  storefront — both rendered as plain text radio pills too, but "Test
  product" has **no native Shopify variant colors/images set** on its
  "Type" option, so there was nothing for "automatically assigns the
  existing variant colors/images" (the live-confirmed tooltip text for
  these 2 options) to actually pull from — a plain-text fallback here is
  the *expected* result for an empty auto-source, not necessarily the
  same bug. Reverted "Type" back to "Custom color dropdown" (its original
  setting) afterward. `Need confirm`: retest on a product with genuine
  Shopify-native variant colors/images configured.
- **Resolved (live-tested 2026-09-09, reconfirmed via the Wholesale
  widget test on "Clay Plant Pot"): the bug also affects the app's own
  offer-widget-internal variant rendering, not just the theme's native
  picker.** Clay Plant Pot's Wholesale widget renders its "Color" values
  (Brown, Black — both configured with real images under "Custom image
  dropdown") as plain clickable text labels, not swatches (see
  [05-wholesale-pricing-offer.md](05-wholesale-pricing-offer.md)
  Storefront section) — same failure pattern as the native picker cases
  above, just inside the app's own widget instead of the theme's.

## Cross-Feature Interaction — Quantity Break's "Variant display" settings (QVD-28)

[Quantity Break](03-quantity-break.md) has its own per-offer "Display
settings" card with a **"Variant display"** section (see
`03-quantity-break.md` Business Rules, "Display settings (QVD-28)").

**Resolved (live-tested 2026-09-09, Offer 15/ID 815, Copper Light):**
turned on **"Allow customers to choose different variants for each
item"** live and it did **NOT** reveal any "Variant swatch settings"
button, nor the "Allow customer to choose variant for single-item offer"
checkbox QVD-28 also describes as conditional on this one — **neither
element exists**, confirming the "on hold" status noted in QVD-28's own
2025-08-01 comment thread extends to the button too, not just the
single-item checkbox. Only the 2 checkboxes already documented in
`03-quantity-break.md` ("Allow customers to choose different variants
for each item" and "Hide theme variant picker") exist in this section —
nothing else appears when the first is checked.

Instead, turning the checkbox on made the offer's own **Preview section**
render a **per-pack-item variant picker directly inline** — each item
slot within a multi-quantity tier (e.g. "2 packs" → items "#1" and "#2")
got its own pair of native `<select>` dropdowns (a color-named option and
a size-named option, using generic mock values like "Sunset/Ice/Dawn" and
"S/M/L" — clearly placeholder/demo data unrelated to this offer's real
trigger product, Copper Light, which has no such options). This confirms
QVD-28's first user story (per-item variant picker within a tier) is
implemented as **inline dropdowns rendered directly in the tier UI**, not
via any link/button out to this feature's swatch-config page — **the
"Variant swatch settings" button documented in QVD-28 was never built,
or was removed.** This session's change was discarded (not saved) to
avoid altering the real offer's config.

**Resolved (live-tested 2026-09-11, Offer 16/816 Multibuy and Offer 17/818
Wholesale): neither has this checkbox, or any "Display settings"/"Variant
display" section at all.** Searched both offers' full settings pages for
"Display settings" and "Allow customers to choose different variants for
each item" — zero matches on either. This section (and its 2 checkboxes)
is **Quantity-Break-specific**, not a shared component across the 3 offer
types — matches the broader pattern already seen elsewhere (each offer
type has its own independent Appearance/Color/Typography field sets too).

**Still open:** whether the inline per-item dropdowns Quantity Break does
show (once a real multi-variant product is used) actually apply this
feature's swatch config (color/image) or — consistent with the
storefront rendering bug found above — also just fall back to plain
dropdowns regardless of configured swatch type. Not tested (Copper Light,
this offer's real trigger product, has no variants to test with).

## Edge Cases

_Some resolved via this session's live Portal check (2026-09-09); others
still drafted from ticket specs only._

- **Resolved (live 2026-09-09):** variant values are matched by exact
  string, case-sensitive — "brown" and "Brown" are distinct rows with
  independently configurable images/colors, not merged. See Business
  Rules above.
- What happens to an offer's storefront widget if the merchant changes a
  variant option's swatch type **after** the offer is live — does it
  update immediately, or does it need a resync/republish? Not tested.
- Behavior when the same variant option name is used differently across
  products (e.g. "Color" meaning fabric color on one product, paint color
  on another) — is swatch config truly per option-name globally, with no
  per-product override? Not tested, but the Portal's data model (one row
  per option *name*, "Connected products" listing all products sharing
  it) strongly implies yes — a single "Color" config would apply to every
  product with a "Color" option, unrelated products included.
- A variant value with **no** configured color/image — **partially
  resolved (live 2026-09-09):** renders as a plain text radio pill, same
  as a *configured* value does (see the major bug above) — there's no
  visible difference between "configured but broken" and "never
  configured," since neither ever shows an actual swatch on this theme.
- **Resolved (live 2026-09-09) — the swatch config does not visually
  render on the storefront at all** (color swatches show as plain text
  radio pills; the configured HEX/image data leaks as raw unrendered text
  below Add to cart instead). See Storefront rendering above and
  [known-issues.md](../known-issues/known-issues.md). Whether this also
  affects "Custom image dropdown" / "Variant color dropdown" / "Variant
  image dropdown", or is specific to "Custom color dropdown", is still
  open.
- What "Variant color dropdown" (the 5th Swatch type option, not in any
  ticket found) actually does — not tested.
- Interaction with the QVD-28 "Hide theme variant picker" checkbox — if
  the theme's native picker is hidden and this app's own swatch widget is
  configured, does the display-type choice from this page still apply to
  the app's widget rendering? Somewhat moot given the storefront
  rendering bug above (nothing currently visually applies), but worth
  re-checking once/if that bug is fixed.
- **Resolved (live 2026-09-09):** the "Variant swatch settings" button
  QVD-28 describes (conditional on "Allow customers to choose different
  variants for each item") **does not exist** — confirmed absent when
  that checkbox was turned on live for a real Quantity Break offer. See
  Cross-Feature Interaction above.
- Whether "Allow customer to choose variant for single-item offer"
  (QVD-28's 2nd checkbox) — flagged in QVD-28's own comment thread
  (2025-08-01) as put **on hold** pending other dependency work —
  **resolved (live 2026-09-09): still not shipped** — confirmed absent
  from the UI alongside the "Variant swatch settings" button, consistent
  with never having come off hold.
- [QVD-134](https://ownego.atlassian.net/browse/QVD-134) "[B4S update]
  Variant swatch error" — a Shopify app-review report that saving in the
  "Variant Editor" produced an unexpected error during their review.
  QA's own comment on the ticket: **"Cannot reproduce." Retested live
  2026-09-11 — still cannot reproduce:** entered a real HEX value
  (`#3366FF`) into option "Type"'s "Custom color dropdown" BK1 field and
  saved — save succeeded with no error, and the value persisted after a
  full page reload. Reverted the value back to empty afterward. Not
  added to [known-issues.md](../known-issues/known-issues.md) — 2 QA
  attempts (2026-02-25 and 2026-09-11) have now failed to reproduce it,
  though this was only 1 save path (a single color value on an existing
  option); the original report was specifically about Shopify's own
  reviewers using "the Variant Editor" during their app review, which may
  differ from this flow.

## Known Issues

See [known-issues.md](../known-issues/known-issues.md) for entries tagged `variant-swatches`.

## Related Jira Tickets

- [QVD-13](https://ownego.atlassian.net/browse/QVD-13) — Variant swatch, Epic — status **Passed**
- [QVD-46](https://ownego.atlassian.net/browse/QVD-46) — [Portal] Variant swatch settings, Story — described a modal; shipped as a dedicated page instead (corrected 2026-09-09)
- [QVD-49](https://ownego.atlassian.net/browse/QVD-49) — [SDK] Variant swatch settings, Story — storefront rendering of the swatch types; lists only 3, Portal live has 5 — not yet reconciled
- [QVD-28](https://ownego.atlassian.net/browse/QVD-28) — [Portal] Variant eligibility settings, Sub-task — lives in [Quantity Break](03-quantity-break.md)'s own Display settings; its "Variant swatch settings" button is the cross-feature link to this page (see Cross-Feature Interaction above)
- [QVD-134](https://ownego.atlassian.net/browse/QVD-134) — [B4S update] Variant swatch error, Sub-task — Shopify review-reported save error, QA could not reproduce

## Source

- Jira (QVD project), via Atlassian MCP, analyzed 2026-09-09.
- Live Playwright MCP test, `test-vl-1`, 2026-09-09 (Portal + storefront:
  product "Color swatch" for the swatch-rendering bug, "Test product" for
  the unconfigured-Type comparison, Offer 15/815 for the QVD-28
  cross-feature check).
- Live Playwright MCP test, `test-vl-1`, 2026-09-11 (extended coverage:
  "Variant color dropdown"/"Variant image dropdown" tested on "Type" +
  "Test product" — inconclusive, no native Shopify color/image data
  existed to auto-assign; "Custom image dropdown" confirmed broken on
  "White Ceramic Pot"; Multibuy Offer 16/816 and Wholesale Offer 17/818
  confirmed to have no "Display settings" section at all; QVD-134 retest
  via a real save on "Type"'s Custom color dropdown — no error, still
  cannot reproduce).
- Help center (may be outdated): https://help.qikify.com/en/article/variant-swatches-1b7x5dy/
