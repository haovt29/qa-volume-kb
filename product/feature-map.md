# Feature Map — interactions between features

**Status:** skeleton

Discount features commonly stack or conflict with each other. This file is
for documenting *cross-feature* rules that don't belong in any single
feature's file — e.g. "what happens if a customer qualifies for both
Wholesale Pricing and the Buy X Get Y variant of Quantity Break at once?"

Update this whenever a change to one feature affects another. Note that
`quantity-break.md` itself has 3 variants (same-product tiers, Buy X Get Y,
subscription products) that can also interact with *each other* — track
those inside `quantity-break.md`, reserve this file for interactions
*across* separate feature files.

## Features in scope

(Custom CSS is not listed here — it's a shared setting, not an offer type. See [product/custom-css.md](custom-css.md).)

- [Multibuy – Mix & Match Combo](../features/multibuy-mix-match-combo.md)
- [Wholesale Pricing Offer](../features/wholesale-pricing-offer.md)
- [Quantity Break](../features/quantity-break.md) (variants: same-product tiers, Buy X Get Y, subscription products)
- [Variant Swatches](../features/variant-swatches.md)

## Known interactions

_TODO — for each pair of features that can be active at once, note: do they
stack, does one take priority, is combining them blocked in the UI, or is
behavior undefined/untested?_

| Feature A | Feature B | Interaction | Source |
|---|---|---|---|
| _TODO_ | _TODO_ | _TODO_ | _TODO_ |
