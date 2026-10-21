# Building the CSS and JavaScript

Zero has no bundler. One Tailwind CLI run makes the stylesheet; JavaScript
is a handful of plain files Magento serves as they are. This page is the
whole pipeline: what goes in, what comes out, when to rerun it, and how the
deploy does it.

## In one picture

```
src/styles/app.css ─┐
  base.css          │  @import "tailwindcss"          tailwindcss CLI
  components.css    ├─ @import generated/modules.css ─────────────────▶ web/css/app.css   (minified, one file for the whole storefront)
  utilities.css     │       ▲
                    │       │ written by scripts/collect-module-styles.mjs
<module>/view/frontend/zero/tailwind.css  (one per enabled module: @source lines + own rules)

node_modules/htmx.org/dist/htmx.min.js ── scripts/copy-htmx.mjs ──▶ web/js/htmx.min.js

web/js/zero.js, web/js/product/*.js, web/js/cancel-order.js   (hand-written, served as is)
```

Magento then publishes `web/` to `pub/static/` (`setup:static-content:deploy`
in production; in developer mode it links or copies files on demand).

## Commands

All of them run in the theme folder, `app/design/frontend/Zero/default`
(or `vendor/zerofront/theme-frontend-zero` in a Composer install), and need
Node 20+ and `npm ci` once.

| Command | What it does | When |
| --- | --- | --- |
| `npm run build` | `build:css` + `build:htmx` | after a pull, after adding or toggling a module with styles |
| `npm run build:css` | `collect-module-styles.mjs`, then Tailwind: `src/styles/app.css` → `web/css/app.css --minify` | after any template or style change |
| `npm run dev:css` | Tailwind in `--watch` mode (no module collection; run `build:css` once first) | while editing templates and styles |
| `npm run build:htmx` | copies the pinned htmx build from `node_modules` into `web/js/htmx.min.js` | after `npm ci`, after bumping htmx in `package.json` |
| `npm run build:dist` | `build`, then `scripts/minify-theme-css.mjs` minifies every Zero theme's hand-written `web/css/theme-overrides.css` in place | the deploy only (see below); the files in git stay readable |

The build needs no database and no PHP: it reads `app/etc/config.php` and
`vendor/composer/installed.json` as files, so it must run inside a Magento
project (the module list comes from there), but that is all.

