# 2. AI Offer Creation

**Status:** in-review

Merchant-facing tool on the offer creation page: lets a merchant describe a
promotion in natural language and get a ready-to-launch offer config
auto-generated, instead of manually picking an offer type and filling every
field. Analyzed against Jira 2026-08-27 (QVD-161, QVD-165, QVD-169; related:
QVD-171, QVD-173, QVD-179 — see cross-references below).

## Purpose

Speed up offer creation and remove the need for merchants to understand
offer types upfront: they type a prompt, the AI detects the correct offer
type and pre-fills the config, which the merchant can then review/edit
before saving (QVD-161).

## Business Rules

- **AI Offer Assistant panel (QVD-161)** sits at the top of the offer
  creation page, above a manual template picker:
  - Free-text prompt textarea, **max 2,000 characters**.
  - "Generate offer" CTA button.
  - Example prompts shown to help merchants start, e.g. "Buy any 3 items
    from collection A for $20", "Original price for 1, 10% off for 2 and
    20% off for 3", "Wholesale tiers: 20+ units 10% off, 50+ units 20% off,
    100+ units 30% off".
  - Character counter + hint text "We'll detect the type & fill the config".
  - Prompt can be in **any language** (per QVD-161 overview — cross-check
    against [Translate Portal](10-translate-portal.md) scope, which is
    about translating the portal UI, not prompt language handling; these
    are two different kinds of "language support").
- **Offer type grid (manual fallback, QVD-161):** below the assistant, a
  "or pick a template manually" divider, then all 5 offer types as cards:
  Quantity Breaks (same product), Buy X Get Y Free, Volume Discount for
  Subscription, Multibuy (Mix & Match Combo), Wholesale Pricing (B2B).
  These map directly to this KB's [Quantity Break](03-quantity-break.md)
  (3 variants), [Multibuy](04-multibuy-mix-match-combo.md), and
  [Wholesale Pricing](05-wholesale-pricing-offer.md) — AI Offer Creation is
  a generator *for* those offer types, not a separate offer type itself.
- **Usage limits (QVD-169):**
  - **Development stores: feature fully disabled/hidden.** A message
    explains it's unavailable on dev stores.
  - **Production stores: capped at 5 uses/day**, reset at **midnight UTC**.
    Once reached, "Generate offer" is disabled with a message that the
    daily limit was hit.
  - Remaining uses shown in the UI, e.g. "Daily usage limit: 3/5".
  - **Limit is per store, not per staff account.**
  - Reason: AI generation costs tokens per request; the app is free, so
    uncapped usage isn't sustainable (QVD-169 background).
- **Post-save review prompt for AI-created offers (QVD-171) — cross-feature
  link:** saving an AI-generated offer triggers the
  [Ask for Review](09-ask-for-review.md) flow, which does **not** apply to
  manually-created offers. See that file for the full spec — this is the
  single most important cross-reference for this feature, since it means
  "was this offer AI-generated" is a persisted flag (`isAiGenerated`) that
  downstream behavior depends on.
