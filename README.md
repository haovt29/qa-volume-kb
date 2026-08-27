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
README.md             human-facing overview (this file)
AGENTS.md              instructions for AI agents editing this repo
CLAUDE.md              -> points to AGENTS.md
product/
  overview.md          what the app does, at a glance
  glossary.md           terms used inconsistently elsewhere
  feature-map.md         cross-feature interactions (stacking, conflicts)
  custom-css.md           shared setting used across offer types, not an offer type itself
features/
  <feature-slug>.md      one file per feature/module
testing/
  regression-checklist.md  cross-feature "always check this" list
  test-data.md              reusable test data / store setup
  environments.md            staging URLs, accounts, access
decisions/
  decision-log.md         append-only "why is it this way"
known-issues/
  known-issues.md          current bugs/limitations, tagged by feature
dev-references/
  references.md            links: Jira, repos, Slack
```

Each feature file has:
- **Purpose** — what the feature does, in one paragraph
- **Business Rules** — the rules the app is supposed to enforce
- **Edge Cases** — boundary/unusual conditions worth testing
- **Known Issues** — link into `known-issues/known-issues.md` (not duplicated locally, to avoid drift)
- **Related Jira Tickets** — tickets that informed or changed this feature
- **Source** — where the info came from (help center article, ticket, direct testing)

Full editing conventions for agents are in [AGENTS.md](AGENTS.md).

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
| [multibuy-mix-match-combo.md](features/multibuy-mix-match-combo.md) | Multibuy – Mix & Match Combo | skeleton |
| [wholesale-pricing-offer.md](features/wholesale-pricing-offer.md) | Wholesale Pricing Offer | skeleton |
| [quantity-break.md](features/quantity-break.md) | Quantity Break (variants: same-product tiers, Buy X Get Y, subscription products) | skeleton |
| [translate-offers.md](features/translate-offers.md) | How to Translate Offers | skeleton |
| [variant-swatches.md](features/variant-swatches.md) | Variant Swatches | skeleton |

Note: the help center is known to lag behind the current app, so treat the
list above as a starting skeleton, not ground truth — features may have been
added, changed, or removed since those articles were last updated. Confirmed
examples of skeleton corrections so far — see [decisions/decision-log.md](decisions/decision-log.md):
- "Product Recommendations (Upselling)" was listed on the help center but does not exist in the app — removed.
- "Buy X Get Y", "Quantity Breaks for Same Product Offer", and "Volume Discounts for Subscription Products" were listed as 3 separate articles but are variants of one feature — merged into `quantity-break.md`.
- "Custom CSS" was listed as its own article, but is actually one shared input field in every offer type's advanced settings, not a standalone offer type — moved to `product/custom-css.md`.
