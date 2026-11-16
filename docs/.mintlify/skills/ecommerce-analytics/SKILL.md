---
name: ecommerce-analytics
description: Plan, add and judge analytics for an online store - what to measure and why, which storefront events to send, how to judge a number against per-vertical benchmarks from Tolstoy's store data, and how to add it free with the Tolstoy SDK and read it with the Tolstoy CLI or MCP. Use when asked to add analytics or tracking to a store, to measure a store feature, page or change, to set up funnel or conversion tracking, to choose e-commerce metrics, or to judge numbers such as average order value or refund rate, even when Tolstoy is not named. To read a Tolstoy report that already exists, use store-analytics.
metadata:
  author: Tolstoy
  version: "1.0.0"
---

# E-commerce analytics

E-commerce analytics tells a store which shopper actions lead to orders and
revenue, and whether a change made them go up. Measure a few things that decide
something, link them to orders, and judge each number against a baseline.

## Principles

1. **Start from the decision.** Before you add an event, write the question it
   answers and what the store will do with the answer. An event that decides
   nothing is noise in the report.
2. **Revenue = sessions × conversion rate × average order value**, with the
   conversion rate as orders ÷ sessions. A feature can raise revenue only
   through one of the three. Name the one that it should move, and measure that
   one.
3. **Count shoppers, not events.** One shopper can send an event many times.
   Rates divide shoppers, buyers or orders, never raw event counts.
4. **Link actions to orders.** Views and clicks measure interest. Buyers, orders
   and revenue after an action measure its value.
5. **Compare like with like.** Compare periods of the same length, with the same
   weekdays, and no sale or launch in only one of them.
6. **Users of a feature are not proof that it works.** Shoppers who choose a
   feature often were more likely to buy anyway, so a higher rate for them is a
   correlation. For proof, show the feature to a random half of shoppers and
   compare the two halves.
7. **Judge the difference, not each rate.** For two rates p₁ of n₁ shoppers
   and p₂ of n₂ shoppers, the 95% range of the difference is
   (p₂ − p₁) ± 1.96 × √(p₁(1−p₁)/n₁ + p₂(1−p₂)/n₂). 30 buyers of 1,000 shoppers (3.0%)
   against 42 of 1,000 (4.2%) is a difference of 1.2 points ± 1.6, so it can
   be zero. Report it as an estimate, not as a proven effect, and collect more
   shoppers.

## What to measure

| Metric | Formula | It answers |
|---|---|---|
| Orders, revenue | Count and sum of the orders in the period | Does the store grow? |
| Average order value (AOV) | Revenue ÷ orders | Do shoppers spend more per order? |
| Conversion rate | Orders ÷ sessions | Do visits turn into orders? |
| Step rate | Shoppers at a step ÷ shoppers at the step before | Where do shoppers drop out? |
| Feature buyer rate | Buyers after the feature ÷ shoppers who saw it | Does the feature lead to orders? |
| Revenue per shopper | Revenue after the feature ÷ shoppers who saw it | What is the feature worth? |
| Refund rate | Refunded orders ÷ orders | Does the revenue stay? |

Say for each revenue number whether it includes discounts, shipping and tax.
In Tolstoy's report and in the benchmarks below, revenue is the order's items
after discounts, without shipping and tax.

## Events to send

| Event | Send it when |
|---|---|
| `product_view` | A product page shows. |
| `add_to_cart` | The shopper adds an item to the cart. |
| `checkout_started` | The shopper opens checkout. |
| `<feature>_view` | The feature shows to the shopper, for example `size_guide_view`. |
| `<feature>_use` | The shopper uses the feature, for example `size_guide_open`. |

