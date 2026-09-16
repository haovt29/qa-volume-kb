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
`features/03-quantity-break.md`, with each former article kept as a "Variant"
subsection (same-product tiers, Buy X Get Y, subscription products).
**Why:** QA confirmed these 3 help-center articles are not independent
features — they're the same underlying quantity-threshold mechanic applied
in 3 different contexts. Keeping them as 3 files would mean documenting
shared business rules 3 times and risking drift between copies. Named the
umbrella "Quantity Break" rather than "Volume Discounts" to avoid ambiguity
with the app/project name ("Volume").
**Source:** QA, direct confirmation in conversation.

## 2026-08-27 — Moved Custom CSS from features/ to product/

**Decision:** moved `features/custom-css.md` to `product/custom-css.md`, and
added an "Advanced Settings" cross-link to it from
`04-multibuy-mix-match-combo.md`, `05-wholesale-pricing-offer.md`, and
`03-quantity-break.md`.
**Why:** QA confirmed Custom CSS is not its own offer type — it's a single
shared input field present in the advanced settings of at least Multibuy,
Wholesale Pricing, and Quantity Break. Treating it as a peer feature file
implied it had its own trigger/business-rule the way an offer type does,
which is misleading. `product/` is reserved for cross-cutting docs (glossary,
feature-map), so a setting shared across offer types belongs there, not in
`features/`.
**Source:** QA, direct confirmation in conversation.

## 2026-09-03 — "Same-product tiers" variant name flagged as misleading; trigger scope and reward type are independent axes

**Decision:** kept the existing "Same-product tiers" / "Buy X Get Y" /
"Subscription products" variant names in `03-quantity-break.md` for now
(no rename yet), but added a prominent naming-correction note and a new
"Trigger Settings" section. Does **not** supersede the 2026-08-27 merge
decision above (the 3-variant merge itself still stands) — this only
corrects what the variant names actually mean.
**Why:** while drilling into discount-calculation edge cases, QA revealed
that Quantity Break offers pick a **trigger** (All products, All products
– Exclude, Collections, Collections – Exclude, Product tags, Specific
products/variants, or a multi-condition Custom trigger) completely
independently of the **reward type** (price break vs. free gift vs.
subscription overlay — the actual thing that distinguishes the 3
variants). "Same-product tiers" is a **reward-type** name (from the old
help-center article "Quantity Breaks for Same Product Offer") but was
being read as if it also constrained the **trigger** to a single product —
it doesn't. An offer with the "price break" reward can still be triggered
by "All products," "Collections," etc. No Jira ticket documenting the
Trigger Settings UI has been found yet (Multibuy has an analogous QVD-104
"Trigger settings" — Quantity Break's equivalent ticket, if any, is still
unidentified). Flagging rather than renaming unilaterally since this name
is already referenced elsewhere in the KB and in Testlane.
**Source:** QA, direct confirmation in conversation (2026-09-03).

## 2026-09-03 — Renamed Quantity Break's 3 variant headings to their actual template names

**Decision:** renamed the 3 variant section headings in
`03-quantity-break.md` from the KB-invented "Same-product tiers" / "Buy X
Get Y" / "Subscription products" to their actual current offer-type-list
template names: **"Quantity breaks for same product"**, **"Free Gift (Buy X
get Y free)"**, **"Volume discount for subscription"**.
Also updated `product/feature-map.md`'s Quantity Break entry to match.
Supersedes the "no rename yet" part of the 2026-09-03 entry above (that
entry's naming-conflict finding — trigger scope and reward type being
independent axes — still stands and is unaffected).
**Why:** the 2026-09-03 entry flagged the old names as misleading but held
off renaming because it wasn't a QA-confirmed rename, just a flag. QA has
now given the exact current template names directly, so this is no longer
a unilateral rename — it's applying QA-confirmed ground truth. Old names
kept as parenthetical "KB called this X before" notes in each section for
anyone following old links/references from before 2026-09-03.
**Source:** QA, direct confirmation in conversation (2026-09-03).

## 2026-09-03 — Corrected Quantity Break's 3 template names again, from a verbal description to the literal live UI text

