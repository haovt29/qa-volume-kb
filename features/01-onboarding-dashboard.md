# 1. Onboarding Dashboard

**Status:** in-review

One screen serving two states, confirmed by QA (2026-08-27) as a single
feature rather than two: the Home screen shows an **onboarding/empty state**
for merchants with zero offers, and switches to a **dashboard** once the
merchant has at least one offer. Analyzed against Jira 2026-08-27 (QVD-14,
QVD-18, QVD-27, QVD-31, QVD-115, QVD-135) — see Related Jira Tickets below.

## Purpose

Get a new merchant to create their first volume discount offer with minimal
friction (activation), and give returning merchants a working hub to
monitor/manage offers. Also a **Built for Shopify (B4S) compliance
requirement**: Shopify's app review explicitly flagged that onboarding must
be visible immediately after install (QVD-135) — this is not just a UX nice-to-have.

## Business Rules

- **Trigger condition:** onboarding view shows automatically only for
  first-time users with **zero existing offers**. Once a merchant has ≥1
  offer, onboarding no longer shows automatically — the dashboard view takes
  over (QVD-18 AC). `Need confirm PO/Dev`: does "existing offers" count any
  offer regardless of status (draft/paused/expired), or active only?
- **B4S requirement (QVD-135):** onboarding must render on the Home screen
  immediately post-install, with a step-by-step setup guide and clear
  primary CTAs — this was a direct fix for a Shopify review flag, so
  regressions here carry App Store risk, not just a UX bug.
- **Onboarding view contents (QVD-18):** header "Hello {{Store name}}!",
  primary button "Create offer", empty-state card "Create your first deal!"
  / "Create your first volume discount offer to get started." / "Create offer" button.
- **Dashboard view contents (QVD-27),** 3 sections:
  1. **Theme Extension Status** (always visible, top of page) — shows
     Active/Inactive state of the Theme app extension, with a quick
     action/help button. Purpose: merchants need to know if their offers
     are actually visible on the storefront.
  2. **Setup Guide** — 2 steps in MVP: (1) Create offer, (2) Enable theme
     app extensions. Shows progress (e.g. "1/2 completed"). `Need confirm
     PO/Dev`: no defined UI state for 2/2 complete (does it hide/collapse?).
  3. **Recent Offers Table** — Name (editable), Status, Trigger, Combination,
     Actions (view/edit/duplicate/delete), paginated. "Manage offers" and
     "Create offer" buttons for navigation.
- **"Enable theme app extension" banner behavior (QVD-31):** by default,
  creating an offer does **not** make it visible on the storefront — the
  merchant must separately enable the Theme App Extension. Specifically:
  - Banner (type: Warning) appears on the dashboard page **specifically
    after a merchant creates an offer, if the extension is not yet
    enabled** — this is the concrete trigger for the Theme Extension
    Status / Setup Guide step 2 UI, not just "dashboard state" generally.
  - Copy: title "Important: Enable theme app extension to display app on
    your store"; message "To display the offer widget on your store,
    please enable the Theme App Extension. If not enabled, customers
    won't see your offer, which may reduce campaign visibility and
    engagement."; button "Enable Theme App Extension".
  - Clicking the button: redirects to the Shopify **Theme Editor in a new
    tab**, auto-selects "the correct theme", auto-inserts the app block
    ("Qikify: Volume Discount") into the relevant section (typically
    product page), and instructs the merchant to click **Save** to apply.
    `Need confirm PO/Dev`: which theme is "the correct theme" when a store
    has multiple themes (published/live theme, presumably) — matters if a
    merchant has draft/unpublished themes.
- **Dashboard ATC click stat (QVD-115):** shows add-to-cart click count for
  the last 30 days. **Cross-reference:** this metric functionally overlaps
  with the standalone [Analytics feature](08-analytics.md) — treat as "a
  metric hosted on the dashboard," not core onboarding/dashboard logic. See
  note added to `08-analytics.md`.

## Edge Cases

- Merchant with 0 offers → onboarding shown. Merchant creates their first
  offer → does the view switch to dashboard immediately/on next load, or
  require a manual refresh/navigation? `Need confirm`.
