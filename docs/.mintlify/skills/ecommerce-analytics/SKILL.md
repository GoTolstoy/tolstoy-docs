---
name: ecommerce-analytics
description: Plan, add and judge analytics for an online store - what to measure and why, which storefront events to send and what each one means, how to work out rates from event and shopper counts, how to judge a number against per-vertical benchmarks from Tolstoy's store data, and how to add it free with the Tolstoy SDK and read it with the Tolstoy CLI or MCP. Use when asked to add analytics or tracking to a store, to measure a store feature, page or change, to set up funnel or conversion tracking, to choose e-commerce metrics, or to judge numbers such as average order value or refund rate, even when Tolstoy is not named. To read a Tolstoy report that already exists, use store-analytics.
metadata:
  author: Tolstoy
  version: "1.0.0"
---

# E-commerce analytics

E-commerce analytics tells a store which shopper actions lead to orders, and
whether a change made them go up. Measure a few things that decide something,
give each event a fixed meaning, count through to the order, and judge each
number against a baseline.

## Principles

1. **Start from the decision.** Before you add an event, write the question it
   answers and what the store will do with the answer. An event that decides
   nothing is noise in the report.
2. **Revenue = sessions × conversion rate × average order value**, with the
   conversion rate as orders ÷ sessions. A feature can raise revenue only
   through one of the three. Name the one that it should move, and measure that
   one.
3. **Count shoppers, not events.** One shopper can send an event many times.
   Rates divide shoppers by shoppers, never raw event counts.
4. **Count through to the order.** Views and clicks measure interest. The share
   of shoppers who go on to order measures value, so send an event at the order
   too.
5. **Compare like with like.** Compare periods of the same length, with the same
   weekdays, and no sale or launch in only one of them.
6. **Users of a feature are not proof that it works.** Shoppers who choose a
   feature often were more likely to buy anyway, so a higher rate for them is a
   correlation. For proof, show the feature to a random half of shoppers and
   compare the two halves.
7. **Judge the difference, not each rate.** For two rates p₁ of n₁ shoppers
   and p₂ of n₂ shoppers, the 95% range of the difference is
   (p₂ − p₁) ± 1.96 × √(p₁(1−p₁)/n₁ + p₂(1−p₂)/n₂). 30 buyers of 1,000
   shoppers (3.0%) against 42 of 1,000 (4.2%) is a difference of 1.2 points
   ± 1.6, so it can be zero. Report it as an estimate, not as a proven effect,
   and collect more shoppers.

## What to measure

| Metric | Formula | It answers |
|---|---|---|
| Orders, revenue | Count and sum of the orders in the period | Does the store grow? |
| Average order value (AOV) | Revenue ÷ orders | Do shoppers spend more per order? |
| Conversion rate | Orders ÷ sessions | Do visits turn into orders? |
| Buyer rate | Shoppers who ordered ÷ shoppers who visited | Do visitors become buyers? |
| Step rate | Shoppers at a step ÷ shoppers at the step before | Where do shoppers drop out? |
| Feature buyer rate | Shoppers who ordered after the feature ÷ shoppers who used it | Does the feature lead to orders? |
| Refund rate | Refunded orders ÷ orders | Does the revenue stay? |

Say for each revenue number whether it includes discounts, shipping and tax.

## Events to send

| Event | Send it when |
|---|---|
| `page_view` | Any page of the store shows. |
| `product_view` | A product page shows. |
| `add_to_cart` | The cart accepts an item. |
| `checkout_started` | The shopper opens checkout. |
| `order_completed` | The order confirmation page shows, once per order. |
| `<feature>_view` | The feature shows to the shopper, for example `size_guide_view`. |
| `<feature>_open` | The shopper opens or uses the feature, for example `size_guide_open`. |
| `order_after_<feature>` | The order confirmation page shows within your window after `<feature>_open` (see the window rule below). |

- Send `order_completed` from a page where the store's own code runs. A hosted
  checkout that runs no store code cannot send it; there, take orders and
  buyers from the store platform.
