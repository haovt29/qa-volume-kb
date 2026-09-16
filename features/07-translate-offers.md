# 7. Translate Offers

**Status:** in-review

Analyzed against Jira 2026-09-11 (QVD-43 epic, QVD-44 Portal spec, QVD-55
SDK spec — see Related Jira Tickets below), then **live-verified
2026-09-11, extended 2026-09-14 via Playwright MCP** against store
`test-vl-1` (Portal fully tested across all 3 offer types; storefront/SDK
translated-content rendering attempted but blocked by a test-environment
limitation — see Storefront display logic below). Several ticket specs
were found to not match shipped behavior 1:1 and are called out inline as
"corrected"/"confirmed live".

## Purpose

A **standalone, global Portal page** ("Translations" in the app's left
nav, alongside Manage offers/Variant swatches/Analytics) that lets a
merchant manage translated text for the storefront widget's UI strings
and for each individual offer's own customer-facing content (title,
tier/subtitle/badge/promotion text, etc.), across every language the
store supports. Distinct from [Translate Portal](10-translate-portal.md),
which is about translating the **merchant-facing Portal UI itself** (a
completely separate, fixed 9-language switcher at the bottom of the
Portal's Dashboard page) for the merchant's own use, not customer-facing
storefront content — confirmed as two genuinely separate mechanisms,
2026-09-14 (an earlier pass in this session briefly merged the two files
by mistake; that was wrong and has been reverted — see decision-log
2026-09-14).

## Business Rules

### Content covered — massively undercounted by the ticket, corrected 2026-09-11

QVD-44 only documented 3 "Default content" fields (out-of-stock message,
tier-quantity message, item unit) and 5 generic "Offer content" fields.
**Live `test-vl-1` has far more of both** — see the two sections below
for the full live-observed lists.

### Default content tab — confirmed live, 24 fields not 3

Live fields, in order (default/English values shown, several already
edited with test text by prior QA sessions on this store — e.g. "Item nè
hẹ", "Free nhé" — real store data is messy, same lesson as
[Variant Swatches](06-variant-swatches.md)):

Variant is out of stock message, Tier quantity not fully available
message, Item text, Items text, From text, Unavailable, Free (price for
free products), Details button, Pick text, View more products button
text, Done text, Reset selections text, Quantity text, Discount text,
Item price text, Reset text, Cancel text, Reset modal title, Reset modal
confirmation, Subtotal, Days, Hours, Minutes, Seconds.

This is clearly the **full shared UI-string vocabulary for every offer
widget's chrome** (stock messages + unit labels + pricing labels + button
labels + the *storefront widget's own* "Reset selections" confirmation
modal + countdown timer labels), not just the 3 fields QVD-44 called out.
Note **"Reset modal title"/"Reset modal confirmation" here are a
different thing from the Portal-level "Reset data" flow QVD-44
describes** — these are the *customer-facing* widget's own "Reset your
selections?" confirmation (seen live: title "Reset your selections?",
body "Are you sure? This will remove all selected variants and
quantities. This action cannot be undone.") — e.g. Wholesale's "Reset
selection" link (see
[05-wholesale-pricing-offer.md](05-wholesale-pricing-offer.md)).

### "Reset data" (QVD-44) — NOT found live, corrected 2026-09-11

**Ticket describes a "Reset data" button that appears once content has
been edited**, with a confirmation popup ("Reset Translations?" / "This
will remove all your edited translation and restore default values.").
**Live: no such button exists anywhere on this page**, despite this
store's Default content clearly having been edited already (visible test
text like "ủa", "nè hẹ", "lào" mixed into several fields). The page's
only action menu ("View actions") has exactly **one** item:
**"Translation to another language"** (see below) — no reset option.
`Need confirm`: whether "Reset data" was never built, was removed, or
lives somewhere else not found in this pass.

### Language switcher — only appears after an explicit action, corrected 2026-09-11

