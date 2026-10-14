# Zero for Magento Open Source

A storefront for Magento Open Source built on the web platform: server-rendered
HTML, HTMX for the parts that refresh, Tailwind CSS v4 and daisyUI. No
RequireJS, jQuery or Knockout on the storefront. Strict CSP, WCAG 2.2 AA,
a one-page checkout without Knockout, and the modules a store needs.

- Demo store: https://zero.fixstacks.com
- Documentation: https://fixstacks.com/zero/docs (the same pages are in `docs/` here)
- Site: https://fixstacks.com/zero

## Install everything

Requires Magento Open Source 2.4.7–2.4.9 and PHP 8.2–8.4.

```bash
composer require fixstacks/zero
bin/magento setup:upgrade
bin/magento setup:di:compile           # production mode
bin/magento setup:static-content:deploy # production mode
bin/magento cache:flush
```

Then choose the theme: Content > Design > Configuration > your store view >
**Zero**. Every module is on by default except the ones that need an account
with someone (payments, analytics, social login, SES mail): those wait under
Stores > Configuration > **Zero** until you fill in their keys.

## Or pick packages

Each package installs on its own with `composer require`; the theme needs
`module-zero-core` and `module-zero-checkout` and brings them along.

| | Package | What it does |
| --- | --- | --- |
| Storefront | [`fixstacks/theme-frontend-zero`](https://github.com/Fixstacks/theme-frontend-zero) | The theme: templates, Tailwind/daisyUI styles, htmx |
| Storefront | [`fixstacks/module-zero-core`](https://github.com/Fixstacks/module-zero-core) | Storefront core for the Zero theme (HTMX fragments for private content, view models, no Knockout) |
| Storefront | [`fixstacks/module-zero-checkout`](https://github.com/Fixstacks/module-zero-checkout) | A one-page checkout without Knockout: server-rendered HTML, one native form, HTMX for the parts that refresh |
| Storefront | [`fixstacks/module-zero-instant-navigation`](https://github.com/Fixstacks/module-zero-instant-navigation) | The next page loads while the shopper still points at its link (Speculation Rules API) |
| Content | [`fixstacks/module-zero-banner`](https://github.com/Fixstacks/module-zero-banner) | Banners and sliders managed in the admin, rendered without JavaScript (CSS scroll snap) |
| Content | [`fixstacks/module-zero-cms`](https://github.com/Fixstacks/module-zero-cms) | CMS pages and blocks built from typed components in a visual editor, with AI help; no page builder markup |
| Content | [`fixstacks/module-zero-blog`](https://github.com/Fixstacks/module-zero-blog) | A blog: posts written in the Zero CMS editor, categories, tags, authors, products in posts, comments |
| Catalog | [`fixstacks/module-zero-search`](https://github.com/Fixstacks/module-zero-search) | Store search that forgives typos and finds categories, pages and posts too |
| Catalog | [`fixstacks/module-zero-layered-navigation`](https://github.com/Fixstacks/module-zero-layered-navigation) | Better filters: several values of one filter at once, counts that stay right, a price range by the customer's own numbers |
| Catalog | [`fixstacks/module-zero-brands`](https://github.com/Fixstacks/module-zero-brands) | Shop by brand: a page per brand, an A–Z of brands, the brand on product pages |
| Catalog | [`fixstacks/module-zero-seo`](https://github.com/Fixstacks/module-zero-seo) | Structured data (schema.org JSON-LD), hreflang, canonical URLs and meta descriptions |
| Catalog | [`fixstacks/module-zero-reviews`](https://github.com/Fixstacks/module-zero-reviews) | Product reviews with photos, "verified purchase" and a Google product reviews feed |
| Catalog | [`fixstacks/module-zero-stock-alert`](https://github.com/Fixstacks/module-zero-stock-alert) | "Tell me when it's back" on sold-out products, for any visitor, and one email when it is |
| Selling | [`fixstacks/module-zero-promotions`](https://github.com/Fixstacks/module-zero-promotions) | More cart price rule actions: "every Nth item, the cheapest, X% off" and a free gift |
| Selling | [`fixstacks/module-zero-reward-points`](https://github.com/Fixstacks/module-zero-reward-points) | Reward points: customers earn them on paid orders and pay part of an order with them |
| Selling | [`fixstacks/module-zero-gift-card`](https://github.com/Fixstacks/module-zero-gift-card) | Gift cards: sold as a product, emailed once paid, spent at checkout like money |
| Selling | [`fixstacks/module-zero-quote-request`](https://github.com/Fixstacks/module-zero-quote-request) | Request a quote from the cart or a product, and "price on request" products |
| Selling | [`fixstacks/module-zero-abandoned-cart`](https://github.com/Fixstacks/module-zero-abandoned-cart) | One reminder email for a cart left with items in it |
| Marketing | [`fixstacks/module-zero-product-feed`](https://github.com/Fixstacks/module-zero-product-feed) | The catalog as XML feeds for Google Merchant Center, Hotline, Rozetka and Prom.ua, one per store view |
| Marketing | [`fixstacks/module-zero-consent`](https://github.com/Fixstacks/module-zero-consent) | A cookie consent banner and settings for every tracker (GDPR), Global Privacy Control and "Do Not Sell" (CCPA) |
| Marketing | [`fixstacks/module-zero-google-analytics`](https://github.com/Fixstacks/module-zero-google-analytics) | GA4 e-commerce events with gtag.js or Google Tag Manager, with Consent Mode v2 |
| Customers | [`fixstacks/module-zero-social-login`](https://github.com/Fixstacks/module-zero-social-login) | Sign in and sign up with Google, Apple or Facebook, without third-party libraries |
| Payments | [`fixstacks/module-zero-paypal`](https://github.com/Fixstacks/module-zero-paypal) | PayPal (Standard and Express) for Zero Checkout, without JavaScript on the shop's pages |
| Payments | [`fixstacks/module-zero-stripe`](https://github.com/Fixstacks/module-zero-stripe) | Stripe cards and wallets (Payment Element) for Zero Checkout |
| Payments | [`fixstacks/module-zero-braintree`](https://github.com/Fixstacks/module-zero-braintree) | Magento's Braintree payment (cards, PayPal, Apple Pay, Google Pay) inside Zero Checkout |
| Payments | [`fixstacks/module-zero-authorize-net`](https://github.com/Fixstacks/module-zero-authorize-net) | Authorize.Net cards (Accept.js) for Zero Checkout |
| Operations | [`fixstacks/module-zero-mail`](https://github.com/Fixstacks/module-zero-mail) | Email transport (Magento's own or Amazon SES), an email log with Resend, one sender, a test email |

## For developers

- `docs/extensions.md`: how a module adds templates, styles and HTMX fragments.
- `docs/child-themes.md`: your own theme on top of Zero.
- `docs/components.md`: the UI components (dialogs, menus, forms) and the JS rule:
  HTML and CSS first, then HTMX, JavaScript only when nothing else can do it.

## License

OSL-3.0 or AFL-3.0, your choice, the same as Magento Open Source. Each package
has the full text and a `NOTICE.md`.

---

<a href="https://vilmate.com"><picture>
  <source media="(prefers-color-scheme: dark)" srcset="https://raw.githubusercontent.com/Fixstacks/zero/main/docs/images/vilmate-logo-dark.svg">
  <img src="https://raw.githubusercontent.com/Fixstacks/zero/main/docs/images/vilmate-logo.svg" alt="Vilmate" height="34" align="right">
</picture></a>

Zero is made by [Fixstacks](https://fixstacks.com/zero) in partnership with [Vilmate](https://vilmate.com), a software development company.