- Merchant deletes their only offer, returning to 0 offers — does onboarding
  reappear? Not specified in tickets. `Need confirm PO/Dev` — **Blocker**,
  this directly affects whether "onboarding never shows twice" is even true.
- Reinstall: does prior offer data (and thus onboarding-skip state) persist,
  or does a reinstalled store see onboarding again? `Need confirm`.
- Theme app extension toggled from the Shopify theme editor directly
  (outside this app) — does the Theme Extension Status block reflect it
  without any action inside the app?
- Store name with unusual values (very long, special characters, emoji) in
  "Hello {{Store name}}!" header — truncation/overflow.
- Large number of offers — Recent Offers Table pagination boundaries.
- Rapid repeated clicks on "Create offer" CTA during page load.
- Multiple staff accounts on the same store viewing Home simultaneously —
  is onboarding/dashboard state store-level (expected) or could it show
  inconsistently per session?
- Store with multiple themes (published + drafts) — does "Enable Theme App
  Extension" auto-select the published/live theme, or could it target the
  wrong one (QVD-31)?
- Merchant clicks "Enable Theme App Extension", the new tab opens to Theme
  Editor, but the merchant closes it or navigates away **without** clicking
  Save — does the banner/status correctly reflect "still not enabled"?
- Extension already enabled when the merchant creates their first offer —
  does the warning banner correctly not appear at all?

## Risks (from /analysis, 2026-08-27)

- **High — B4S compliance regression:** QVD-135 exists because Shopify's
  own review flagged this. A regression here risks Built for Shopify
  standing, not just a normal UX bug — treat as a priority regression area.
- **Medium — Theme Extension Status staleness:** if the app doesn't
  correctly poll/sync the real Shopify theme editor state, merchants could
  be misled about whether their offers are actually live on the storefront.
- **Medium — onboarding/dashboard trigger edge cases:** the 0-vs-≥1-offer
  binary condition has real consequences both directions (see Edge Cases).

## Testability Notes

Per [`qikify-volume.md` QA app note](../dev-references/references.md): **no
API access for QA automation** — everything here must be verified through
the UI manually (Playwright MCP browser-driven only, no
`APIRequestContext`). Practical implications:
- Onboarding↔dashboard switching: verifiable via UI (create/delete test
  offers, reload Home).
- Theme Extension Status accuracy: verifiable via UI, but needs a test
  store/theme where the extension can be deliberately toggled — may need
  Dev/Support help to set up.
- "Enable Theme App Extension" deep link (QVD-31): fully verifiable via UI —
  click button, confirm it opens Theme Editor in a new tab with the app
  block ("Qikify: Volume Discount") pre-inserted into the expected section,
  and that saving there flips the dashboard's status to Active on return.
- ATC click count (QVD-115): correctness of the underlying computation is
  not verifiable from the UI alone (would need real storefront ATC events
  or log/dev access) — at most, smoke-test that a number renders.
- B4S "immediately after install" requirement: only testable via an actual
  fresh install (or uninstall/reinstall cycle) on a test store.

## Known Issues

See [known-issues.md](../known-issues/known-issues.md) for entries tagged `onboarding-dashboard`.

## Related Jira Tickets

- [QVD-14](https://ownego.atlassian.net/browse/QVD-14) — Onboarding (MVP version), Epic
- [QVD-18](https://ownego.atlassian.net/browse/QVD-18) — Onboarding (MVP version), Story — AC + UX requirement
- [QVD-27](https://ownego.atlassian.net/browse/QVD-27) — Dashboard page (MVP), Epic — 3-section layout
- [QVD-31](https://ownego.atlassian.net/browse/QVD-31) — Enable theme app extension, Story — banner behavior + auto-insert deep link
- [QVD-115](https://ownego.atlassian.net/browse/QVD-115) — Update dashboard/home page — adds ATC click stat
- [QVD-135](https://ownego.atlassian.net/browse/QVD-135) — [B4S update] Onboarding — Shopify review requirement

## Source

- Jira (QVD project), via Atlassian MCP, analyzed 2026-08-27.
- QA, direct confirmation in conversation (2026-08-27) — combined-feature decision.
