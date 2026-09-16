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
  - **Usage counter (QA confirmed 2026-08-28):** panel shows live usage vs.
    limit, e.g. `AI generations: 10/50` — reflects whatever
    `ai_prompt_limit` is currently set to (default 1 in production).
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
- **Usage limits — PO-confirmed current behavior (2026-08-27), intentionally
  differs from QVD-169's written AC:**
  - **Development stores: feature fully disabled/hidden.** A message
    explains it's unavailable on dev stores. (Matches QVD-169 AC.)
  - **Production: capped at 1 use per instance, with no automatic reset**
    (not the 5-uses/day-reset-at-midnight-UTC that QVD-169's AC describes).
    Once the 1 use is spent, the merchant cannot generate again unless an
    admin raises the limit (see below).
  - **To grant more uses:** an admin adds `"ai_prompt_limit": 10` (or
    another number) to that store's **Config** in the Admin page — this
    is the actual mechanism for both raising a merchant's limit in
    production and for QA testing, not just a QA-only bypass.
  - **Limit is per store/instance, not per staff account** (this part
    matches QVD-169 AC).
  - Reason: AI generation costs tokens per request; the app is free, so
    uncapped usage isn't sustainable (QVD-169 background — this rationale
    still holds even though the concrete limit shipped differently).
  - **Resolved (2026-08-27):** PO confirmed 1/instance with no reset is the
    intended current behavior — QVD-169's AC (5/day, UTC-midnight reset)
    is outdated/superseded, not a bug. Write test cases against 1/instance,
    no reset, `ai_prompt_limit` Config override — not against the ticket's AC.
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
  data). **On hold (per Dev, 2026-08-28):** still mid-coding, flow not
  stable yet — don't analyze/test this sub-part further until Dev signals
  it's ready; revisit the "own feature file vs sub-part of AI Offer
  Creation" question at that point.
- **Not yet built — "AI entry creation" (QVD-165, Epic, status: To Do):** no
  description on the ticket; status shows this hasn't started. Don't treat
  anything about additional AI "entry points" as current behavior until
  this ships and gets its own analysis pass.

## Edge Cases

- Prompt is empty, gibberish, or doesn't map to any of the 5 offer types —
  **Resolved (QA confirmed 2026-08-28):** a warning banner shows for
  gibberish/unmappable prompts; for empty prompt or prompt **< 5
  characters**, the "Generate offer" button is disabled instead (can't
  even submit).
- Prompt is ambiguous between two offer types (e.g. could read as either
  Quantity Break or Wholesale Pricing) — **Resolved (QA confirmed
  2026-08-28):** no clarification step; the AI picks based on the prompt
  and an internal "closest-matching offer type" priority rule (no further
  detail on the rule's exact logic — treat as a black box, test by
  observed output on ambiguous prompts).
- Prompt near/over the 2,000-character limit — **Resolved (QA confirmed
  2026-08-28):** hard cutoff at 2,000 characters (can't type past it).
- Prompt requests something outside the 5 supported offer types (e.g. a
  feature that doesn't exist) — **Resolved (QA confirmed 2026-08-28):** a
  warning banner shows (same mechanism as the unmappable-prompt case
  above), not a hallucinated config.
- Merchant edits the AI-generated config before saving — **Resolved (QA
  confirmed 2026-08-28):** yes, `isAiGenerated` (QVD-171) still persists —
  it's saved into the offer data itself, not derived from the edit path —
  so the review prompt still fires. Still worth a dedicated regression
  test since it's easy to break (e.g. if a future refactor overwrites the
  flag on edit).
- Dev store merchant somehow reaches the Generate flow via a stale
  URL/direct link — **Resolved (QA confirmed 2026-08-28):** UI is hidden
  and a banner explains the feature is only available on real (non-dev)
  stores. **Note:** this confirms the *UI-level* behavior only — QA has
  not separately verified server-side enforcement (i.e. whether the
  underlying generate request would also be rejected if called directly,
  bypassing the UI). Worth a dedicated check if/when API-level testing
  becomes possible.
- Merchant hits the 1-use limit mid-session, e.g. after starting to edit a
  previously-generated offer — **Resolved (QA confirmed 2026-08-28):**
  yes, editing/regenerating a previously-generated offer also counts
  against the limit (not just brand-new generations).
- Merchant reaches the limit and has no way to get more without an admin
  manually raising `ai_prompt_limit` in Config — **Resolved (QA confirmed
  2026-08-28):** UI shows a "limit reached" message and disables the
  prompt input field — doesn't imply a reset is coming.
- Admin raises `ai_prompt_limit` to N — **Resolved (QA confirmed
  2026-08-28):** the UI shows a live counter reflecting the new limit,
  e.g. `AI generations: 10/50` when the limit is raised to 50 and the
  merchant has used 10.
- QA needs more than 1 generation for testing — **see Testability Notes**,
  raise `ai_prompt_limit` in Config, don't file running out as a blocker bug.

## Testability Notes

Per [`qikify-volume.md` QA app note](../dev-references/references.md): no
API access for QA automation — verify via UI (Playwright MCP) unless noted
otherwise below.

- **Raising the limit for testing (QA-confirmed 2026-08-27, also
  documented in a QVD-169 dev comment):** in the Admin page for the
  relevant user/store, add `"ai_prompt_limit": 10` (or another number) to
  **Config** to raise the allowed prompt count above the default of 1.
  Since there's no daily reset, this is the *only* way to test
  at-limit/over-limit and multi-generation scenarios — there's no waiting
  it out. The QVD-169 comment also mentions an Admin **"Test"** toggle;
  confirm whether that's still needed alongside `ai_prompt_limit` or was
  superseded by it. **This is admin/config-level access, not pure UI** —
  note that in automation-constraint terms even though it's not the app's
  own API.
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
- [QVD-169](https://ownego.atlassian.net/browse/QVD-169) — AI Offer Usage Limit, Story — dev-store block; AC describes 5/day+UTC-reset, but this is **outdated — PO-confirmed current behavior is 1/instance, no reset** (see Business Rules); `ai_prompt_limit` Config override documented in a comment
- [QVD-171](https://ownego.atlassian.net/browse/QVD-171) — Ask for review for AI-created offer, Story — cross-ref, see [09-ask-for-review.md](09-ask-for-review.md)
- [QVD-173](https://ownego.atlassian.net/browse/QVD-173) — [BE] Update AI offer analytics, Task — backend for QVD-171's review funnel, cross-ref
- [QVD-179](https://ownego.atlassian.net/browse/QVD-179) — Sidekick app extension — UI for AI Data-Response Cards, Story — **In Progress**, related but distinct AI surface

## Source

- Jira (QVD project), via Atlassian MCP, analyzed 2026-08-27.
- QA, direct confirmation in conversation (2026-08-27) — feature exists.
