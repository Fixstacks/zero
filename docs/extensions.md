# Extensions on Zero

Zero pages don't load RequireJS, jQuery widgets or Knockout. Server-rendered
markup and HTMX do the work. An extension whose storefront relies on Luma's
JavaScript stack (`x-magento-init`, `data-mage-init`, UI components) renders
on Zero but its interactive parts do nothing. Its Tailwind classes are also
missing until the theme build knows about its templates.

There are three ways to make an extension work. Pick one per extension.

| Situation | What to do |
| --- | --- |
| You write the extension, or it is small | Support Zero in the extension itself: add `view/frontend/zero/tailwind.css` (§1); JavaScript and Zero_Core's building blocks in §4 |
| A third-party extension, worth adapting | A **compat module** with Zero versions of its templates (§2) |
| Not adapted yet, or rarely used pages | Render its pages with Luma: **Pages on Another Theme** (§3) |

Start with the report. It lists every enabled module with storefront
templates or layouts and how each one works on Zero:

```bash
bin/magento zero:compat:report        # third-party modules
bin/magento zero:compat:report --all  # Magento's own modules too
```

```
+------------------+------------------+---------+---------------------------------+
| Module           | Storefront files | Luma JS | Works on Zero via               |
+------------------+------------------+---------+---------------------------------+
| Acme_Rma         | 14               | 6       | Luma pages: rma/*               |
| Acme_Faq         | 3                | 0       | compat: ZeroCompat_AcmeFaq      |
| Acme_Banner      | 5                | 2       | not covered                     |
+------------------+------------------+---------+---------------------------------+
```

"Luma JS" counts the files that use RequireJS or Knockout. Those parts need
rewriting, not just restyling.

## 1. Styles: any module can add to the theme CSS

Tailwind generates only the classes it finds in the files it scans. The
theme build scans the theme's own templates plus every **enabled** module
that ships `view/frontend/zero/tailwind.css`:

```css
/* view/frontend/zero/tailwind.css */
@source "../templates/**/*.phtml";

/* Optional: the module's own styles, with the theme's tokens. */
@layer components {
    .acme-faq-answer { padding-block: --spacing(3); }
}
```

`scripts/collect-module-styles.mjs` runs before each `npm run build:css`.
It reads the enabled modules from `app/etc/config.php`, finds them in
`app/code` and `vendor/` the way Magento does, and writes
`src/styles/generated/modules.css`. When a module is disabled, its styles
leave the next build. Zero's own modules use the same file. To see what
gets included:

```bash
cd app/design/frontend/Zero/default && npm run build:css
# Module styles: Zero_Core, Zero_Banner, Zero_Checkout, ..., ZeroCompat_AcmeFaq
```

Rebuild the theme CSS after you enable, disable or add a module. The
deployment pipeline builds it on every deploy. Modules installed with
Composer are found through `vendor/composer/installed.json`, so the build
must run where `composer install` has run. This project's CodeBuild installs
Composer packages only with `ZERO_CI_COMPOSER=1`; without it, only `app/code`
modules are included, and the script prints a warning.

## 2. Compat modules: Zero templates for someone else's module

A compat module carries Zero versions of another module's templates and
leaves the original untouched. On a Zero theme (Zero/default or any theme
that inherits from it), a template resolves in this order:

1. the theme's own `Vendor_Module/templates/...` (themes always win);
2. the compat module's `view/frontend/templates/...`;
3. the original module's template.

A template the compat module doesn't have comes from the original, so the
compat module holds only what it changes. On Luma and other themes nothing
changes. Emails rendered for a Zero store view use the compat templates too.

Create one with the scaffold command. `--template` copies an original
template to start from; repeat it for each template:

```bash
bin/magento zero:compat:create Acme_Faq --template=faq/list.phtml
bin/magento module:enable ZeroCompat_AcmeFaq
bin/magento setup:upgrade
(cd app/design/frontend/Zero/default && npm run build:css)
```

This creates `app/code/ZeroCompat/AcmeFaq` (use `--name=` for another name):

```
registration.php
etc/module.xml                      loads after Acme_Faq and Zero_Core
etc/di.xml                          registers it for Acme_Faq (below)
view/frontend/zero/tailwind.css     its templates in the theme build
view/frontend/templates/faq/list.phtml   the copy to rewrite
view/frontend/layout/               layout changes, merged after Acme_Faq
README.md
```

The registration is a DI argument. Any module can declare it, and a module
can have several compat modules (the first one that has the template wins):

```xml
<type name="Zero\Core\Model\Compat\Registry">
    <arguments>
        <argument name="compatModules" xsi:type="array">
            <item name="Acme_Faq" xsi:type="array">
                <item name="ZeroCompat_AcmeFaq" xsi:type="string">ZeroCompat_AcmeFaq</item>
            </item>
        </argument>
    </arguments>
</type>
```

Rewriting a template for Zero usually means:

