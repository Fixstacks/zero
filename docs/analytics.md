# Google Analytics (GA4) and consent

Module: `Zero_GoogleAnalytics`. GA4 e-commerce events for the Zero
storefront, sent either straight to GA4 (gtag.js) or into a Google Tag
Manager container, with Google Consent Mode v2 and Zero's own consent banner.
It doesn't use RequireJS and keeps the storefront's strict CSP. Magento's own
Google Analytics and Gtag modules don't send anything on Zero pages.

## Setup

**Stores > Configuration > Zero > Google Analytics**

| Setting | |
| --- | --- |
| Enabled | Off by default |
| Send With | **GA4 directly**: gtag.js with the Measurement ID. **Google Tag Manager**: the same events go to the dataLayer in GA4 e-commerce format (`{event, ecommerce}`), and the container decides where they go |
| GA4 Measurement ID | `G-…`. If empty, the one in Sales > Google API > Google Analytics (Magento's own setting) is used |
| GTM Container ID | `GTM-…` |
| Debug Mode | Events show in GA4 DebugView and in the browser console |
| Ask for Consent | Zero's banner (Accept all / Reject all / Choose). No = everything granted from the start, only for stores without EEA/UK visitors |
| Before Consent | **Advanced**: Google's tag loads at once and sends cookieless pings until consent. **Basic**: nothing from Google loads until the visitor accepts |
| Privacy Policy URL | Linked from the banner |

A "Cookie Settings" link in the footer's Legal column reopens the banner.

## Events

| Event | When | Where it comes from |
| --- | --- | --- |
| `view_item_list` | category, search results | the page (cached pages carry it) |
| `select_item` | click on a listed product | the page |
| `view_item` | product page | the page |
| `search` | search results | the page |
| `add_to_cart`, `remove_from_cart` | cart changes | the server, after the POST |
| `add_to_wishlist` | wishlist | the server, after the POST |
| `view_cart` | cart page | the page |
| `begin_checkout` | Zero Checkout | the page |
| `add_shipping_info`, `add_payment_info` | shipping or payment method chosen | the checkout |
| `purchase` | order success page, once per order | the page |
| `login`, `sign_up` | sign in, account created | the server, after the POST |

Items use the SKU as `item_id`. Configurable products also carry the chosen
child's SKU as `item_variant`, and prices are in the store view's currency.

Events that happen in a POST (Add to Cart and the rest) are kept in the
session. The `zero_ga_events` cookie tells the next page to fetch them once
from `zero/analytics/events`. No other page makes a request for analytics.

## Consent

| Banner choice | Consent Mode v2 |
| --- | --- |
| Analytics | `analytics_storage` |
| Marketing | `ad_storage`, `ad_user_data`, `ad_personalization` |

Until the visitor chooses, everything is `denied` (with `wait_for_update`).
The choice is kept in the browser (`localStorage`, `zero-consent-v1`) and sent
as a consent update. In GTM mode the same defaults and updates go to the
container, so its own tags respect them.

## CSP

`etc/csp_whitelist.xml` allows Google's tag hosts for GA4 and Google Ads
(`*.googletagmanager.com`, `*.google-analytics.com`,
`*.analytics.google.com`, `*.g.doubleclick.net`, `*.google.com`). Tags in a
GTM container that load other hosts need those hosts too. Custom HTML tags
with inline code are blocked by the strict CSP.

## Tests

`dev/tests/zero-e2e/tests/analytics.spec.ts` reads the dataLayer with
requests to Google blocked. It covers the consent flow, lists, product,
cart, checkout steps, and a purchase sent once. It's skipped where the
module is off.

## Other trackers on the same events

Zero_MarketingPixels (Meta, TikTok, Pinterest, Microsoft Clarity) uses these
events too: analytics.js dispatches each one as `zero:analytics`
({ name, params }) and keeps them in `window.zeroAnalytics`. A module that
needs them registers an `Zero\GoogleAnalytics\Api\EventListenerInterface`
with `Zero\GoogleAnalytics\Model\Config` (di.xml, argument `listeners`):
while it is active the events are collected, and analytics.js runs without
loading Google's tag when Google Analytics itself is off.