**Decision:** corrected the 3 variant section headings in
`03-quantity-break.md` (and `product/feature-map.md`'s Quantity Break
entry) a second time, from the 2026-09-03 rename above to the **literal**
text read off the live "Create offer" template cards via Playwright MCP:
**"Quantity breaks for same product"** (plural "breaks," was singular
"break"), **"Buy X, get Y free"** (was "Free Gift (Buy X get Y free)" —
no "Free Gift (" wrapper exists on the actual card), **"Volume discount
for subscription"** (was "Subscription (Volume discount for
subscription)" — no "Subscription (" wrapper exists on the actual card).
Supersedes only the *exact wording* from the entry above — the
naming-conflict finding (trigger scope vs. reward type being independent
axes) and the decision to rename at all are unaffected.
**Why:** the earlier same-day rename was based on QA's verbal description
of the template names in chat, which turned out to be a paraphrase, not a
literal quote. Navigating to the live "Create offer" screen and reading
the actual card headings via the accessibility tree gave exact ground
truth, which differed in wording from the verbal description on all 3
names. Also confirmed live: a 6th card, "Product recommendations
(Upselling)," exists on this screen but is a cross-sell/app-install
promo (button reads "Free to install," not "Create") — not an offer
template, consistent with the 2026-08-27 decision to exclude that name
from `features/`.
**Source:** Live Playwright MCP session against store `test-vl-1`,
`/offers/create` screen, 2026-09-03.

## 2026-09-14 — Merged "Translate Portal" into "Translate Offers"; no fixed language list

**Decision:** merged `features/10-translate-portal.md` into
`features/07-translate-offers.md`. `10-translate-portal.md` is now a
short redirect stub pointing to `07`; `product/feature-map.md`'s feature
list updated to list only `07` (noting the absorption). Also resolved
`10`'s open "8 vs. 9 languages" question: there is no fixed language list
at all.
**Why:** `10-translate-portal.md` originally guessed it was a distinct
feature — translating the merchant-facing Portal UI itself, as opposed
to `07`'s customer-facing offer/storefront content. Analyzing `07` in
depth (2026-09-11/14) found no separate Jira ticket for a merchant-UI
translator — only QVD-43/44/55, which all describe the single
"Translations" page already documented in `07`. `10`'s own Purpose text
("lets merchants translate the portal into multiple languages so
**shoppers** see it in their own language") describes exactly that same
customer-facing page, not a merchant-UI translator. Separately, live-
checked `test-vl-1`'s native Shopify Settings → Languages page (Shopify's
own "Translate & Adapt" system) and found only English/Polish/Vietnamese
published — not matching `10`'s "8 or 9 languages" list at all. This
app has **no independent language catalog**: both the app's own
"Translation to another language" picker and the storefront's language
switcher draw directly from whatever languages the merchant has added
under Shopify's native Settings → Languages. Follows the same precedent
as the 2026-08-27 Custom CSS merge (QA-confirmed duplication is enough
to merge, without needing separate PO/Dev ticket confirmation).
**Source:** QA, direct investigation (Jira search + live Playwright MCP
session against `test-vl-1`'s app Translations page and Shopify's native
Settings → Languages page), 2026-09-14.

## 2026-09-14 — Reverted the "Translate Portal" merge above: they are two separate features

**Decision:** reverted the entire merge described in the previous entry.
`features/10-translate-portal.md` is restored as its own file (not a
redirect stub); `features/07-translate-offers.md`'s Purpose section again
states the two are distinct; `product/feature-map.md` and `README.md`
list both files separately again.
**Why:** QA (the user) directly corrected this in conversation —
`07-translate-offers.md` is offer/widget content shown on the SDK
(storefront), `10-translate-portal.md` is the merchant-facing Portal UI
for the merchant's own use. These are genuinely two different features,
not one. The previous entry's reasoning (no separate Jira ticket found,
matching Purpose-text wording) was circumstantial and turned out to be
wrong — QA's direct statement of the actual product scope overrides that
inference. The technical finding that `07`'s own language picker sources
from Shopify's native Settings → Languages (not a fixed catalog) still
stands and remains documented in `07` — only the "these are the same
feature" conclusion is retracted. `10`'s original open question (8 vs. 9
languages listed) is **still open** — not resolved by `07`'s finding,
since `10` is a different feature and may have its own separate language
list.
**Source:** QA (user), direct correction in conversation, 2026-09-14.

## 2026-09-14 — Found and live-confirmed "Translate Portal" as its own real feature; resolved the 8-vs-9-language question

**Decision:** filled in `features/10-translate-portal.md` for real, based
on the ticket QVD-159 ("Portal Translation," found via direct key lookup
after a broader Jira text search missed it) and a live Playwright MCP
test against `test-vl-1`. Resolved the file's original open question:
the "Language:" selector at the bottom of the Portal's Dashboard page has
**exactly 9 options** — English (default) + the 8 already-listed
languages. QA's "9 ngôn ngữ" count included English itself; the 8-language
list was already complete, nothing was missing.
**Why:** live-tested the selector directly: switching it to Tiếng Việt
instantly re-translated the entire embedded Portal UI (nav labels, page
heading, greeting, Setup guide, the offers table's column headers *and*
cell values like offer-type names and status text) while the surrounding
Shopify Admin chrome stayed in English — confirming this is the app's
own self-contained i18n layer, completely independent of both Shopify
Admin's language and [Translate Offers](07-translate-offers.md)'s
Shopify-Settings-sourced language list (Polish, e.g., is available in
`07` but is not one of this feature's fixed 9). This directly
corroborates the previous entry's correction (two genuinely separate
features) with hands-on evidence, not just QA's say-so.
**Source:** Jira QVD-159 (Sprint 23 goal field, via Atlassian MCP) + live
Playwright MCP session against `test-vl-1`'s Portal Dashboard, 2026-09-14.
