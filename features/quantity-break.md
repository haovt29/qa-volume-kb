# Quantity Break

**Status:** skeleton

Umbrella feature: reward customers for buying more units. What used to be
listed on the help center as three separate articles are variants of the
same underlying mechanic, applied to different contexts — merged here per
QA (2026-08-27) to avoid documenting shared business rules three times.
See [decisions/decision-log.md](../decisions/decision-log.md).

## Purpose

_TODO: rewrite/confirm against current app behavior._ Core mechanic: define
quantity thresholds, apply a reward (price break, discount tier, or free
gift) once a customer's quantity in cart/order crosses each threshold.

## Variants

### Same-product tiers

(formerly "Quantity Breaks for Same Product Offer") — tiered pricing on a
single product page, reward scales with units of that same product bought.

- **Business Rules:** _TODO: fill in via /analysis against related Jira tickets._
- **Edge Cases:** _TODO_

### Free gift reward (Buy X Get Y)

(formerly "Buy X Get Y") — reward is a free/discounted gift item rather than
a price reduction on the purchased product itself, once a qualifying
quantity is reached.

- **Business Rules:** _TODO: fill in via /analysis against related Jira tickets._
- **Edge Cases:** _TODO_

### Subscription products

(formerly "Volume Discounts for Subscription Products") — same mechanic
applied where the purchased item is a subscription, interacting with
subscription purchase options (interval, first-order vs recurring price, etc.).

- **Business Rules:** _TODO: fill in via /analysis against related Jira tickets._
- **Edge Cases:** _TODO_

## Shared / cross-variant rules

_TODO — rules that apply across all three variants (e.g. how thresholds are
counted, rounding, currency handling, how UI selects which variant applies)._

## Advanced Settings

Shares the [Custom CSS](../product/custom-css.md) input field with other offer types.

## Known Issues

See [known-issues.md](../known-issues/known-issues.md) for entries tagged
`quantity-break`, `quantity-break/same-product`, `quantity-break/buy-x-get-y`,
or `quantity-break/subscription`.

## Related Jira Tickets

_TODO_

## Source

- Help center (may be outdated, originally 3 separate articles):
  - https://help.qikify.com/en/article/quantity-breaks-for-same-product-offer-1fr75vy/
  - https://help.qikify.com/en/article/buy-x-get-y-17jb4y7/
  - https://help.qikify.com/en/article/volume-discounts-for-subscription-products-e1sghp/