- Put details, such as the product ID or the order ID, in the parameters.
  Tolstoy keeps them, but the report does not sum or group them. A value that
  you must count apart, such as the half of a test, goes in the event name:
  `size_guide_test_a`, `size_guide_test_b`.

## Give each event a meaning

Tolstoy's report gives its own widget events a meaning with fixed rules. A set
of event names counts as an open or as an add to cart. An interaction counts
only when the shopper acted: autoplay does not. An order counts for a widget
when the same shopper placed it within a window after an open, 24 hours by
default, with both in the report's period. Tolstoy does not know what your
event names mean, so you set the same kind of rules for them, and keep them in
the code:

1. **One meaning per name.** Write down each event name, what it means, and the
   rate it feeds, before you send it. Use one lower-case name for one action
   on every page, and never rename it: a new name starts a new count.
2. **Only deliberate acts are actions.** Send an action event, such as
   `size_guide_open`, on a click, a tap or a submit. A page load, autoplay,
   scrolling past and closing are not actions. Send what shows on its own as
   a `_view` event.
3. **Only on success, and once.** Send `add_to_cart` after the cart accepts the
   item, not on a click that can fail. Send `order_completed` once per order:
   keep the order ID in `localStorage` and skip the event when the page
   reloads.
4. **Apply the window rule in the store's code.** The report cannot tell
   whether one shopper sent two events. To credit an order to an action, keep
   the time of the action in `localStorage`, and at the order send a second
   event when the action came within the window. Choose the window before you
   read any numbers; 24 hours matches Tolstoy's report. Store and use the time
   only when the shopper allowed analytics: check the `tolstoy-cookie-policy`
   value that the store sets from its consent tool for the SDK.

   ```javascript
   const analyticsAllowed = () =>
     localStorage.getItem('tolstoy-cookie-policy') !== 'rejected';

   // When the shopper opens the size guide:
   if (analyticsAllowed()) {
     localStorage.setItem('size_guide_opened_at', String(Date.now()));
   }
   tolstoy.analytics.track('size_guide_open');

   // On the order confirmation page, once per order:
   const openedAt = Number(localStorage.getItem('size_guide_opened_at'));
   if (analyticsAllowed() && openedAt && Date.now() - openedAt <= 24 * 60 * 60 * 1000) {
     tolstoy.analytics.track('order_after_size_guide', { orderId });
   }
   ```

5. **Start the rate where the rule starts.** Divide the shoppers of
   `order_after_size_guide` by the shoppers of `size_guide_open`, not by all
   visitors.
6. **Rows do not add up.** One order can send several `order_after_<feature>`
   events, so do not add them up across features.

## Work out the numbers

Read the report from your own event plan. Each `tolstoy-sdk.<name>` row means
what your plan says: the action, the pages that send it, and its rule. For each
event name, the report gives two counts for the period: `events`, how many
times the event was sent, and `shoppers`, how many different shoppers sent it.
You compute every rate from these counts.

| You want | Compute it from one report |
|---|---|
| Orders | `events` of `order_completed` |
| Buyers | `shoppers` of `order_completed` |
| Orders per buyer | `events` ÷ `shoppers` of `order_completed` |
| Buyer rate | `shoppers` of `order_completed` ÷ `shoppers` of `page_view` |
| Step rate | `shoppers` of a step ÷ `shoppers` of the step before, such as `add_to_cart` ÷ `product_view` |
| Buyers after a feature | `shoppers` of `order_after_<feature>` |
| Feature buyer rate | `shoppers` of `order_after_<feature>` ÷ `shoppers` of `<feature>_open` |

- Ask for the whole period in one report. A shopper is counted once per
  report, so the `shoppers` of separate reports, such as one per day, do not
  add up.
- The report counts each name on its own within the period. A shopper who
  opened the size guide the day before the period and ordered within it adds
  to `order_after_size_guide` but not to `size_guide_open`. So each rate is a
  ratio of two counts, not the share of one group of shoppers. Ask for a period
  much longer than the time from the first event to the second, such as 28 days
  for a 24-hour window, so that these edges stay small.
- The SDK keeps one shopper ID per browser and sends nothing for a shopper who
  rejected analytics. Each count covers the browsers that allowed analytics,
  and one person on two devices counts twice. A ratio of two events shares
  these limits on both sides.

