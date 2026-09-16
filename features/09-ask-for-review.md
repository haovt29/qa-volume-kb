# 9. Ask for Review

**Status:** in-review

**Correction (2026-08-27):** originally guessed as a customer-facing
post-purchase review request (e.g. "review this product"). That was wrong —
per QVD-171, this is a **merchant-facing** prompt asking the merchant to
rate/review the **app itself** on the Shopify App Store, shown after they
save an offer **that was created by the AI Offer Assistant** — see
[AI Offer Creation](02-ai-offer-creation.md), the flag it sets
(`isAiGenerated`) is what gates this whole feature.

## Purpose

Capture App Store reviews and low-rating feedback at a moment when the
merchant just had a (hopefully positive) AI-assisted offer-creation
experience — and route unhappy merchants to support instead of a public
1-star review.

## Business Rules

- **Trigger (QVD-171):** shows a full-screen rating overlay **after saving
  an offer, only if that offer session was created by AI** — regardless of
  whether the merchant went straight to "Create offer" from the AI Summary
  step, or previewed → edited in the Editor first. **Manually-created
  offers never see this prompt** (existing/unrelated behavior for those is
  unchanged).
- **Flagging mechanism:** any offer produced by an AI prompt session is
  marked `isAiGenerated` at the moment the session produces a draft —
  before save — and the flag must persist through preview/edit. This flag
  is the **sole gate** for whether the review prompt fires.
- **Overlay flow:**
  1. Illustration + "How was your AI offer creation experience?" + 5-star
     selector, **defaults to 5 stars**.
  2. Selecting a star immediately reveals the matching branch:
     - **5 stars:** "Generated with AI" tag; "Rewrite" button that cycles
       AI-written review copy; editable textarea; "Copy & Share on App
       Store" button — copies the text to clipboard (shows "Copied!" for
       2s), then redirects the merchant to the App Store's review modal.
     - **1–4 stars:** feedback textarea + "Send Feedback to Support".
  3. Submitting 1–4-star feedback sends it to the support team **with a
     private note containing the original AI prompt** the merchant typed
     in that session — gives support context on ask-vs-generated.
  4. Closing (× or after either branch's action) returns to the offer list.
- **Persisted fields (QVD-171):** `isAiGenerated: boolean`,
  `reviewRating: int (1–5) | null`,
  `reviewAction: enum('copied', 'feedback_sent', 'dismissed') | null`.
- **Tracked events (QVD-171/173):** `ai_review_prompt_shown`,
  `ai_review_star_selected`, `ai_review_copy_clicked`,
  `ai_review_feedback_sent`, `ai_review_dismissed` — exactly **one terminal
  event per session** (`copy_clicked` / `feedback_sent` / `dismissed`);
  firing more than one double-counts the session in backend metrics
  (QVD-173 FE contract — BE cannot enforce this, a FE bug here silently
  corrupts analytics, not just UI).
- **Backend storage (QVD-173):** events go into an existing MongoDB
  `trigger_analytics` collection via the `TriggerAnalytic` model, session
  key = `offer_id` + `ai_offer_event_id`. No new model/migration. Metrics
  computed from this: star distribution (last rating per session wins if
  changed), view rate, response rate, time-to-close, copy rate, feedback
  rate. Implemented on branch `feature/ai-offer-review-trigger-analytic`
  (per ticket, status Done — treat as shipped, but this is backend detail,
  not independently something QA verifies through the UI).

## Edge Cases

- Merchant changes their star rating after initially selecting one (e.g.
  picks 5, then changes to 2) — QVD-173 explicitly expects this ("last
  star_selected wins" for distribution) — confirm the UI actually allows
  changing the selection, not just a one-shot pick.
- Merchant clicks "Copy & Share on App Store" then later also dismisses —
  QVD-173's FE contract says this must not happen (exactly one terminal
  event) — a UI test worth having explicitly, since the bug would be
  invisible in the UI itself (only visible in backend event data).
- 5-star branch: "Rewrite" cycles AI-written copy — does repeated clicking
  cycle through a fixed pool, or generate fresh copy each time (cost
  implications, ties to the same token-cost concern as QVD-169)?
- 1–4 star feedback submitted with an empty textarea — allowed, or
  required field? `Need confirm`.
- Merchant closes the overlay (×) without picking a star at all — does
  this record as `dismissed` with `reviewRating: null`? Matches the schema
  (rating is nullable) but worth an explicit test.
- AI-generated offer created, then the merchant discards it without saving
  — overlay shouldn't show at all (trigger is "after saving"), confirm no
  false-positive.
- Offer created manually, but the merchant later edits it using the AI
  assistant (if that flow exists) — does it retroactively gain
  `isAiGenerated` and trigger this on next save? `Need confirm PO/Dev` —
  the flag is described as set "at the moment the session produces a
  draft," which suggests no, but worth confirming explicitly since it's
  the crux of this whole feature's trigger condition.

## Testability Notes

Per [`qikify-volume.md` QA app note](../dev-references/references.md): no
API access — verify via UI. Practical notes specific to this feature:
- The overlay flow itself (star selection, both branches, copy/feedback
  actions) is fully UI-testable.
- **Not verifiable from the UI alone:** whether backend events actually
  land correctly in `trigger_analytics` with the right session key, or
  whether the derived metrics (star distribution, response rate, etc.) compute
  correctly — that's backend/log verification, needs Dev/Support access or
  a documented query (QVD-173 includes example Tinker/Eloquent snippets a
  developer could re-run against a test session, if QA needs to escalate
  a suspected analytics bug).
- To reliably reach this overlay for testing, first need an AI-created
  offer — combine with [AI Offer Creation](02-ai-offer-creation.md)'s QA
  bypass (Admin "Test" toggle / `ai_prompt_limit` config) to generate one
  without burning the 5/day limit.
- **Blocked live 2026-09-14 on `test-vl-1`:** attempted to reach this
  overlay by first generating an AI offer, via
  `/apps/staging-bk-volume/offers/create`. Hit the same block already
  documented in [AI Offer Creation](02-ai-offer-creation.md): `test-vl-1`
  is a **development store**, and the AI offer assistant is **fully
  disabled** there ("Not available on development stores. The AI offer
  assistant is only available on production stores.") — confirmed via a
  fresh screenshot of the Create Offer page, template cards only, no AI
  prompt UI at all. This is a **hard UI-level gate unrelated to
  `ai_prompt_limit`** — raising that Config value doesn't help, since the
  block happens before the limit even applies. **This whole feature's
  overlay (star selection, both branches, terminal-event exclusivity,
  every edge case listed above) remains entirely untested against real
  UI** — needs either a production store, or Dev/Support access to
  directly flag an offer `isAiGenerated` for QA purposes, neither
  available in this session.

## Known Issues

See [known-issues.md](../known-issues/known-issues.md) for entries tagged `ask-for-review`.

## Related Jira Tickets

- [QVD-171](https://ownego.atlassian.net/browse/QVD-171) — Ask for review for AI-created offer, Story — full spec
- [QVD-173](https://ownego.atlassian.net/browse/QVD-173) — [BE] Update AI offer analytics, Task — backend event storage + metric definitions

## Source

- Jira (QVD project), via Atlassian MCP, analyzed 2026-08-27.
- QA, direct confirmation in conversation (2026-08-27) — feature exists (original scope guess corrected during analysis, see note at top).
- Live Playwright MCP attempt, `test-vl-1`, 2026-09-14 — blocked by the
  dev-store gate on AI Offer Creation; see Testability Notes.
