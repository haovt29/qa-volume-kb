# 8. Analytics

**Status:** skeleton

Added by QA (2026-08-27) — confirmed to exist as a feature, not yet described in detail.

**Cross-reference:** [QVD-115](https://ownego.atlassian.net/browse/QVD-115)
adds an add-to-cart (ATC) click count (last 30 days) directly onto the
[Onboarding Dashboard](01-onboarding-dashboard.md) — a metric hosted there,
not necessarily part of a broader Analytics section/tab. Confirm during
analysis of this feature whether Analytics is a separate area with more
metrics, or whether the dashboard stat *is* the whole of "Analytics" so far.

**Second cross-reference (found analyzing AI Offer Creation, 2026-08-27):**
[QVD-173](https://ownego.atlassian.net/browse/QVD-173) builds a backend
event/metrics pipeline (star distribution, response rate, copy rate, etc.)
for the [Ask for Review](09-ask-for-review.md) flow specifically — this is
an **AI-review funnel analytics** pipeline, narrower than and separate from
whatever this "Analytics" feature turns out to cover. Don't assume they
share infrastructure without confirming — treat as two candidate
data sources until this feature is analyzed.

**Third cross-reference:** [QVD-179](https://ownego.atlassian.net/browse/QVD-179)
(Sidekick app extension, In Progress) surfaces offer performance data
(views, ATC, conversion, revenue) inside Shopify's own Sidekick assistant —
see [AI Offer Creation](02-ai-offer-creation.md). Worth checking whether
this reuses the same performance data this Analytics feature is built on,
once both are analyzed.

## Purpose

_TODO: what does this report on — offer performance, revenue impact,
conversion by offer type? Where does it live in the app (own tab, embedded
in dashboard)?_

## Business Rules

_TODO: fill in via /analysis against related Jira tickets._

## Edge Cases

_TODO_

## Known Issues

See [known-issues.md](../known-issues/known-issues.md) for entries tagged `analytics`.

## Related Jira Tickets

_TODO_

## Source

- QA, direct confirmation in conversation (2026-08-27).