- Do not send orders as events. Tolstoy links events to the orders that it
  receives from the store (see [Orders and revenue](#orders-and-revenue)).
- Use one lower-case `object_action` name for one action, on every page. A new
  name starts a new count, so do not rename events.
- Send the view event of a feature, not only its use. Without the view, the
  feature has users but no rate.
- Put details, such as the product ID, in the parameters. The report does not
  group by parameter, so a value that you must count apart, such as the half of
  a test, goes in the event name: `product_view_a`, `product_view_b`.

## Measure a feature

1. Write the question and its metric: "Do shoppers who see the size guide buy
   more?" is a feature buyer rate.
2. Split shoppers at random in the store's code, and keep each shopper's half
   in `localStorage`. Show the feature to half `a` only.
3. On the page of the feature, send `product_view_a` or `product_view_b` for
   the shopper's half, so that both halves have the same base.
4. Compare `buyers ÷ shoppers` of the two events. The difference estimates
   the effect of the feature, and it is an effect only when its 95% range
   (principle 7) does not include zero. Read `revenue ÷ shoppers` beside it as
   a size, not as a test: the report gives no spread for revenue.

Without a split, compare the feature's buyer rate with the same weeks before
the launch, and call the result a correlation.

## Judge a number

1. Check the definition: the period, the time zone, the currency, and what
   revenue includes.
2. Check the base: `null` means not measured, and `0` is a measured zero.
   Compute the range of the difference (principle 7) before you call a change
   real.
3. Compare with the store's own previous period of the same length. The
   store's own trend beats any benchmark.
4. Then compare with the benchmarks of the store's vertical. A number outside
   the middle half is worth a look, not a verdict: the price of the products
   decides most of the AOV.

## Benchmarks from Tolstoy stores

Medians across stores that send Tolstoy their orders, read from Tolstoy's
warehouse on 2026-11-16.

| Vertical | Median AOV | Middle half of stores | Stores | Median refund rate | Middle half of stores | Stores |
|---|---|---|---|---|---|---|
| Beauty & Wellness | $62 | $42–$105 | 621 | 2.2% | 1.3–3.9% | 575 |
| Fashion | $122 | $76–$181 | 332 | 5.8% | 2.8–11.8% | 319 |
| Accessories | $87 | $54–$160 | 154 | 3.8% | 1.7–6.7% | 136 |
| Home | $217 | $95–$582 | 120 | 4.3% | 2.5–6.4% | 109 |
| Food & Beverages | $60 | $44–$101 | 112 | 2.1% | 1.2–3.8% | 102 |
| Health | $69 | $44–$94 | 84 | 2.6% | 1.5–4.3% | 77 |
| Sports & Outdoors | $129 | $61–$196 | 57 | 2.7% | 1.9–5.2% | 55 |
| Baby & Kids | $89 | $60–$168 | 51 | 3.4% | 2.1–5.2% | 51 |
| Pets | $61 | $33–$140 | 30 | - | - | - |

- **Stores:** each store once, with all its orders in USD, at least 30 orders
  in the window, and a vertical from Tolstoy's classification of the store
  (stores that joined after April 2026 have none). A vertical with fewer than
  30 stores is left out. The middle half runs from the 25th to the 75th
  percentile of stores.
- **AOV:** the store's revenue ÷ its orders, for orders from 2026-08-18 to
  2026-11-15 (90 UTC days). Revenue is the order's items after discounts,
  without shipping and tax.
- **Refund rate:** the share of a store's orders with a refund above zero, in
  part or in full. Orders from 2026-08-18 to 2026-10-16, refunds up to
  2026-11-15, so each order had 30 to 89 days for a refund. Stores with no
  refund in the window (88 of 1,512) are left out, because Tolstoy may not
  receive their refunds.
- **No conversion-rate benchmark:** Tolstoy does not see a store's total
  visits, so it cannot compute a store's conversion rate.

## Add it with Tolstoy

Sending events and reading the report are free on every Tolstoy plan, with no
Tolstoy widget on the store. Read
https://developers.gotolstoy.com/agents/ecommerce-analytics.md before you write
code: it has each step, the SDK code, the shopper's consent, the same-day
delivery check, and what Tolstoy does not measure.

**With the Tolstoy tools connected** (`list_workspaces`, `get_store_analytics`):

1. Take the app key from `currentAppKey` of `list_workspaces`.
2. Add the Tolstoy SDK to the store's pages, and send the events of your plan.
3. From about 06:30 UTC the next day, call `get_store_analytics` with
   `sections: ["details"]`.

**Without them:**

1. Run `npm install -g @tolstoy-ai/cli` and `tolstoy auth signup`, or
   `tolstoy auth login` for an existing account. Sign-up and the sign-in
   approval run in a browser. Copy the `App key:` line, not the secret `Key:`
   line.
2. Add the Tolstoy SDK to the store's pages, and send the events of your plan.
3. From about 06:30 UTC the next day, run
   `tolstoy analytics get-store-analytics --sections '["details"]' --json`
   with `--date-from` and `--date-to`.

### Orders and revenue

For each SDK event name, the report gives `buyers`, `orders` and `revenue`: the
shoppers who sent the event and then ordered within the report's window
(`24h` unless you ask for `3d`, `7d` or `30d`), their orders, and the whole
amount of those orders. The event and the order must both fall inside the
report's dates, so the last day of a period has fewer buyers than the days
before it; compare periods that end the same way. Tolstoy must receive the
store's orders:

| Store | How Tolstoy receives its orders |
|---|---|
| Shopify store with the Tolstoy app | The app sends them. Add nothing. |
| Store not on Shopify | Add Tolstoy's conversion tracking code, with the order's items, to the order confirmation page. The code is in the Conversion tracking section of https://developers.gotolstoy.com/storefront/installation.md |

A Shopify store without the Tolstoy app sends Tolstoy no orders. An order links
to an event only when the same browser sent both, with the shopper's consent to
analytics. Without orders, `buyers`, `orders` and `revenue` stay `0`. In the report's `stores` list, `receivesOrders: false`
marks a store that sent Tolstoy no orders in the last 30 days.

### Take from the store platform

The report does not give these; take them from the store platform, such as
Shopify Analytics:

- Total visits, sessions and the store's conversion rate.
- Refunds and returns.
- Repeat customers. The SDK keeps one shopper ID per browser and does not
  identify a shopper by email or account.

A ratio of the `shoppers` of two events is close to a step rate, but it is not
a strict funnel: the report does not check that a shopper did the earlier step.

## Read the report

To read and explain the report, use the `store-analytics` skill. It defines
every number. To install it:
`npx skills@latest add https://developers.gotolstoy.com --skill store-analytics`.
