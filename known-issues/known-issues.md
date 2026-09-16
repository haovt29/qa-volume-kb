# Known Issues / Limitations

**Status:** skeleton

Current known bugs and intentional limitations, tagged by feature slug so
they're easy to filter. This is the single source for known issues — feature
files under `features/` link here instead of duplicating entries.

## Format

```
## <feature-slug> — short title

**Status:** open | wontfix | intentional limitation
**Impact:** who/what it affects
**Source:** ticket ID
```

## Issues

## quantity-break — Negative discount value not blocked via keyboard/paste input

**Status:** open — logged to Dev, investigated, **no fix found yet**
**Impact:** merchant can type or paste a negative number (e.g. "-10") directly
into a tier's discount value field and it is **not** rejected — validation
only blocks negative values when using the field's arrow-key
increment/decrement controls, not direct text entry. Could let a merchant
accidentally save a tier with a negative discount value; effect on actual
storefront pricing not yet verified. QA filed this with Dev; Dev looked
into it and reported they couldn't find a fix — not a wontfix decision,
just currently stuck/unresolved. Worth periodically retesting rather than
treating as permanently accepted.
**Source:** QA, direct confirmation in conversation (2026-09-03); Dev
investigation outcome per QA, 2026-09-03. See
[03-quantity-break.md](../features/03-quantity-break.md) Edge Cases.

## quantity-break, wholesale-pricing-offer, variant-swatches — Variant Swatches config data leaks as raw unrendered text instead of rendering as a swatch

**Status:** open
**Impact:** below the Add to cart/Buy it now buttons, the storefront renders a
literal, unformatted text string dumping a JS-object-like map of the
product's variant option values to their configured [Variant Swatches](../features/06-variant-swatches.md)
value (color HEX or image URL) — e.g. `{ [Default Title]: "", }` on a
single-variant product with an unconfigured option (Quantity Break,
product "Copper Light"), `{ [Regular]: "", }, { [Large]: "", }` on
Wholesale's "Clay Plant Pot" (Size option, unconfigured — "Default
dropdown"), and, most tellingly, `#eaabab { [Beige]: "#eaabab", } , {
[Pink]: "#FFC0CB", } , { [stock = 2]: "#000000", } , { [Green]:
"#05AA3D", }` on product "Color swatch" (no app offer at all — plain
theme product page) whose "Color" option **is** configured with real HEX
values via Variant Swatches ("Custom color dropdown").
**Root cause, corrected/upgraded 2026-09-09 (was previously described as
just a harmless leftover debug artifact) — this is the swatch feature
itself failing to render, not a cosmetic leak:** on that same "Color
swatch" product, live-confirmed the native variant picker **still shows
plain text radio pills ("Beige", "Pink", "stock = 2", "Green") — no
color swatches at all**, identical to how an unconfigured option would
render. The configured HEX values are being computed/loaded correctly
(proven by them leaking verbatim in the debug text) but never actually
turned into a visual swatch — so [Variant Swatches](../features/06-variant-swatches.md)'s
color/image customization **does not visibly work on the storefront**
for the native theme variant picker, at least on this Dawn theme setup.
**Confirmed across offer types and even with zero app offer present** —
this is a shared SDK/theme-extension-layer bug in how the variant
options data is passed to the picker template, not isolated to one offer
type or tied to any offer widget at all.
**Further confirmed 2026-09-11, ruling out "color-only" as the cause:**
product "White Ceramic Pot" (`test-vl-1`, no app offer) has its "Color"
option set to **"Custom image dropdown"** with real images already
uploaded for values "White" and "Green" — storefront still rendered
plain text radio pills, not image swatches. **Same bug reproduces with
both of the 2 "Custom ..." swatch types tested so far** (color and
image); the 2 "Variant ..." (auto-source) types were also tested but
inconclusively (no native Shopify color/image data existed on the test
product to auto-assign in the first place — see
[06-variant-swatches.md](../features/06-variant-swatches.md) for detail).
**Also confirmed to affect the app's own offer-widget rendering, not
just the theme's native picker:** Wholesale's widget on "Clay Plant Pot"
renders its "Color" values (Brown, Black — both with real configured
images) as plain clickable text too, not swatches. Not yet reported to
Dev.
**Source:** QA, direct observation during live Playwright testing,
`test-vl-1` (2026-09-03 for Quantity Break, 2026-09-07 for Wholesale,
2026-09-09 for the Variant Swatches root-cause confirmation on product
"Color swatch", 2026-09-11 for the "Custom image dropdown" / widget-
internal-rendering confirmation on "White Ceramic Pot"/"Clay Plant
Pot"). See [03-quantity-break.md](../features/03-quantity-break.md),
[05-wholesale-pricing-offer.md](../features/05-wholesale-pricing-offer.md),
and [06-variant-swatches.md](../features/06-variant-swatches.md).