**Ticket implies the language switcher is simply visible whenever the
store has >1 language.** **Live behavior is different:** `test-vl-1`
already had 2 languages configured (**Polish**, **Vietnamese**) before
this test, yet the page's default view showed only the single "English /
Default" content editor, no switcher, no "Offer(s)" tab. Clicking **"View
actions" → "Translation to another language"** is what actually reveals:
- A **language switcher dropdown** (Polish/Vietnamese, "Polish" selected
  by default in this test).
- A now-visible **"Offer(s)"** tab alongside "Default content" (previously
  just "Default content" alone, even though this store clearly has many
  offers).
- Each Default content field now shows **2 columns**: original English
  (disabled, read-only) and the selected language's translation (mostly
  empty, a few pre-filled with real test values like "Free translation",
  "Details translation") — matching QVD-44's 2-column table spec closely,
  once you're actually in this mode.

So QVD-44's Case 1 vs. Case 2 distinction isn't really about "how many
languages the store has" — it's about **whether the merchant has clicked
into "Translation to another language" this session**; the page defaults
to a single-language editing view regardless of how many languages
actually exist underneath.

### Supported languages — confirmed live 2026-09-14: sourced from Shopify's own store Languages, not a fixed catalog

Live-checked `test-vl-1`'s native Shopify **Settings → Languages** page
(`/store/test-vl-1/settings/languages`, Shopify's own "Translate & Adapt"
localization system, unrelated to this app) and found exactly **English
(default) + Polish + Vietnamese published** — matching the 2 languages
(Polish/Vietnamese) seen in this feature's own "Translation to another
language" picker.

