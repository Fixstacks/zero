# Zero for Magento Open Source

A storefront for Magento Open Source built on the web platform: server-rendered
HTML, HTMX for the parts that refresh, Tailwind CSS v4 and daisyUI. No
RequireJS, jQuery or Knockout on the storefront. Strict CSP, WCAG 2.2 AA,
a one-page checkout without Knockout, and the modules a store needs.

- Demo store: https://demo.zerofront.dev
- Documentation: https://zerofront.dev/docs (the same pages are in `docs/` here)
- Site: https://zerofront.dev

## Install

Requires Magento Open Source 2.4.7–2.4.9 and PHP 8.2–8.4 with the GD extension built with WebP (`php -r 'var_dump(function_exists("imagewebp"));'` says `true`; Magento's own PHP requirements cover the rest). Without WebP in GD, Zero skips its WebP image copies and serves the originals.

`zerofront/zero` is the theme and what every store needs: the checkout,
instant navigation, search, SEO, CMS, banners, the blog, mail, abandoned
carts and every payment (Stripe, Braintree, PayPal, Adyen, Klarna, Mollie,
Amazon Pay, Authorize.Net): the ✓ in the table below. Add any other module
with its own `composer require`, now or later.

```bash
composer require zerofront/zero
bin/magento setup:upgrade
bin/magento setup:di:compile           # production mode
bin/magento setup:static-content:deploy # production mode
bin/magento cache:flush
```

Then choose the theme: Content > Design > Configuration > your store view >
**Zero**. Every module is on by default except the ones that need an account
with someone (payments, analytics, social login, SES mail): those wait under
Stores > Configuration > **Zero** until you fill in their keys.

## Every package

Each package installs on its own with `composer require`; the theme needs
`module-zero-core` and `module-zero-checkout` and brings them along. ✓: part
of `zerofront/zero`.

| | Package | In zero | What it does |
| --- | --- | --- | --- |
| Storefront | [`zerofront/theme-frontend-zero`](https://github.com/zerofrontdev/theme-frontend-zero) | ✓ | The theme: templates, Tailwind/daisyUI styles, htmx |
| Storefront | [`zerofront/module-zero-core`](https://github.com/zerofrontdev/module-zero-core) | ✓ | Storefront core for the Zero theme (HTMX fragments for private content, view models, no Knockout) |
| Storefront | [`zerofront/module-zero-checkout`](https://github.com/zerofrontdev/module-zero-checkout) | ✓ | A one-page checkout without Knockout: server-rendered HTML, one native form, HTMX for the parts that refresh |
| Storefront | [`zerofront/module-zero-instant-navigation`](https://github.com/zerofrontdev/module-zero-instant-navigation) | ✓ | The next page loads while the shopper still points at its link (Speculation Rules API) |
| Content | [`zerofront/module-zero-banner`](https://github.com/zerofrontdev/module-zero-banner) | ✓ | Banners and sliders managed in the admin, rendered without JavaScript (CSS scroll snap) |
| Content | [`zerofront/module-zero-cms`](https://github.com/zerofrontdev/module-zero-cms) | ✓ | CMS pages and blocks built from typed components in a visual editor, with AI help; no page builder markup |
| Content | [`zerofront/module-zero-blog`](https://github.com/zerofrontdev/module-zero-blog) | ✓ | A blog: posts written in the Zero CMS editor, categories, tags, authors, products in posts, comments |
| Content | [`zerofront/module-zero-mega-menu`](https://github.com/zerofrontdev/module-zero-mega-menu) |  | A mega menu managed in the admin: full-width panels with subcategory columns, extra links and a promo card, plus the menu's own links; CSS only |
| Catalog | [`zerofront/module-zero-search`](https://github.com/zerofrontdev/module-zero-search) | ✓ | Store search that forgives typos and finds categories, pages and posts too |
| Catalog | [`zerofront/module-zero-layered-navigation`](https://github.com/zerofrontdev/module-zero-layered-navigation) |  | Better filters: several values of one filter at once, counts that stay right, a price range by the customer's own numbers |
| Catalog | [`zerofront/module-zero-brands`](https://github.com/zerofrontdev/module-zero-brands) |  | Shop by brand: a page per brand, an A–Z of brands, the brand on product pages |
| Catalog | [`zerofront/module-zero-seo`](https://github.com/zerofrontdev/module-zero-seo) | ✓ | Structured data (schema.org JSON-LD), hreflang, canonical URLs and meta descriptions |
| Catalog | [`zerofront/module-zero-reviews`](https://github.com/zerofrontdev/module-zero-reviews) |  | Product reviews with photos, "verified purchase" and a Google product reviews feed |
| Catalog | [`zerofront/module-zero-stock-alert`](https://github.com/zerofrontdev/module-zero-stock-alert) |  | "Tell me when it's back" on sold-out products, for any visitor, and one email when it is |
| Catalog | [`zerofront/module-zero-quick-view`](https://github.com/zerofrontdev/module-zero-quick-view) |  | "Quick view" on product cards: the product in a dialog with its photo, price and stock, and Add to Cart for a product with nothing to choose |
| Catalog | [`zerofront/module-zero-product-labels`](https://github.com/zerofrontdev/module-zero-product-labels) |  | More product badges next to New and Sale: the product's own (Bestseller, Limited), Only a few left, Free shipping |
| Catalog | [`zerofront/module-zero-size-chart`](https://github.com/zerofrontdev/module-zero-size-chart) |  | Size guides on product pages: tables made in the admin, shown by category or per product in a dialog |
| Catalog | [`zerofront/module-zero-unit-price`](https://github.com/zerofrontdev/module-zero-unit-price) |  | The price per kilogram, litre, metre or piece under every price, as EU price indication rules ask (per 100 g / 100 ml for small packs) |
| Catalog | [`zerofront/module-zero-product-attachments`](https://github.com/zerofrontdev/module-zero-product-attachments) |  | Files on product pages (manuals, data sheets, certificates): attached by SKU in the admin, a Downloads section with kind and size |
| Catalog | [`zerofront/module-zero-bought-together`](https://github.com/zerofrontdev/module-zero-bought-together) |  | "Often bought together" on the product page, from the orders: picks with checkboxes, a total and one Add to Cart |
| Selling | [`zerofront/module-zero-promotions`](https://github.com/zerofrontdev/module-zero-promotions) |  | More cart price rule actions: "every Nth item, the cheapest, X% off" and a free gift |
| Selling | [`zerofront/module-zero-reward-points`](https://github.com/zerofrontdev/module-zero-reward-points) |  | Reward points: customers earn them on paid orders and pay part of an order with them |
| Selling | [`zerofront/module-zero-gift-card`](https://github.com/zerofrontdev/module-zero-gift-card) |  | Gift cards: sold as a product, emailed once paid, spent at checkout like money |
| Selling | [`zerofront/module-zero-quote-request`](https://github.com/zerofrontdev/module-zero-quote-request) |  | Request a quote from the cart or a product, and "price on request" products |
| Selling | [`zerofront/module-zero-abandoned-cart`](https://github.com/zerofrontdev/module-zero-abandoned-cart) | ✓ | One reminder email for a cart left with items in it |
| Selling | [`zerofront/module-zero-store-pickup`](https://github.com/zerofrontdev/module-zero-store-pickup) |  | Magento's In-Store Pickup in Zero Checkout, only stores that can take the whole cart, and a store locator page with maps on demand |
| Selling | [`zerofront/module-zero-delivery-date`](https://github.com/zerofrontdev/module-zero-delivery-date) |  | A delivery date and time window picked in checkout, kept on the order and shown everywhere the order is |
| Selling | [`zerofront/module-zero-quick-order`](https://github.com/zerofrontdev/module-zero-quick-order) |  | A quick order page: SKUs and quantities, a pasted list or a CSV file, straight into the cart |
| Marketing | [`zerofront/module-zero-product-feed`](https://github.com/zerofrontdev/module-zero-product-feed) |  | The catalog as XML feeds for Google Merchant Center, Hotline, Rozetka and Prom.ua, one per store view |
| Marketing | [`zerofront/module-zero-consent`](https://github.com/zerofrontdev/module-zero-consent) |  | A cookie consent banner and settings for every tracker (GDPR), Global Privacy Control and "Do Not Sell" (CCPA) |
| Marketing | [`zerofront/module-zero-google-analytics`](https://github.com/zerofrontdev/module-zero-google-analytics) |  | GA4 e-commerce events with gtag.js or Google Tag Manager, with Consent Mode v2 |
| Marketing | [`zerofront/module-zero-marketing-pixels`](https://github.com/zerofrontdev/module-zero-marketing-pixels) |  | Meta, TikTok, Pinterest and Microsoft Clarity from an ID in the admin each, loaded only after consent, fed by the storefront's GA4 e-commerce events |
| Customers | [`zerofront/module-zero-social-login`](https://github.com/zerofrontdev/module-zero-social-login) |  | Sign in and sign up with Google, Apple or Facebook, without third-party libraries |
| Customers | [`zerofront/module-zero-returns`](https://github.com/zerofrontdev/module-zero-returns) |  | Returns (RMA): the customer asks from My Account, the store approves, receives and refunds in Sales > Returns, with messages and emails |
| Payments | [`zerofront/module-zero-paypal`](https://github.com/zerofrontdev/module-zero-paypal) | ✓ | PayPal (Standard and Express) for Zero Checkout, without JavaScript on the shop's pages |
| Payments | [`zerofront/module-zero-stripe`](https://github.com/zerofrontdev/module-zero-stripe) | ✓ | Stripe cards and wallets (Payment Element) for Zero Checkout |
| Payments | [`zerofront/module-zero-braintree`](https://github.com/zerofrontdev/module-zero-braintree) | ✓ | Magento's Braintree payment (cards, PayPal, Apple Pay, Google Pay) inside Zero Checkout |
| Payments | [`zerofront/module-zero-authorize-net`](https://github.com/zerofrontdev/module-zero-authorize-net) | ✓ | Authorize.Net cards (Accept.js) for Zero Checkout |
| Payments | [`zerofront/module-zero-amazon-pay`](https://github.com/zerofrontdev/module-zero-amazon-pay) | ✓ | Amazon Pay for Zero Checkout: the customer pays on Amazon's page after placing the order, no Amazon script on the shop's pages |
| Payments | [`zerofront/module-zero-adyen`](https://github.com/zerofrontdev/module-zero-adyen) | ✓ | Payment on Adyen's hosted page for Zero Checkout: cards, wallets and every local method switched on in the Adyen account; HMAC-signed webhooks |
| Payments | [`zerofront/module-zero-klarna`](https://github.com/zerofrontdev/module-zero-klarna) | ✓ | Klarna on its Hosted Payment Page for Zero Checkout: pay now, pay later or in instalments; authorised at checkout, captured on invoice |
| Payments | [`zerofront/module-zero-mollie`](https://github.com/zerofrontdev/module-zero-mollie) | ✓ | Payment on Mollie's page for Zero Checkout: iDEAL, Bancontact, cards, SEPA, Klarna and every method switched on for the order's currency |
| Operations | [`zerofront/module-zero-mail`](https://github.com/zerofrontdev/module-zero-mail) | ✓ | Email transport (Magento's own or Amazon SES), sent now or through a queue (RabbitMQ or the database), an email log with Resend, one sender, a test email |
| Operations | [`zerofront/module-zero-redirects`](https://github.com/zerofrontdev/module-zero-redirects) |  | A log of the storefront's 404s, most asked for first, and a 301 redirect for each in one click, with a suggested target from the catalog |

## For developers

- `docs/build.md`: the CSS and JavaScript build: the commands, the files, when to rebuild, the deploy.
- `docs/extensions.md`: how a module adds templates, styles and HTMX fragments.
- `docs/child-themes.md`: your own theme on top of Zero.
- `docs/components.md`: the UI components (dialogs, menus, forms) and the JS rule:
  HTML and CSS first, then HTMX, JavaScript only when nothing else can do it.

## License

OSL-3.0 or AFL-3.0, your choice, the same as Magento Open Source. Each package
has the full text and a `NOTICE.md`.

---

<a href="https://vilmate.com"><picture>
  <source media="(prefers-color-scheme: dark)" srcset="https://raw.githubusercontent.com/zerofrontdev/zero/main/docs/images/vilmate-logo-dark.svg">
  <img src="https://raw.githubusercontent.com/zerofrontdev/zero/main/docs/images/vilmate-logo.svg" alt="Vilmate" height="34" align="right">
</picture></a>

Zero is made by [Fixstacks](https://fixstacks.com) in partnership with [Vilmate](https://vilmate.com), a software development company.
