# Decision Log

**Status:** skeleton — append-only

Why is a rule/behavior the way it is? One entry per decision, dated, with
the reasoning. Never delete or rewrite past entries — add a new one if a
decision changes, and note it supersedes the old one.

## Format

```
## YYYY-MM-DD — <short title>

**Decision:** what was decided
**Why:** the reasoning / constraint that drove it
**Source:** ticket ID, Slack thread, or who decided
```

## Log

## 2026-08-27 — Removed "Product Recommendations (Upselling)" from features

**Decision:** removed `features/product-recommendations-upselling.md` and its
references in README/feature-map. Not tracked as a QVL feature.
**Why:** the initial feature skeleton was pulled from the public help center
(help.qikify.com/en/category/qikify-volume-discounts-jy3mk3/), which listed
"Product Recommendations (Upselling)" as an article under that category. QA
confirmed directly that this feature does not exist in the app — the help
center entry is stale, mislabeled, or for a different app. Treat this as
confirmation that the help center is not reliable ground truth; every
skeleton feature should be verified the same way before being trusted.
**Source:** QA, direct confirmation in conversation.

## 2026-08-27 — Merged Buy X Get Y, Quantity Breaks for Same Product Offer, and Volume Discounts for Subscription Products into "Quantity Break"

**Decision:** merged `features/buy-x-get-y.md`, `features/quantity-breaks-same-product.md`,
and `features/volume-discounts-subscription-products.md` into a single
`features/quantity-break.md`, with each former article kept as a "Variant"
subsection (same-product tiers, Buy X Get Y, subscription products).
**Why:** QA confirmed these 3 help-center articles are not independent
features — they're the same underlying quantity-threshold mechanic applied
in 3 different contexts. Keeping them as 3 files would mean documenting
shared business rules 3 times and risking drift between copies. Named the
umbrella "Quantity Break" rather than "Volume Discounts" to avoid ambiguity
with the app/project name ("Volume").
**Source:** QA, direct confirmation in conversation.