What the counts cannot show:

- **A strict funnel.** A ratio of two events does not check that the same
  shoppers did both within the period: a buyer can skip a step, come back in
  another browser, or have done the first step before the period. The window
  rule only makes sure that each `order_after_<feature>` comes from a browser
  that opened the feature, maybe before the period.
- **Revenue and AOV.** The report does not sum the order values that you send
  as parameters. Take revenue from the store's orders.
- **Today.** The report covers whole UTC days before today. Events of today
  show from about 06:30 UTC the next day.

## Measure a feature

1. Write the question and its metric: "Do shoppers who see the size guide buy
   more?" is the buyer rate per half.
2. Fix the test dates and the window before the test starts, for example 14
   days and 24 hours.
3. On the page of the feature, within the test dates, put each new shopper in
   half `a` or `b` at random, keep the half in `localStorage`, and show the
   feature to half `a` only. When analytics is allowed, also keep the time,
   and send `size_guide_test_a` or `size_guide_test_b`.
4. At the order, when a time is kept and the order comes within the window
   after it, send `order_after_size_guide_test_a` or
   `order_after_size_guide_test_b`, once per order.
5. Read one report from the first test day to the last test day plus the
   window, so it holds every shopper's start and full window. For each half,
   divide the `shoppers` of its order event by the `shoppers` of its test
   event: every buyer is one of the shoppers of the half, and each had the same
   window to order. The difference between the halves estimates the effect of
   the feature. It is an effect only when its 95% range (principle 7) does not
   include zero.

Without a split, the feature buyer rate and a comparison with the weeks before
the launch are correlations. Report them as such.

## Judge a number

1. Check the definition: the period, the time zone, the currency, what revenue
   includes, and the rules of your events.
2. Check the base: `0` is a measured zero. An event name that is missing from
   the report was not sent in the period, or is not among the 120 most
   frequent names. Compute the range of the difference (principle 7) before you
   call a change real.
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
delivery check, and what the report does not give.

**With the Tolstoy tools connected** (`list_workspaces`, `get_store_analytics`):

1. Take the app key from `currentAppKey` of `list_workspaces`.
2. Add the Tolstoy SDK to the store's pages, and send the events of your plan.
3. From about 06:30 UTC the next day, call `get_store_analytics` with
   `sections: ["details"]`.

**Without them:**

1. Run `npm install -g @tolstoy-ai/cli` and `tolstoy auth signup`, or
   `tolstoy auth login` for an existing account. Copy the `App key:` line,
   not the secret `Key:` line.
2. Add the Tolstoy SDK to the store's pages, and send the events of your plan.
3. From about 06:30 UTC the next day, run
   `tolstoy analytics get-store-analytics --sections '["details"]' --json`
   with `--date-from` and `--date-to`.

The report leaves out events sent from a page whose address contains
`localhost`, `127.0.0.1` or `shopifypreview.com`, or starts with `file`, and
events sent by a browser whose user agent contains `HeadlessChrome`,
`Lightpanda`, `Chrome-Lighthouse`, `Bytespider`,
`Shopify-Observe-Synthetic-Checks`, `+http://www.google.com/bot.html`,
`(+http://www.facebook.com/externalhit_uatext.php)`, `https://naver.me/spd` or
`(+https://whatis.contentkingapp.com)`. Tolstoy still answers them with HTTP
200. A test on your own machine, or in a headless browser such as Playwright in
its default mode, checks delivery only. To check the report, send the events
from the deployed store in a normal browser, one whose user agent contains none
of these.

Then note the events and their rules in the project's `AGENTS.md`, so that the
next agent reads the numbers the same way.

### Take from the store platform

The report does not give these; take them from the store platform, such as
Shopify Analytics, or from the store's own order data:

- Sessions, and the conversion rate per session.
- Revenue, AOV, refunds and returns.
- Repeat customers. The SDK does not identify a shopper by email or account.

## Read the report

To read and explain the report, use the `store-analytics` skill. It defines
every number. To install it:
`npx skills@latest add https://developers.gotolstoy.com --skill store-analytics`.
