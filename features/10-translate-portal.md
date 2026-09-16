# 10. Translate Portal

**Status:** in-review

Distinct from [Translate Offers](07-translate-offers.md): Translate Offers
translates offer/widget content shown to **shoppers** on the SDK/
storefront; this feature translates the **merchant-facing Portal UI
itself** (nav, dashboard, tables, labels) — for the **merchant's own**
use, not shoppers. Confirmed genuinely separate features, 2026-09-14 (an
earlier pass in this session briefly merged the two by mistake, based on
a misreading of this file's own Purpose text and the absence of an
easily-searchable Jira ticket; QA corrected this directly and the ticket
was found — see Related Jira Tickets and decision-log 2026-09-14).

## Purpose

Lets merchants use the Qikify Volume Discounts **Portal** (the embedded
Shopify admin app itself — nav, Dashboard, tables, buttons, everything)
in their own language, independent of both the shopper-facing storefront
language and Shopify Admin's own language setting.

## Business Rules

### Language switcher — confirmed live 2026-09-14

- A **"Language:" dropdown** sits at the very bottom of the Portal's
  Dashboard page (`/apps/staging-bk-volume`), below the "Ultimate kit for
  conversion" promo cards — not in the header/nav, easy to miss on first
  glance.
- **Confirmed live: exactly 9 options** — this resolves the file's
  original open question ("QA said 9 ngôn ngữ but only 8 were listed"):
  **English (default) + 8 translated languages** = 9 total dropdown
  options. QA's "9 ngôn ngữ" count was including English itself as one
  of the 9 — the 8-language list was already complete. Live options,
  exact order: English, Tiếng Việt, 日本語, Français, Deutsch, Español,
  Italiano, Português (Brasil), Português (Portugal).
- **Selecting a language re-translates the entire Portal instantly**, no
  page reload needed. Confirmed live switching to Tiếng Việt: browser tab
  title changed ("Dashboard" → "Bảng điều khiển"), and essentially every
  visible string translated — left nav items ("Manage offers" → "Quản lý
  ưu đãi", "Translations" → "Dịch", "Variant swatches" → "Mẫu biến thể",
  "Analytics" → "Phân tích", "Feature requests" → "Yêu cầu tính năng"),
  page heading, greeting ("Hi Hao Vu!" → "Xin chào Hao Vu!"), Setup guide
  section, the offers table's **column headers** ("Internal name" → "Tên
  nội bộ", "Status" → "Trạng thái", "Trigger" → "Kích hoạt", "Combination"
  → "Kết hợp", "Actions" → "Hành động") and **cell values** (offer-type
  names: "Wholesale" → "Bán buôn", "Quantity break" → "Giảm giá theo số
  lượng", "Multibuy" unchanged; status values: "Running" → "Đang chạy",
  "Expired" → "Hết hạn"; "1 product" → "1 sản phẩm", "All" → "Tất cả").
  Even the dropdown's own label translated ("Language:" → "Ngôn ngữ:").
- This is clearly a **comprehensive, app-wide i18n layer** covering
  navigation, page chrome, and live data-table content (not just static
  labels) — not a partial/cosmetic translation.
- **Scope not yet fully mapped:** only the Dashboard page was checked
  this session. `Need confirm`: does the language selection persist when
  navigating to other Portal pages (Manage offers, Translations, Variant
  swatches, Analytics, Feature requests, individual offer settings), or
  is it a per-page/per-session setting that resets? Whether the selector
  itself appears on every page (only seen on Dashboard so far) or just
  this one.
- **Persistence not yet confirmed:** does the language choice persist
  across sessions/reloads (e.g. stored per merchant-user, per store, or
  just a client-side/cookie preference that resets)? Not tested.

### Relationship to other language settings

- **Independent of Shopify Admin's own language** — the surrounding
  Shopify chrome (top bar, left global nav — Home/Orders/Products/etc.)
  stayed in English throughout this test; only the embedded app's own
  UI (inside its iframe) translated. Confirms this is the app's own
  self-contained i18n, not inherited from the merchant's Shopify Admin
  locale setting.
- **Independent of [Translate Offers](07-translate-offers.md)** — that
  feature's language list is sourced dynamically from whatever languages
  the merchant has added under Shopify's native Settings → Languages
  (confirmed live: only English/Polish/Vietnamese on `test-vl-1`, per
  `07`'s own live testing). This feature's list is a **fixed set of 9**
  baked into the Portal itself, unrelated to Shopify's Settings →
  Languages at all — Polish, for instance, is available for shopper-
  facing offer translation but is **not** one of this feature's 9 Portal
  UI options.

## Edge Cases

_Some resolved via this session's live Portal check (2026-09-14); others
still open._

- **Resolved (live 2026-09-14):** the "8 vs. 9 languages" question — 9 =
  8 translated languages + English itself, not a missing 9th language.
- Does the language selection persist across pages/sessions, or reset?
  Not tested — only the Dashboard page, single session, was checked.
- Is any Portal content left untranslated (e.g. dynamic content like
  merchant-entered offer titles/internal names — correctly NOT
  translated, since those are the merchant's own text, not UI chrome;
  but worth checking less obvious spots like error/validation messages,
  modal dialogs, the individual offer settings pages' many field labels).
- Whether Português (Brasil) vs. Português (Portugal) actually differ in
  translated text (not just locale code) — not tested.
- RTL languages — not applicable to this list (no RTL language among the
  9), consistent with the original file's note.

## Advanced Settings

Shares no settings with [Custom CSS](../product/custom-css.md) — unrelated. Does **not** share a settings surface with [Translate Offers](07-translate-offers.md) either — confirmed as two independent language mechanisms, see Business Rules above.

## Known Issues

See [known-issues.md](../known-issues/known-issues.md) for entries tagged `translate-portal`.

## Related Jira Tickets

- [QVD-159](https://ownego.atlassian.net/browse/QVD-159) — Portal Translation, New Feature — status **Done**. No description/comments/subtasks in Jira itself, but its Sprint 23 goal field states: *"Portal translated and live in German, French, Spanish, Portuguese (Brazil), Portuguese (Portugal), Italian, and Vietnamese"* (7 of the 8 non-English languages — Japanese isn't in that specific sprint goal, but is live in the app today, so was likely added in a later untracked pass). 1 merged PR linked, no further detail available via Jira.

## Source

- Jira (QVD project), via Atlassian MCP — found 2026-09-14 after an
  initial broader search missed it; located via direct key lookup
  (QVD-159) recalled from earlier conversation context.
- Live Playwright MCP test, `test-vl-1`, 2026-09-14 (Portal Dashboard:
  found the "Language:" selector, switched to Tiếng Việt and back,
  confirmed live translation across nav/headings/table).
- QA, direct confirmation in conversation (2026-08-27, language list) and
  2026-09-14 (corrected this session's mistaken merge with `07`).
