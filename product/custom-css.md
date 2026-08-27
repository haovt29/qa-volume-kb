# Custom CSS (shared setting)

**Status:** skeleton

Not a standalone offer type — this is an input field in the advanced
settings of each offer type's widget, letting merchants target a class/
selector and apply CSS rules. Documented once here rather than duplicated
per feature. Moved out of `features/` (2026-08-27), see
[decisions/decision-log.md](../decisions/decision-log.md).

## Applies within

- [Multibuy – Mix & Match Combo](../features/04-multibuy-mix-match-combo.md)
- [Wholesale Pricing Offer](../features/05-wholesale-pricing-offer.md)
- [Quantity Break](../features/03-quantity-break.md)
- _TODO: confirm which other offer types also expose this field, and whether it's identical across all of them or has per-widget differences._

## Purpose

To customize an element in the widget, find the right class (or selector)
and apply CSS rules.

_TODO: rewrite/confirm against current app behavior._

## Business Rules

_TODO: fill in via /analysis against related Jira tickets. Things worth
confirming: is CSS scoped per-offer or applied globally across all widgets
on the store? Does it persist if the merchant switches offer type? Any
sanitization/limits on what CSS is accepted?_

## Edge Cases

_TODO_

## Known Issues

See [known-issues.md](../known-issues/known-issues.md) for entries tagged `custom-css`.

## Related Jira Tickets

_TODO_

## Source

- Help center (may be outdated): https://help.qikify.com/en/article/custom-css-11k13sw/
