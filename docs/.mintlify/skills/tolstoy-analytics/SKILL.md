---
name: tolstoy-analytics
description: Measure a storefront widget that Tolstoy did not build with the Tolstoy widgets SDK, and read a store's analytics report through the Tolstoy CLI. Use when the user builds or edits their own widget and wants its shopper events, engagement and sales measured, or asks for store analytics numbers.
---

# Store analytics and outside widgets

An outside widget is a widget that Tolstoy did not build, on the merchant's own
storefront pages. It records shopper events with the Tolstoy widgets SDK. Its
results come from the same report as the merchant's Analytics page, read for the
outside widgets only.

## Prerequisites

The Tolstoy CLI (https://developers.gotolstoy.com/agents/cli) or the Tolstoy MCP
(https://developers.gotolstoy.com/agents/connect), signed in to the merchant's workspace.
The developer guide is https://developers.gotolstoy.com/storefront/widgets-sdk.

## Add the SDK to a widget

The SDK is an ES module on the Tolstoy CDN:

```javascript
import { TolstoySDK } from 'https://widgets-sdk.gotolstoy.com/widgets-sdk.js';

const tolstoy = new TolstoySDK({
  appKey: 'APP_KEY',
  widgetId: '01JC8Z6Q4M3T9V2X7B5N1K0RWD',
  analyticsSource: 'size-quiz',
  viteDev: false,
  outsideWidget: true,
});
```

- `appKey`: the workspace's app key. It is in the code that Tolstoy shows in
  **Website setup & tracking** (open a widget, click its status button).
- `widgetId`: one id for each widget, chosen by the developer and kept the same
  across releases. The report groups numbers by it. It must be a ULID (26
  characters, upper case) or 13 lowercase letters and digits; the SDK throws for
  any other shape when `outsideWidget` is `true`.
- `outsideWidget: true` marks every event of the widget. Marked events never
  count toward Shoppable Videos view billing and never enter the numbers of the
  widgets that Tolstoy built.
- `analyticsSource`: `trackEvent('click')` sends `<analyticsSource>.click`.
- `viteDev: true` turns analytics off. In a Vite app pass `import.meta.env.DEV`.
- Call `tolstoy.destroy()` when the widget leaves the page; it sends the queued
  events.

## Consent

The SDK sends no event and stores no shopper id until the shopper allows
analytics. Events from before that are dropped, not held. It follows Shopify's
Customer Privacy API, Cookiebot and OneTrust, and the `tolstoy-cookie-policy`
value in `localStorage`: `rejected` blocks analytics and `accepted` allows it.
With no consent tool and no stored value, analytics is allowed.

## Events and what the report counts

The report reads these SDK calls. A report number stays empty when its events
are not sent.

| SDK call | Report number |
|---|---|
| `trackEmbedView()` or `trackEmbedViewOnVisibility(element)` | Impressions |
| `trackSessionStart()` or `trackSessionStartOnInteraction(element)` | Opens. An order counts for the widget when the same shopper orders within the window after an open. |
| `trackVideoWatched({ videoId, videoWatchedTime, ... })` or `createNativeVideoAnalyticsSession(...)` | Plays and engaged shoppers (watch time above 0) |
| `trackAddToCart({ productId, variantId, ... })` | Add to carts, and in-video revenue for the ordered variant |
| `productId`, `productIds` or `currentPageProductId` on `trackVideoLoaded` or `trackVideoWatched`, and `currentPageProductId` on `trackSessionStart` | Post-video revenue for the ordered product |
| `trackEvent(name, properties)` | Listed by name in the `events` of the details section |

Product and variant ids are the store's ids; the SDK turns Shopify `gid://` ids
into numbers. `videoId` must be a UUID: the event intake drops any other value.

## Orders

On a Shopify store with the Tolstoy app, the SDK writes the shopper id to the
cart attribute `__tolstoyAnonymousId` at the first event the shopper allows, and
Tolstoy reads it from the order. On another platform, add the conversion
tracking code to the order confirmation page (developers.gotolstoy.com,
Storefront > Installation). An order links only when the shopper used the widget
in the same browser.

## Read the report

```bash
# Outside widgets of the default store
tolstoy analytics get-store-analytics \
  --date-from 2026-11-01 \
  --date-to 2026-11-09 \
  --outside-widgets \
  --sections '["overview","details"]' \
  --json

# The widgets Tolstoy built: the same command without --outside-widgets
```

- `--app-url` selects another store from `tolstoy stores list`.
- `--widget-ids '["01JC8Z6Q4M3T9V2X7B5N1K0RWD"]'` limits the report to some widgets.
- `--window` (`24h`, `3d`, `7d`, `30d`) is how long after an open an order still counts.
- `overview.current` holds the totals, `widgets` one row per widget id, and
  `daily` the days. Outside widgets have no name: `widgetName` is `null`.
- The report reads tables that build each night and covers UTC days before
  today. Events sent today appear after the next build, from about 06:30 UTC.

The Tolstoy MCP tool `get_store_analytics` returns the same report; pass
`outsideWidgets: true` for outside widgets.