**This feature has no fixed language catalog of its own for offer/
storefront content.** The "Translation to another language" picker on
this Translations page (and the storefront's own language switcher) both
draw directly from whatever languages the merchant has already added
under Shopify's native **Settings → Languages** — this app translates
its own content (Default content + per-offer content) into however
many/whichever languages the store maintains there. Adding a language
for this feature to translate into means adding it in Shopify's own
Languages settings first — not something configured inside this app's
own UI at all. (Note: this is specifically about *this* feature's
language list — [Translate Portal](10-translate-portal.md) is a
confirmed-separate feature for the merchant-facing Portal UI, with its
own **fixed 9-language list** (English + 8 others) baked into the app,
unrelated to Shopify's Settings → Languages at all. E.g. Polish is
available here for shopper-facing offer translation but is **not** one
of Translate Portal's 9 options.)

### "Offer(s)" tab — confirmed live, matches ticket closely

- Heading "Offer Translations", a **"Search offers..."** search box, and
  a list of every existing offer (by internal name) each with its own
  **"Translate"** button — pagination present (only 1 page needed for
  this store's offer count).
- **Confirmed live: clicking "Translate" opens a real modal** (contrast
  with [Variant Swatches](06-variant-swatches.md), where the equivalent
  ticket claim turned out to be false — that one opens a full page
  instead). Translations' modal claim holds up.

### Offer translation modal — confirmed live, far richer than the ticket (Offer 15, Quantity Break)

- Alert banner at the top: **"Only translate the text outside the curly
  brackets."** — not in the ticket; a guardrail for fields containing a
  `{variable}` placeholder (e.g. Badge text's `{discount}`, per
  [03-quantity-break.md](03-quantity-break.md)'s live-confirmed default
  `UP TO {discount} OFF`).
- **Offer-level fields (live, Offer 15):** Offer title, Purchase option
  title, Subscription title, Subscription subtitle, One-time purchase
  title, One-time purchase subtitle, Heading content, Subheading content,
  Badge text — **8 more than QVD-44's simple "Offer title" listing**, and
  directly tied to Quantity Break's own live-confirmed Subscriptions
  toggle and countdown-timer heading/subheading (see
  [03-quantity-break.md](03-quantity-break.md)).
- **Per-tier fields (live, repeated for every tier — this offer has 3):**
  Title, Subtitle, Badge, Label text, **Promotion message** — the last
  one isn't documented anywhere in QVD-44's simple "Tier title/Subtitle/
  Badge/Highlight label" list.
- Same 2-column (original/disabled + translated/editable) layout as the
  Default content tab.
- **Cancel / Save** buttons at the bottom of the modal.

**Resolved (live-tested 2026-09-14): the field set is offer-type-specific
— confirmed 3 distinct, non-overlapping sets, one per offer type.**

- **Multibuy (Offer 16) — 6 offer-level fields + 2 per-tier fields, all
  different from Quantity Break's:** Offer title (Fixed price) — live
  `"BUY ANY {quantity_tier} ITEMS FOR {discount}"`; Offer title
  (Percentage discount) — live `"BUY ANY {quantity_tier} ITEMS GET
  {discount} OFF"` (**2 separate title fields for the 2 discount-type
  variants**, not 1 like Quantity Break); Remaining quantity message —
  live `"Add {remaining_quantity} more items to get this deal"`; Heading
  content; Subheading content; Badge text (live `MULTIBUY`). Per tier
  (3 tiers): **only** Title (live `"{quantity} for {tier_price}"`) and
  Badge (live `"Saved {saved_price}"`) — **no Subtitle/Label text/
  Promotion message** (Quantity Break's tiers have those, Multibuy's
  don't). No Subscription/Purchase-option/One-time-purchase fields at
  all (Multibuy has no Subscriptions feature).
- **Wholesale (Offer 17) — only 5 fields total, no per-tier fields at
  all:** Tier pricing title (live `TIER PRICING`), Variant title (live
  `VARIANTS`), Heading content, Subheading content, Badge text (live
  `BULK SAVING`). Matches Wholesale's own table-based tier UI (see
  [05-wholesale-pricing-offer.md](05-wholesale-pricing-offer.md)) — no
  individual tier cards to have their own Title/Subtitle/Badge, so no
  per-tier translation rows exist. **The "Only translate the text
  outside the curly brackets" alert did NOT appear for Wholesale** —
  confirms that alert is conditional on the offer's own fields actually
  containing a `{variable}` placeholder (none of Wholesale's 5 fields
  do), not a static banner shown for every offer.
- Confirms the field set is driven entirely by each offer type's own
  Portal settings (Multibuy's Fixed/Percentage discount type + Remaining
  quantity message; Wholesale's Tier pricing/Variant titles from its own
  General settings) — not a shared generic template.

### Storefront display logic (QVD-55) — attempted 2026-09-14, blocked by a test-environment limitation

- SDK is supposed to detect the current storefront's active language,
  load/display the translated content for it, and **fall back to the
  default language** if no translation exists — per the ticket. **Could
  not be directly verified this session.**
- **What was confirmed:** the storefront's native language switcher
  (Shopify's own `/localization` mechanism) does list **"Polski"** and
  **"Tiếng việt"** as real, selectable options on `test-vl-1` — so the
  store does have these locales published at the Shopify level, matching
  the 2 languages configured in this app's Translations page.
  - **What could NOT be confirmed:** actually switching into Polish/
    Vietnamese and observing the rendered widget content. 3 approaches
    were tried on the "Preview on Online Store" bypass link (used
    throughout this session to view the storefront without the store's
    password): clicking the language link in the UI, appending
    `?locale=vi` to the URL, and directly submitting the page's own
    `#HeaderLanguageForm` (POST to `/localization`) via
    `locale_code=vi`. **All 3 left the page rendered in English** — the
    `products_preview` bypass route appears not to honor/persist locale
    switching at all, which looks like a property of that specific
    preview mechanism rather than of this app. Directly navigating the
    real `test-vl-1.myshopify.com` domain with a `/vi/` path hit the
    store's password wall (no credentials available this session).
  - `Need confirm`: retest with either the store's actual password (to
    use the real domain, not the bypass-token preview route) or a
    different verification method — this specific limitation should not
    be read as evidence the feature doesn't work, only that it wasn't
    reachable with the tools available this session.
- **Two stock-related scenarios use translated error messages, with
  different UI treatment (per ticket, not live-tested):**
  1. **Variant is out of stock** — shown the moment the customer selects
     that variant; customer simply can't proceed with it for that tier.
     Error renders **inline within the widget**, near the affected
     tier/variant selector.
  2. **Tier quantity not fully available** — Add to cart gets **disabled**
     for that tier, with the error message shown **above the Add to cart
     button**. Prevents adding a tier when only some of its quantity is
     actually in stock.

## Edge Cases

_Some resolved via this session's live Portal check (2026-09-11); others
still drafted from ticket specs only._

- **Resolved (live 2026-09-11):** the "Reset data" button described in
  QVD-44 does not appear anywhere on the page, despite edited content
  existing. See Business Rules above.
- What happens to an offer's translations if the offer is deleted — do
  they get cleaned up, or orphaned? Not tested.
- Adding a **new** language after translations already exist for other
  languages — do existing default-content translations carry over/pre-
  fill anywhere, or does every new language start fully blank? Not
  tested (this store already had Polish/Vietnamese pre-existing; the
  "add a language" flow itself — via "Translation to another
  language" — wasn't tested for what happens with a genuinely new one).
- Editing an offer's original content (title/tier text/etc.) directly in
  that offer's own settings, **after** translations already exist for it
  — do the existing translations stay as-is (now possibly stale/out of
  sync with the new original), or get invalidated/flagged somehow? Not
  tested.
- **Resolved (live 2026-09-14):** Multibuy and Wholesale each have their
  own distinct translation field set, different from Quantity Break's
  and from each other — see Business Rules above.
- Interaction with [AI Offer Creation](02-ai-offer-creation.md) — does an
  AI-generated offer's title/tier text get a translation entry
  automatically, or does the merchant have to visit this page first same
  as any manually-created offer? Not tested.
- Whether the per-item unit label ("item") and the 2 stock error messages
  are truly global across **all** offer types (Quantity Break, Multibuy,
  Wholesale) or whether any offer type has its own variant of this
  copy — not distinguished in QVD-44/55, not tested.
- **Attempted, blocked (2026-09-14):** storefront rendering (QVD-55) —
  language detection, fallback behavior, and the 2 stock-related
  translated error messages. The storefront's language switcher does
  list Polish/Vietnamese as real options, but 3 different methods to
  actually switch into one on the "Preview on Online Store" bypass link
  all failed to change the rendered language. Needs either the real
  store password or a different test approach to verify. See Business
  Rules above for exactly what was tried.

## Known Issues

See [known-issues.md](../known-issues/known-issues.md) for entries tagged `translate-offers`.

## Related Jira Tickets

- [QVD-43](https://ownego.atlassian.net/browse/QVD-43) — Translation, Epic — status **Passed**
- [QVD-44](https://ownego.atlassian.net/browse/QVD-44) — [Portal] Translation page, Story — full Translations page spec; live testing found it significantly undercounts both the Default content fields (3 vs. 24 live) and the per-offer fields (5 vs. 14 live), and found no "Reset data" button
- [QVD-55](https://ownego.atlassian.net/browse/QVD-55) — [SDK] Display translated & default content, Story — storefront language detection/fallback + the 2 stock-related translated error messages; attempted live 2026-09-14, blocked by a test-environment limitation (see Business Rules)

## Source

- Jira (QVD project), via Atlassian MCP, analyzed 2026-09-11.
- Live Playwright MCP test, `test-vl-1`, 2026-09-11 (Portal: Default
  content tab, "View actions" menu, language switcher, Offer(s) tab, and
  Offer 15's translation modal).
- Live Playwright MCP test, `test-vl-1`, 2026-09-14 (Portal: Offer 16 and
  Offer 17 translation modals, confirming distinct per-offer-type field
  sets; storefront: 3 attempts to verify language-switching, all blocked
  by the preview-bypass route not honoring locale; Shopify's own
  Settings → Languages page, confirming this app has no independent
  language catalog — see Supported languages above).
- Help center (may be outdated): https://help.qikify.com/en/article/how-to-translate-offers-zpi4ln/
