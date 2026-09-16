# Feature Map — interactions between features

**Status:** skeleton

Discount features commonly stack or conflict with each other. This file is
for documenting *cross-feature* rules that don't belong in any single
feature's file — e.g. "what happens if a customer qualifies for both
Wholesale Pricing and the Buy X Get Y variant of Quantity Break at once?"

Update this whenever a change to one feature affects another. Note that
`03-quantity-break.md` itself has 3 template variants ("Quantity breaks for
same product", "Buy X, get Y free", "Volume
discount for subscription" — live-confirmed exact card names, 2026-09-03) that can also
interact with *each other* — track those inside `03-quantity-break.md`,
reserve this file for interactions *across* separate feature files.

## Features in scope

(Custom CSS is not listed here — it's a shared setting, not an offer type. See [product/custom-css.md](custom-css.md).)

Offer types (can stack/conflict with each other at cart/checkout — the
interactions this file is mainly for):
- [Multibuy – Mix & Match Combo](../features/04-multibuy-mix-match-combo.md)
- [Wholesale Pricing Offer](../features/05-wholesale-pricing-offer.md)
- [Quantity Break](../features/03-quantity-break.md) (templates: "Quantity breaks for same product", "Buy X, get Y free", "Volume discount for subscription")

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
| Quantity Break | Multibuy | **Trigger overlap is detected and warned, but not blocked. Confirmed: overlapping offers do NOT both apply — only one discount wins per cart, and which one wins is not yet understood.** Creating/editing an offer whose trigger includes a product already covered by another offer's trigger shows a non-blocking warning: *"This trigger overlaps with offer(s): &lt;name&gt;. Overlapping triggers may cause discount conflicts."* **Confirmed live 2026-09-03** (store `test-vl-1`, offers "Offer 15" [Quantity Break] and "Offer 816" [Multibuy], both triggered on product "Copper Light"): both widgets render simultaneously on that product's page. Two cart tests, both from real add-to-cart flows: **Test 1** — cart already had 3× Copper Light carrying Quantity Break's 20% discount, then added 1× Pink Armchair via Multibuy's "Pick" (4 total eligible items, meeting Multibuy's "4 for $30" tier) → **Multibuy's discount did not apply at all**; Quantity Break's discount kept winning, Pink Armchair stayed full price, cart total $219 (not $30). **Test 2** — clean cart, added 1× Pink Armchair via Multibuy's "Pick" first, then 2× Copper Light via the product page's own Add-to-cart (3 total eligible items, meeting Multibuy's "3 for $20" tier) → **this time Multibuy's discount won instead**: both lines discounted proportionally to their original price toward the $20 tier total (Copper Light 2×$6.16, Pink Armchair 1×$7.69 — $12.31 + $7.69 = $20.00 exactly, confirming Multibuy's Fixed-price tiers **do** allocate proportionally by original price across different SKUs when the combo is met exactly, not just when there's a "highest-priced-first" remainder), and **Quantity Break's discount did not apply at all** this time, even though 2× Copper Light alone should qualify for its own Tier 2. **Conclusion: exactly one of the two overlapping offers' discounts applies per cart — never both, never split between them — but which one wins flipped between the two tests** (Quantity Break won in Test 1, Multibuy won in Test 2) with no variable identified yet that explains why. `Need confirm PO/Dev`: what determines priority when 2 offers' automatic discounts both qualify for the same cart (discount rank/order in Shopify admin? which offer's Function evaluates last? something about add-to-cart order?) — needed before writing deterministic regression test cases for this scenario. | Live Playwright MCP test, `test-vl-1`, 2026-09-03 |
| Variant Swatches | Quantity Break only — **Multibuy and Wholesale confirmed 2026-09-11 to NOT have this section at all** | **Resolved 2026-09-09/11: no shared modal/link exists at all — QVD-28's "Variant swatch settings" button was never built (or was removed), and the whole "Display settings" section is Quantity-Break-specific.** Quantity Break's own per-offer "Display settings → Variant display" section has an "Allow customers to choose different variants for each item" checkbox; live-tested turning it on for a real offer (Offer 15/815, Copper Light) and **no "Variant swatch settings" button appeared, nor did QVD-28's 2nd checkbox ("...for single-item offer")** — only the 2 checkboxes already documented in `03-quantity-break.md` exist. Instead, the checkbox makes the offer's Preview render **inline per-item `<select>` dropdowns** directly in the tier UI (one dropdown pair per item slot within a multi-quantity tier) — generic mock data, not obviously wired to [Variant Swatches](../features/06-variant-swatches.md)'s own swatch config at all. **Live-tested 2026-09-11 on Multibuy Offer 16/816 and Wholesale Offer 17/818: neither has a "Display settings" section, the checkbox, or any equivalent** — confirmed this whole mechanism is unique to Quantity Break, not shared. Separately, live-testing (2026-09-09, extended 2026-09-11) found Variant Swatches itself has a **major storefront bug, confirmed across 2 of its 5 swatch types with real configured data ("Custom color dropdown" and "Custom image dropdown") and inside the app's own offer widgets too (Wholesale on "Clay Plant Pot")**: configured color/image swatches don't actually render, falling back to plain text (see [known-issues.md](../known-issues/known-issues.md), tag `variant-swatches`) — so even if the inline per-item dropdowns were wired to the swatch config, the config doesn't currently do anything visible on the storefront regardless. `Need confirm`: whether the inline per-item dropdowns respect swatch config once/if the storefront rendering bug is fixed (not directly testable — Copper Light, this offer's real trigger product, has no variants). | QVD-28 — live Portal test, `test-vl-1`, 2026-09-09 and 2026-09-11 |
| Quantity Break, Multibuy | Wholesale Pricing Offer | **3-way overlap confirmed — Wholesale participates in the same "only one offer's discount applies" pattern as the Quantity Break/Multibuy row above.** **Live-tested 2026-09-09** (store `test-vl-1`, product "Copper Light"): added Copper Light to Wholesale offer "Offer 17"/818's "Specific products" trigger, alongside its existing coverage by Quantity Break's "Offer 15" and Multibuy's "Offer 16" — overlap warning correctly listed both: *"This trigger overlaps with offer(s): Offer 15, Offer 16."* All 3 widgets (Quantity Break tier cards, Multibuy combo picker, Wholesale TIER PRICING table) rendered simultaneously on the PDP. Test: set quantity = 3 via **Wholesale's own** quantity stepper and clicked **Wholesale's own** "Add to cart" button → the cart line got **Quantity Break's** discount instead ($48.00/item, "Buy 3 save 20% off" tier, discount code `QIKIFY-5WBE1R`) — not Wholesale's own Tier 1 rate. **Caveat — not a fully clean 3-way priority test:** at qty=3, Wholesale's own Tier 1 (1-10 items) is 0% off, so there was no genuine conflict between two *active* discounts, just Quantity Break's discount applying where Wholesale would've offered full price anyway. Trigger was reverted to Clay Plant Pot only afterward (offer saved back to "1 product selected"), and the test cart line was removed. `Need confirm PO/Dev`: same open question as the row above, now extended to 3 offer types — a cleaner test needs a quantity where Wholesale's own tier has a *nonzero* discount (e.g. 12-20 units) simultaneously qualifying for a Quantity Break/Multibuy tier, to see which of the two *active* discounts actually wins. | Live Playwright MCP test, `test-vl-1`, 2026-09-09 |
