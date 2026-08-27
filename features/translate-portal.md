# Translate Portal

**Status:** in-review

Distinct from [Translate Offers](translate-offers.md): Translate Offers
translates offer/promotion content, this feature translates the **portal**
(widget UI shown to shoppers) itself. Confirm exact relationship/overlap
between the two once both are analyzed — may share settings or UI.

## Purpose

Lets merchants translate the portal into multiple languages so shoppers see
it in their own language.

Confirmed languages (QA, 2026-08-27):
1. Tiếng Việt – Việt Nam
2. 日本語 – Tiếng Nhật
3. Français – Tiếng Pháp
4. Deutsch – Tiếng Đức
5. Español – Tiếng Tây Ban Nha
6. Italiano – Tiếng Ý
7. Português (Brasil)
8. Português (Portugal)

**Open question:** QA said "9 ngôn ngữ" (9 languages) but only 8 are listed
above — confirm the 9th language before treating this list as complete.

## Business Rules

_TODO: fill in via /analysis against related Jira tickets. Things worth
confirming: is language auto-detected from store/browser locale or manually
selected by shopper? Does it cover all portal text or only specific strings?
What's the fallback when a translation is missing for a string?_

## Edge Cases

_TODO — e.g. RTL/pluralization not applicable to this list, but check
untranslated-string fallback, switching language mid-session, and whether
Português (Brasil) vs Português (Portugal) differ in more than just locale code._

## Advanced Settings

_TODO: confirm whether this shares any settings with [Custom CSS](../product/custom-css.md) or [Translate Offers](translate-offers.md)._

## Known Issues

See [known-issues.md](../known-issues/known-issues.md) for entries tagged `translate-portal`.

## Related Jira Tickets

_TODO_

## Source

- QA, direct confirmation in conversation (2026-08-27).
