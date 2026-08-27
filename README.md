# QVL QA Knowledge Base

QA-oriented knowledge base for **Qikify Volume Discounts** (Testlane project key: `QVL`).
This repo is the source of truth. It is *not* the merchant-facing help center —
it exists to capture business rules, edge cases, and known issues at the level
of detail QA/test-case writing actually needs.

## Status legend

Each file starts as a **skeleton** pulled from the public help center (which may
be outdated) and gets filled in via `/analysis` against real Jira tickets /
current app behavior. Status is tracked at the top of each file:

- `skeleton` — topic list only, from help.qikify.com, not yet verified
- `in-review` — being analyzed against tickets/current behavior
- `verified` — confirmed against current app behavior, safe to use for test-case generation

## Structure

```
features/            one file per feature/module
  <feature-slug>.md
```

Each feature file has:
- **Purpose** — what the feature does, in one paragraph
- **Business Rules** — the rules the app is supposed to enforce
- **Edge Cases** — boundary/unusual conditions worth testing
- **Known Issues / Limitations** — anything already known to be broken or intentionally limited
- **Related Jira Tickets** — tickets that informed or changed this feature
- **Source** — where the info came from (help center article, ticket, direct testing)

## How this KB is used

- **Claude Code**: reads these files directly from the repo (local filesystem).
- **Testlane**: the same `.md` files get uploaded into the QVL project's
  "project context" so the `/generate-tc` skill can use them automatically
  when generating test cases. Re-upload after any meaningful edit — Testlane
  keeps a copy, it does not sync live from git.
- **ChatGPT**: no direct git access — attach the relevant `.md` file(s) to a
  Project/conversation, or paste content in. Also a manual copy, keep it
  in sync by hand.

## Workflow to fill in a feature

1. Pick a feature file below (currently all `skeleton`).
2. Pull related Jira tickets / changelog entries for that feature.
3. Run the `/analysis` skill against those tickets to extract current
   business rules, edge cases, and gaps.
4. Update the feature file, bump its status.
5. Re-upload changed files into Testlane's QVL project context.

## Features (from help.qikify.com/en/category/qikify-volume-discounts-jy3mk3/)

| File | Topic | Status |
|---|---|---|
| [custom-css.md](features/custom-css.md) | Custom CSS | skeleton |
| [multibuy-mix-match-combo.md](features/multibuy-mix-match-combo.md) | Multibuy – Mix & Match Combo | skeleton |
| [wholesale-pricing-offer.md](features/wholesale-pricing-offer.md) | Wholesale Pricing Offer | skeleton |
| [quantity-breaks-same-product.md](features/quantity-breaks-same-product.md) | Quantity Breaks for Same Product Offer | skeleton |
| [product-recommendations-upselling.md](features/product-recommendations-upselling.md) | Product Recommendations (Upselling) | skeleton |
| [volume-discounts-subscription-products.md](features/volume-discounts-subscription-products.md) | Volume Discounts for Subscription Products | skeleton |
| [translate-offers.md](features/translate-offers.md) | How to Translate Offers | skeleton |
| [buy-x-get-y.md](features/buy-x-get-y.md) | Buy X Get Y | skeleton |
| [variant-swatches.md](features/variant-swatches.md) | Variant Swatches | skeleton |

Note: the help center is known to lag behind the current app, so treat the
list above as a starting skeleton, not ground truth — features may have been
added, changed, or removed since those articles were last updated.
