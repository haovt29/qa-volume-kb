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
