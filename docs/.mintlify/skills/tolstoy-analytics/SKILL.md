---
name: tolstoy-analytics
description: Measure a storefront widget that Tolstoy did not build with the Tolstoy SDK, and read a store's analytics report through the Tolstoy CLI. Use when the user builds or edits their own widget and wants its shopper events, engagement and sales measured, or asks for store analytics numbers.
---

# Store analytics and outside widgets

An outside widget is a widget that Tolstoy did not build, on the merchant's own
storefront pages. It records shopper events with the Tolstoy SDK. Its results
come from the same report as the merchant's Analytics page, read for the outside
widgets only.

## Prerequisites

The Tolstoy CLI (https://developers.gotolstoy.com/agents/cli) or the Tolstoy MCP
(https://developers.gotolstoy.com/agents/connect), signed in to the merchant's workspace.
The developer guide is https://developers.gotolstoy.com/storefront/tolstoy-sdk.

## Add the Tolstoy SDK to a widget

The SDK is an ES module on the Tolstoy CDN. One SDK serves every widget on the
page; `tolstoy.analytics.widget(widgetId)` returns the trackers of one widget:

```javascript
import { TolstoySDK } from 'https://widgets-sdk.gotolstoy.com/tolstoy-sdk.js';

const tolstoy = new TolstoySDK({ appKey: 'APP_KEY' });
const sizeQuiz = tolstoy.analytics.widget('01JC8Z6Q4M3T9V2X7B5N1K0RWD');

sizeQuiz.trackEmbedViewOnVisibility(widgetElement);
sizeQuiz.trackSessionStartOnInteraction(widgetElement);
```

- `appKey`: the workspace's app key. It is in the code that Tolstoy shows in
  **Website setup & tracking** (open a widget, click its status button).
- `widgetId`: one id for each widget, chosen by the developer and kept the same
  across releases. The report groups numbers by it. It must be a ULID (26
  characters, upper case) or 13 lowercase letters and digits; the SDK throws for
  any other shape. The same id returns the same trackers.
- Every event of the SDK is marked as an outside-widget event. Marked events
  never count toward Shoppable Videos view billing and never enter the numbers
  of the widgets that Tolstoy built.
- Call `tolstoy.destroy()` when the page no longer needs the SDK; it sends the
  queued events.

## Consent

The SDK sends no event and stores no shopper id until the shopper allows
analytics. Events from before that are dropped, not held. At each event it reads
the `tolstoy-cookie-policy` value in `localStorage`: `accepted` allows analytics
and `rejected` blocks it. The widget sets this value from the store's consent
tool, before its first event and again when the shopper changes the answer.
Without the value, the SDK reads the consent tools that the page already loaded
(Shopify's Customer Privacy API, Cookiebot, OneTrust); with none, analytics is
allowed. The SDK does not load or wait for a consent tool.

On Shopify, the answer comes from the Customer Privacy API:

```javascript
const passConsent = (analyticsAllowed) =>
  localStorage.setItem('tolstoy-cookie-policy', analyticsAllowed ? 'accepted' : 'rejected');

window.Shopify.loadFeatures([{ name: 'consent-tracking-api', version: '0.1' }], (error) => {
  if (error) throw error;
  passConsent(window.Shopify.customerPrivacy.analyticsProcessingAllowed());
  startWidget(); // creates TolstoySDK and sends the first events
});

document.addEventListener('visitorConsentCollected', (event) => passConsent(event.detail.analyticsAllowed));
```

Shopify calls the `loadFeatures` callback at once when the page already loaded
the API, so this code runs after `startWidget` and everything it uses (such as
`linkCart` below) are defined. With another consent tool, call `passConsent`
from that tool's consent callback.

## Events and what the report counts

The report reads these SDK calls. A report number stays empty when its events
are not sent.

| Tracker of `tolstoy.analytics.widget(widgetId)` | Report number |
|---|---|
| `trackEmbedView()` or `trackEmbedViewOnVisibility(element)` | Impressions |
| `trackSessionStart()` or `trackSessionStartOnInteraction(element)` | Opens. An order counts for the widget when the same shopper orders within the window after an open. |
| `trackVideoWatched({ videoId, videoWatchedTime, ... })` | Plays and engaged shoppers (watch time above 0) |
| `trackAddToCart({ productId, variantId, ... })` | Add to carts, and in-video revenue for the ordered variant |
| `productId`, `productIds` or `currentPageProductId` on `trackVideoLoaded({ videoId, ... })` or `trackVideoWatched`, and `currentPageProductId` on `trackSessionStart` | Post-video revenue for the ordered product |
| `trackEvent(name, properties)` | Listed by name in the `events` of the details section |

Product and variant ids are the store's ids. The video and cart trackers turn
Shopify `gid://` ids into numbers; pass `currentPageProductId` to
`trackSessionStart` as a number. `videoId` must be a UUID: the event intake drops any other value.

## Orders

On a Shopify store with the Tolstoy app, the widget writes the SDK's shopper id,
`tolstoy.anonymousId`, to the cart attribute `__tolstoyAnonymousId`,
and Tolstoy reads it from the order. Write it only when the shopper allowed
analytics, after the SDK sent an event such as the open: at its first allowed
event the SDK takes the id that another widget on the page stored first.

```javascript
let isCartLinked = false;

const linkCart = async () => {
  if (isCartLinked || localStorage.getItem('tolstoy-cookie-policy') !== 'accepted') return;
  isCartLinked = true;
  await fetch(`${window.Shopify.routes.root}cart/update.js`, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ attributes: { __tolstoyAnonymousId: tolstoy.anonymousId } }),
  });
};

widgetElement.addEventListener('click', linkCart);
```

On another platform, add the conversion tracking code to the order confirmation
page (developers.gotolstoy.com, Storefront > Installation); it reads the shopper
id that the SDK stores. An order links only when the shopper used the widget in
the same browser.

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