- **Markup**: Tailwind and DaisyUI classes, the same components the theme
  uses (`btn`, `input`, `card`; see the theme's templates for patterns).
- **JavaScript**: replace `x-magento-init` widgets with HTMX attributes
  (`hx-post`, `hx-target`) or a small static file in the compat module's
  `view/frontend/web/js/`, loaded with `<script src="..." defer>`. Pass
  data in `<script type="application/json">` or `data-` attributes, never
  as inline code: Zero runs under a strict Content Security Policy (no
  inline scripts, no `eval`, no `onclick=""`), enforced on Zero Checkout.
  Third-party hosts a module loads from go in its `etc/csp_whitelist.xml`.
- **Knockout UI components** (`jsLayout`, `.html` templates): render the
  markup in PHP instead. A view model gives the template its data.
- **Layout**: in the compat module's `view/frontend/layout/`, remove what
  doesn't apply on Zero (`<referenceBlock name="..." remove="true"/>`) or
  point blocks at new templates.

## 3. Pages on Another Theme

For an extension that isn't adapted yet, its pages can render with another
theme, typically Luma. They keep working with their own JavaScript, and
the rest of the store stays on Zero.

**Stores → Configuration → Zero → Theme → Pages on Another Theme**:

- **Pages**: one per line, `*` as wildcard. Each line is matched against
  `front name/controller/action` and the URL path:

  ```
  # Acme RMA: not adapted yet
  rma/*
  amasty_rma/guest/*
  blog/*
  ```

- **Theme for These Pages**: Magento/luma by default.

Both settings can differ per website or store view. The theme switches
while Magento loads the design, before layout and translations, so the page
is a complete Luma page with Luma's header and footer.

Keep in mind:

- The fallback theme's static files must be deployed. The deploy script's
  `setup:static-content:deploy` deploys all registered themes, Luma
  included.
- Translations come from the fallback theme. A Zero child theme's own
  `i18n/*.csv` does not apply on these pages.
- Blocks rendered on every page (header, mini cart, footer) come from the
  fallback theme on these pages. So a block an extension adds to the header
  shows in its Luma style there and needs a compat template elsewhere.

## 4. JavaScript and what Zero_Core gives a module

**JavaScript.** HTML and CSS first, HTMX second, a script only where neither
can do it (a payment SDK, a gallery zoom). A script is a static file in the
module's `view/frontend/web/js/`, loaded from the template with
`<script src="<?= $block->getViewFileUrl('Acme_Faq::js/faq.js') ?>" defer>`
and wired by `data-` attributes, never by inline code or `onclick=""`:
the storefront's Content Security Policy allows no inline scripts and no
`eval`. Data goes in `data-` attributes or `<script type="application/json">`.
Hosts a script talks to go in the module's `etc/csp_whitelist.xml`. The
theme's `web/js/zero.js` shows the pattern (delegated listeners, so markup
HTMX swaps in later works too).

**Per-visitor content on cached pages.** Every page is served from the page
cache, so anything that depends on the visitor is a fragment: a controller
implementing `Zero\Core\Controller\PrivateFragmentActionInterface` (never
cached, `Cache-Control: no-store`) rendered through its own layout handle,
loaded by `hx-get`. Trigger it with `hx-trigger="zero:session-ready from:body"`
so it runs once the visitor's session exists; an element with
`data-zero-fragment="name"` is filled by the session request itself
(`Zero\Core\Controller\Session\Bootstrap::FRAGMENTS`). Forms get the
visitor's `form_key` from `session.js`; a cached `form_key` in the HTML is
fine. Events the theme fires: `zero:session-ready`, `cart:updated`,
`wishlist:updated`, `zero:consent`.

**Building blocks** (see each class's docblock):

| Need | Use |
| --- | --- |
| A CAPTCHA on a public form of a cached page | `Zero\Core\Block\Captcha\Field` as the form's `captcha` child (layout), `Zero\Core\Model\Captcha\Check::verify()` in the controller, the form id under `customer/captcha/frontend/areas` in `etc/config.xml` |
| A store view at a path of the main domain (`/ua/`, `/ua/en/`), even from another website | `paths` argument of `Zero\Core\Plugin\Store\StorePathPrefix` in `etc/di.xml` |
| Whole amounts for a currency (`4 800 ₴`) | `currencies` argument of `Zero\Core\Plugin\Directory\WholeAmounts` in `etc/frontend/di.xml` |
| Product cards in a list of your own | `Magento_Catalog::product/list.phtml` with its `card_cache` argument (`Zero\Core\ViewModel\ProductCardCache`): one block cache entry per card |
| Responsive product images (`<picture>`, WebP) | `Zero\Core\Model\Image\ProductPicture::getSrcsets()`; `bin/magento zero:images:warm` after an import |
| Prices outside a price box (badges, analytics) | `Zero\Core\Model\Pricing\CachedPrices::get()` |
| A storefront tracker that respects cookie consent | `Zero\Consent\Model\Trackers` (di.xml) and `window.zeroConsent.get()` |
| Sending email, SES, the Email Log | nothing to do: `Zero_Mail` wraps Magento's transport |

## Ready-made compat modules

Composer packages in `packages/zero-compat/`, installed next to the
extension they adapt:

```bash
composer config repositories.zero-compat path "packages/zero-compat/*"
composer require zero/compat-<name>:@dev
bin/magento setup:upgrade
```

| Package | Adapts | What changes |
| --- | --- | --- |
| `zero/compat-magepal-googletagmanager` | [MagePal Google Tag Manager](https://github.com/magepal/magento2-google-tag-manager) | Data layer and consent rules without RequireJS or inline scripts; session data from `customer/section/load`; CSP whitelist for GTM/GA. e2e: `compat-magepal-gtm.spec.ts` |

Each package's README lists what it covers and what it can't.

## Checklist for one extension

1. `bin/magento zero:compat:report`: is it covered, and how much Luma JS
   does it use?
2. Quick win: list its pages under Pages on Another Theme.
3. Adapt it properly: `zero:compat:create`, rewrite the templates, enable,
   `setup:upgrade`, `npm run build:css`.
4. Remove its lines from Pages on Another Theme once the compat module
   covers them.
