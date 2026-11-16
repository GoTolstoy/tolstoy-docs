---
name: store-analytics
description: Read a store's analytics report through the Tolstoy CLI or MCP - engaged shoppers, opens, orders and attributed revenue of its Tolstoy widgets, and the events it sends with the Tolstoy SDK, with the orders and revenue that follow them. Use when the user asks how the store's widgets perform, for store analytics numbers, or for the counts, buyers, orders or revenue of the store's own SDK events.
---

# Store analytics

The store analytics report is the calculation behind the merchant's Analytics
page in Tolstoy and its export. A number taken from it matches what the merchant
sees; a number built another way has another meaning and will not match.

## Prerequisites

The Tolstoy CLI (https://developers.gotolstoy.com/agents/cli) or the Tolstoy MCP
(https://developers.gotolstoy.com/agents/connect), signed in to the merchant's workspace.

## Read the report

```bash
tolstoy analytics get-store-analytics \
  --date-from 2026-11-01 \
  --date-to 2026-11-09 \
  --sections '["overview","details"]' \
  --json
```

- The period is UTC days, up to 180 days. The report reads tables that build
  each night, so it covers the days before today; events sent today appear after
  the next build, from about 06:30 UTC.
- Without `--app-url` the report reads the store saved at sign-in. `tolstoy
  stores list` lists the account's stores, and the report lists them in `stores`.
- `--widget-ids '["WIDGET_ID"]'` limits the report to some widgets, and
  `--channel` to `web` or `app` (Tapcart).
- `--window` (`24h`, `3d`, `7d`, `30d`; default `24h`) is how long after a widget
  open, or after an SDK event, an order still counts.
- The `overview` section holds `overview` (`current` and `previous` period),
  `daily`, `widgetCatalog`, `liveWidgetTypes`, `widgetTypes` and `widgets`. The
  `details` section holds `topVideos`, `topProducts`, `events`, `tryOn` and
  `chat`. A section that was not asked for is `null`.
- `null` means nothing was measured; `0` is a measured zero. Rows do not add up:
  one shopper or order can count under several widgets.

The Tolstoy MCP tool `get_store_analytics` takes the same inputs and returns the
same report. Its description holds the definition of every number.

## Events sent with the Tolstoy SDK

A store's developer can send the store's own events, such as clicks on a
button, with the Tolstoy SDK. The SDK is for code that the store runs on its own
pages; a Tolstoy AI widget sends its events with its analytics plugin instead.

```javascript
import { TolstoySDK } from 'https://widgets-sdk.gotolstoy.com/tolstoy-sdk.js';

const tolstoy = new TolstoySDK({ appKey: 'APP_KEY' });
tolstoy.analytics.track('expand_product_details', { productId: '123' });
```

Setup, the shopper's consent and the API:
https://developers.gotolstoy.com/storefront/tolstoy-sdk.

The report lists these events in `events` (details section) with widget type
`tolstoy-sdk` and the name `tolstoy-sdk.<name>`. Each of these rows has:

- `events`: how many times the event was sent. `shoppers`: how many different
  shoppers sent it.
- `buyers`: the shoppers who sent the event and then placed an order in this
  store within the window after it.
- `orders`: those orders. `revenue`: their whole amount, each order counted
  once, in the report's `currency`, rounded to 2 decimals.

`0` is a measured zero. Tolstoy gets the orders from the Tolstoy Shopify app; on
a store without it, Tolstoy receives no orders and `buyers`, `orders` and
`revenue` stay `0`. In `stores`, `receivesOrders: false` marks a store that sent
Tolstoy no orders in the last 30 days.

Revenue is the whole order, not the product that the event names. One order
counts under every SDK event and widget that its shopper touched, so rows do not
add up. On rows of other widget types the three fields are `null`. SDK events
are never opens, engaged shoppers, try-ons, add to carts or attributed revenue
of Tolstoy's widgets.

To plan which events to send and to judge the numbers against benchmarks, use
the `ecommerce-analytics` skill.
