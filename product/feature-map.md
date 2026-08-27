# Feature Map — interactions between features

**Status:** skeleton

Discount features commonly stack or conflict with each other. This file is
for documenting *cross-feature* rules that don't belong in any single
feature's file — e.g. "what happens if a customer qualifies for both
Wholesale Pricing and the Buy X Get Y variant of Quantity Break at once?"

Update this whenever a change to one feature affects another. Note that
`03-quantity-break.md` itself has 3 variants (same-product tiers, Buy X Get Y,
subscription products) that can also interact with *each other* — track
those inside `03-quantity-break.md`, reserve this file for interactions
*across* separate feature files.

## Features in scope

(Custom CSS is not listed here — it's a shared setting, not an offer type. See [product/custom-css.md](custom-css.md).)

Offer types (can stack/conflict with each other at cart/checkout — the
interactions this file is mainly for):
- [Multibuy – Mix & Match Combo](../features/04-multibuy-mix-match-combo.md)
- [Wholesale Pricing Offer](../features/05-wholesale-pricing-offer.md)
- [Quantity Break](../features/03-quantity-break.md) (variants: same-product tiers, Buy X Get Y, subscription products)

Other features (config/reporting/onboarding — listed for completeness;
unlikely to conflict with offer types, but note here if one turns out to):
- [Variant Swatches](../features/06-variant-swatches.md)
- [Translate Offers](../features/07-translate-offers.md)
- [Translate Portal](../features/10-translate-portal.md)
- [Analytics](../features/08-analytics.md)
- [Onboarding Dashboard](../features/01-onboarding-dashboard.md)
- [Ask for Review](../features/09-ask-for-review.md)
- [AI Offer Creation](../features/02-ai-offer-creation.md) (meta: creates the offer types above — check it produces valid configs for each)

## Known interactions

_TODO — for each pair of features that can be active at once, note: do they
stack, does one take priority, is combining them blocked in the UI, or is
behavior undefined/untested?_

| Feature A | Feature B | Interaction | Source |
|---|---|---|---|
| AI Offer Creation | Quantity Break, Multibuy, Wholesale Pricing | AI generates a config for one of these 3 offer types (5 template cards map to Quantity Break's 3 variants + Multibuy + Wholesale); the generated offer must still obey each offer type's own business rules — not a separate validation path | QVD-161 |
| AI Offer Creation | Ask for Review | Saving an AI-created offer (`isAiGenerated` flag) triggers the Ask for Review overlay; manually-created offers never do | QVD-171 |
| AI Offer Creation | Onboarding Dashboard | AI Offer Creation lives on the offer creation page (reached via "Create offer" from onboarding/dashboard), not on Home itself — confirm no AI-assistant surface exists directly on Home | QVD-161, inferred |