- **Related but distinct — Sidekick app extension (QVD-179, status: In
  Progress, not confirmed shipped):** a *different* AI surface — when a
  merchant asks Shopify's own **Sidekick** assistant about this app,
  Sidekick calls this app's extension and renders performance/health data
  cards back into the Sidekick panel (not into this app's UI at all). One
  of its 3 use cases (1C — planning suggestion) ends with a CTA that
  deep-links into **this** AI Offer Creation flow with pre-filled data
  ("Create this offer in Qikify" → opens the settings page with filled
  data). `Need confirm PO/Dev`: is Sidekick data-response-cards a sub-part
  of "AI Offer Creation" for KB/test purposes, or should it become its own
  feature file once shipped? Flagging rather than deciding unilaterally —
  see Edge Cases.
- **Not yet built — "AI entry creation" (QVD-165, Epic, status: To Do):** no
  description on the ticket; status shows this hasn't started. Don't treat
  anything about additional AI "entry points" as current behavior until
  this ships and gets its own analysis pass.

## Edge Cases

- Prompt is empty, gibberish, or doesn't map to any of the 5 offer types —
  what does "Generate offer" do? `Need confirm PO/Dev`.
- Prompt is ambiguous between two offer types (e.g. could read as either
  Quantity Break or Wholesale Pricing) — which wins, or does it ask for
  clarification? `Need confirm`.
- Prompt near/over the 2,000-character limit — hard cutoff vs validation
  error.
- Prompt requests something outside the 5 supported offer types (e.g. a
  feature that doesn't exist) — expected to fail gracefully, not
  hallucinate a nonexistent config. `Need confirm` how this is surfaced.
- Merchant edits the AI-generated config before saving — does the
  `isAiGenerated` flag (QVD-171) still persist, so the review prompt still
  fires? QVD-171 explicitly says yes ("persists whether the merchant
  creates the offer directly from Summary, or previews and edits it
  first") — worth a dedicated regression test since it's easy to break.
- Dev store merchant somehow reaches the Generate flow via a stale
  URL/direct link — confirm the block is enforced server-side, not just
  UI-hidden.
- Merchant hits the 5/day production limit mid-session, e.g. after
  starting to edit a previously-generated offer — does the *editing* count
  against the limit, or only new generations?
- Limit resets at UTC midnight — store in a very different timezone
  (e.g. UTC+12) sees the reset at a locally odd time — confirm this is
  intentional (per QVD-169 AC, it is: "reset at midnight UTC") rather than
  a localization gap, but worth a UI-copy check that it doesn't claim
  "resets at midnight" without the UTC qualifier.
- QA needs to exceed the 5/day limit for testing — **see Testability Notes,
  there's a documented bypass**, don't file this as a blocker bug.

## Testability Notes

Per [`qikify-volume.md` QA app note](../dev-references/references.md): no
API access for QA automation — verify via UI (Playwright MCP) unless noted
otherwise below.

- **QA bypass for the 5/day limit (from QVD-169, dev comment):** in the
  Admin page for the relevant user, enable **"Test"** to allow prompting
  regardless of the limit, and/or add `ai_prompt_limit: xxx` to that user's
  **Config** to raise the allowed prompt count. Use this to test both
  under-limit and at-limit/over-limit behavior without waiting for a daily
  reset. **This is admin/config-level access, not pure UI** — note that in
  automation-constraint terms even though it's not the app's own API.
- Dev-store blocking: needs an actual Shopify development store to verify
  (not just a flag) — confirm test store type before relying on this check.
- `isAiGenerated` persistence through preview→edit→save (QVD-171 requirement,
  also called out as fragile in QVD-173's "FE contract" — losing
  `ai_offer_event_id` through that flow "means the offer loses its AI flag,
  the overlay never shows, and view rate drops"): verifiable via UI, but
  the *symptom* of a break here is silent (no review prompt shows) — needs
  an explicit test case, not just incidental observation.
- Sidekick data cards (QVD-179): **not verifiable via this app's own UI at
  all** — the rendering surface is inside Shopify Sidekick, a
  Shopify-owned assistant, so this requires either genuine Sidekick access
  in a test store or Dev/Support-provided screenshots/logs. Mark as
  constrained testability once this ships.

## Known Issues

See [known-issues.md](../known-issues/known-issues.md) for entries tagged `ai-offer-creation`.

## Related Jira Tickets

- [QVD-161](https://ownego.atlassian.net/browse/QVD-161) — AI Offer Assistant – Prompt-to-Config Feature, Story — core spec
- [QVD-165](https://ownego.atlassian.net/browse/QVD-165) — AI entry creation, Epic — **To Do, not started**, no description
- [QVD-169](https://ownego.atlassian.net/browse/QVD-169) — AI Offer Usage Limit, Story — dev-store block + 5/day production cap; QA bypass documented in a comment
- [QVD-171](https://ownego.atlassian.net/browse/QVD-171) — Ask for review for AI-created offer, Story — cross-ref, see [09-ask-for-review.md](09-ask-for-review.md)
- [QVD-173](https://ownego.atlassian.net/browse/QVD-173) — [BE] Update AI offer analytics, Task — backend for QVD-171's review funnel, cross-ref
- [QVD-179](https://ownego.atlassian.net/browse/QVD-179) — Sidekick app extension — UI for AI Data-Response Cards, Story — **In Progress**, related but distinct AI surface

## Source

- Jira (QVD project), via Atlassian MCP, analyzed 2026-08-27.
- QA, direct confirmation in conversation (2026-08-27) — feature exists.