Not in git, built on every machine: `web/css/app.css`, `web/js/htmx.min.js`,
`src/styles/generated/` (the theme's `.gitignore`).

## CSS, step by step

1. **The entry point** is `src/styles/app.css`. It imports Tailwind with
   `source(none)` and lists what Tailwind may scan for class names: the
   theme's own `.phtml` templates (`@source "../../**/*.phtml"`), a few
   inline names core markup uses (`footer progress indicator`), and the
   generated module file. Scanning the whole repository is deliberately
   off: prose and PHP in other files would turn into classes.
2. **Modules join through one file each.** A module that has storefront
   templates ships `view/frontend/zero/tailwind.css`:

   ```css
   /* Built into the Zero theme's CSS while Zero_Foo is enabled. */
   @source "../templates/**/*.phtml";   /* Tailwind scans the module's templates */

   @layer components {                   /* the module's own rules, if any */
       .foo-panel { ... }
   }
   ```

   `scripts/collect-module-styles.mjs` finds every such file among the
   modules that are **enabled in `app/etc/config.php`**, in the module load
   order, and writes `src/styles/generated/modules.css` as one `@import`
   per module. Modules are found the way Magento finds them, from
   `app/code/*/*/registration.php` and the Composer packages in
   `vendor/composer/installed.json`. A disabled module drops out of the
   build; a module without the file adds nothing. Details and the compat
   module pattern: [extensions.md](extensions.md).
3. **Tailwind v4 + daisyUI** compile it all. `app.css` also declares the two
   daisyUI themes (`zero`, `zero-dark`), the design tokens (`@theme`), and
   imports the theme's own layers: `base.css`, `components.css`,
   `utilities.css`. Output: `web/css/app.css`, minified, the only stylesheet
   the storefront loads (`Magento_Theme/layout/default_head_blocks.xml`
   removes stock's `styles-m.css`/`styles-l.css` and adds `css/app.css`).
4. **Child themes** do not rebuild anything: they load the parent's `app.css`
   and add `web/css/theme-overrides.css` with token overrides and fonts. It
   is served as written; `build:dist` minifies it for the deploy. See
   [child-themes.md](child-themes.md).

The stock checkout fallback (`Zero_Core`'s `checkout_index_index.xml`) is
the one page that still loads the compiled Luma CSS from `web/css/vendor/`;
nothing builds those files.

### Rules of the CSS

- Put a class on the markup first; write a rule only for what utilities
  cannot say. Module rules go in `@layer components` so utilities in the
  template still win.
- Names must not collide with daisyUI component names (`.menu`, `.badge`,
  `.progress`, `.megamenu`, ...): daisyUI's rule comes later in the cascade.
  Prefix module classes (`.free-shipping-bar`, `.mega-panel`).
- No inline `style=""`, no `--var` set in a style attribute: the storefront
  runs under a strict Content Security Policy (`style-src` without
  `unsafe-inline`). Vary with classes or data attributes.
- Tailwind only emits classes it sees in a scanned file. A class built from
  PHP strings (`'col-' . $n`) is invisible: write the full names out, or add
  them with `@source inline(...)`.

## JavaScript

There is no JS build. The rule (see [components.md](components.md)): HTML
and CSS first, HTMX for anything that refreshes, JavaScript only when the
platform has nothing else, and then as a small plain file, never inline.

| File | Loaded | Purpose |
| --- | --- | --- |
| `web/js/htmx.min.js` | every page, `default_head_blocks.xml` | htmx, copied from the npm package by `build:htmx`; pinned in `package.json` |
| `web/js/zero.js` | every page, `defer` | sitewide behaviours wired by data attributes (`data-qty-step`, `data-enter-clicks`, ...); listeners delegated from `document`, so markup htmx swaps in later works as is |
| `web/js/product/gallery.js`, `swatches.js`, `custom-options.js`, `bundle-summary.js` | by the template that needs them, `<script src=... defer>` | product page pieces the platform cannot do alone |
| `web/js/cancel-order.js` | the cancel-order dialog | one dialog's behaviour |
| `Zero_Core/view/frontend/web/js/session.js`, `ui-library.js`, `customer-data-mixin.js` | `Zero_Core`'s layouts | session bootstrap (private fragments, form key), the `/zero/ui/` library, the customer-data shim |
| `Zero_Core/view/frontend/web/js/vendor/require.js` | the stock checkout fallback only | RequireJS, removed sitewide and re-added on that one page |

A module that truly needs a script ships it under its own
`view/frontend/web/js/` and loads it from its template with
`$block->getViewFileUrl('Zero_Foo::js/thing.js')` and `defer`. Payment SDKs
(Stripe, Braintree, ...) are loaded the same way and allowed in the CSP
whitelist of the module (`etc/csp_whitelist.xml`).

Magento's own JS minification (`dev/js/minify_files`) applies at
`static-content:deploy` time if the store turns it on; the theme does not
depend on it.

## When to rebuild

| You changed | Run |
| --- | --- |
| a `.phtml` template (new utility classes) | `npm run build:css` |
| a rule in `src/styles/*.css` or a module's `tailwind.css` | `npm run build:css` (or keep `dev:css` watching) |
| enabled or disabled a module, installed a package with styles | `npm run build:css` (the module list is read on each build) |
| `theme-overrides.css` of a child theme | nothing locally; the deploy minifies it |
| a JS file | nothing; in production, `setup:static-content:deploy` |
| htmx version in `package.json` | `npm install && npm run build:htmx` |

After a rebuild in developer mode the browser gets the new file at once
(static files are served from the source). The page itself may still come
from the full-page cache: clear it (`bin/magento cache:clean full_page`,
or locally `rm -rf var/cache var/page_cache`, the file cache's tag index
drops entries on 2.4.9).

## The deploy

`buildspec.yml` (CodeBuild) runs, after Composer:

```bash
cd app/design/frontend/Zero/default && npm ci && npm run build:dist
```

The artifact carries the built `app.css`, `htmx.min.js` and minified
overrides; `node_modules` is stripped. On the server
`scripts/after_install.sh` removes `pub/static/frontend` and
`var/view_preprocessed` and runs `setup:static-content:deploy -f`: with
the folders in place the deploy had kept day-old files while reporting
success, so they go first.

The public Composer packages work the other way round: `zerofront/theme-
frontend-zero` ships `web/css/app.css` already built (the export runs the
build before packing), so `composer require` gives a working store with no
Node at all. A store that adds, enables or disables a module with styles
rebuilds as above, inside its own project.

## Troubleshooting

| Symptom | Cause | Fix |
| --- | --- | --- |
| A class in your template has no effect | Tailwind never saw it: the module has no `tailwind.css`, is disabled in `config.php`, or the build did not rerun | add the file / enable the module, `npm run build:css` |
| `generated/modules.css` missing, Tailwind errors on `@import` | `build:css` was never run on this machine | `npm run build:css` (never `dev:css` alone on a fresh clone) |
| Your rule is there but another wins | a daisyUI component of the same name, or a utility in the markup | rename the class with a module prefix; keep rules in `@layer components` |
| `htmx.min.js` 404 | `npm run build:htmx` not run after `npm ci` | `npm run build` |
| Styles right locally, stale on production | `pub/static` kept old files | `after_install.sh` already removes them; by hand: `rm -rf pub/static/frontend var/view_preprocessed && bin/magento setup:static-content:deploy -f` |
| A style attribute is ignored and the console shows a CSP report | inline styles are blocked by design | move it to a class or a data attribute |
