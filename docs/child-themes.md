# Child themes for Zero

Zero/default is built so that most rebranding is a list of CSS variables. A
child theme inherits every template, layout and asset from Zero/default and
replaces only what it names. A complete, working example ships next to the
parent: `app/design/frontend/Zero/example` (see its README). Four finished
looks for different kinds of business, each with a demo store
(Zero_DemoStores): `Zero/noir` (luxury), `Zero/pulse` (electronics),
`Zero/bloom` (beauty) and `Zero/forge` (tools and trade). Making third-party
extensions work on Zero is covered in [extensions.md](extensions.md); the reusable
components are in [components.md](components.md) and live at `/zero/ui/`.

There are two levels of customization. Most stores only need the first.

| You want to change… | How | Build step? |
| --- | --- | --- |
| Colors, fonts, type sizes, corners, border width | Redefine tokens in `web/css/theme-overrides.css` | No |
| Markup, using classes Zero already uses | Override the template / layout file | No |
| Markup with Tailwind or DaisyUI classes Zero doesn't use | Override the template, then build the child's own `app.css` | Yes |

## 1. Create the theme

Copy `Zero/example` and rename it, e.g. to `app/design/frontend/Acme/store`:

1. `registration.php`: `'frontend/Acme/store'`.
2. `theme.xml`: your `<title>`; keep `<parent>Zero/default</parent>`.
3. `package.json` (only for level 3): your package name.
4. Register and select it:

   ```bash
   bin/magento setup:upgrade
   ```

   Content → Design → Configuration → (store view) → Applied Theme (set it
   there, not with `bin/magento config:set`: design settings saved from the
   CLI don't reach the store view). If
   `app/etc/config.php` pins `design/theme/theme_id` (this project does, for
   the default scope), set the theme on the store view, or change the pinned
   value there.

   If your deployment commits `app/etc/config.php`, add the theme to its
   `themes` section (copy the `frontend/Zero/example` entry) so config imports
   don't treat it as unknown; deploy scripts must run
   `setup:upgrade --no-interaction` (a changed theme list asks for
   confirmation).

## 2. Restyle with tokens (no build)

The child's `Magento_Theme/layout/default_head_blocks.xml` loads
`css/theme-overrides.css` after the parent's `app.css`. In it, set any tokens
on `:root`:

```css
:root {
    --color-primary: #2747d6;
    --color-primary-content: #ffffff;
    --radius-field: 0.5rem;
    --font-sans: ui-sans-serif, system-ui, sans-serif;
    --text-page-title: 2.25rem;
}
```

A plain `:root` rule wins without `!important`: DaisyUI defines the colors on
`:where(:root)` (zero specificity) and Tailwind puts fonts and sizes in
`@layer theme`, which unlayered rules beat.

### Colors (DaisyUI theme `zero`)

Each fill color has a `*-content` partner: the text color used on top of it.
Keep the pair readable (contrast ≥ 4.5:1 for text).

| Token | Zero/default | Used for |
| --- | --- | --- |
| `--color-primary` / `-content` | `#d6ff4b` / `#151515` | Main CTAs (Place Order, Proceed to Checkout), "Sale" badge, active filters, footer band |
| `--color-secondary` / `-content` | `#6f7d3a` / `#f7f6f1` | Italic accent words in headings |
| `--color-accent` / `-content` | `#95a655` / `#151515` | Small highlights |
| `--color-neutral` / `-content` | `#151515` / `#f0efe9` | Add to Cart and other dark buttons, "New" badge, dark bands, demo-store bar |
| `--color-base-100` | `#f7f6f1` | Page background, cards |
| `--color-base-200` | `#eeede8` | Subtle panels, hover fills, current menu item |
| `--color-base-300` | `#e4e5dc` | Borders and dividers |
| `--color-base-content` | `#000000` | Body text (muted text mixes this with transparency) |
| `--color-info` / `success` / `warning` / `error` (+ `-content`) | muted blue / green / ochre / red | Messages and validation |

A dark variant, `zero-dark`, is defined too; it applies to an element with
`data-theme="zero-dark"`.

### Shape

| Token | Zero/default | Used for |
| --- | --- | --- |
| `--radius-box` | `0` | Cards, panels, dropdowns |
| `--radius-field` | `0` | Buttons, inputs, selects |
| `--radius-selector` | `0` | Checkboxes, radios, badges |
| `--border` | `1px` | Border width of DaisyUI components |

### Fonts

| Token | Zero/default |
| --- | --- |
| `--font-sans` | DM Sans (self-hosted, variable 100–1000), then `DM Sans Fallback` — all text |
| `--font-display` | Instrument Serif italic, then `Instrument Serif Fallback` — only `.font-display` accent words |

Every Zero theme loads its fonts the same way; keep to it in a child theme,
it is what keeps the first paint clean (no text that jumps or re-renders):

1. **One family for everything, at most one accent face.** A store needs a
   body font; an accent face (headings, the hero's italic words) is optional.
   For a single family set `--font-display: var(--font-sans)`.
2. **Self-hosted, subset, variable.** `woff2` files in the child's `web/fonts/`,
   one file per script (`latin`, `latin-ext`, `cyrillic`, …) with a
   `unicode-range`, so a visitor downloads only the script the page uses; one
   variable file instead of a file per weight when the family has one. No
   Google Fonts `<link>`: it is a third origin before the first paint, and in
   the EU a privacy question. Put the license file next to the fonts (SIL OFL
   requires it).
3. **`font-display: optional` on every `@font-face`, with a metric-matched
   fallback.** The font is preloaded, so it is normally in before the CSS and
   the text paints once, in the right font. When it is late (a slow first
   visit) the page keeps the fallback for that visit and never swaps mid-read;
   the next page has the font from the cache. The fallback is the system font
   scaled to the webfont's metrics, so the text takes the same room either
   way. Generate it:

   ```sh
   pip3 install fonttools brotli   # once
   python3 dev/tools/font-fallback.py app/design/frontend/Zero/<theme>/web/fonts/<family>-latin.woff2 --match arial
   ```

   `--match` is the system font to scale: `arial` for a sans (Roboto on
   Android is close), `arial-narrow` for a condensed face, `georgia` or
   `georgia-italic` for a serif, `times`. Paste the printed `@font-face`
   after the family's own faces and put the fallback name right after the
   family in the stack: `"Space Grotesk", "Space Grotesk Fallback", system-ui, sans-serif`.
4. **Preload only what the first screen needs**: the body font's latin
   regular file and the heading face the hero uses, three files at most, in
   `Magento_Theme/layout/default_head_blocks.xml`; remove the parent's
   preloads the theme doesn't use (they compete with the LCP image):

   ```xml
   <head>
       <remove src="fonts/dm-sans-latin-400-700.woff2"/>
       <remove src="fonts/instrument-serif-italic-400.woff2"/>
       <link src="fonts/space-grotesk-latin-400-700.woff2" rel="preload" as="font" type="font/woff2" crossorigin="anonymous"/>
       <css src="css/theme-overrides.css"/>
   </head>
   ```

To replace the font in a child theme: drop the `woff2` files and the license
into `web/fonts/`, write the `@font-face` rules (with `font-display: optional`
and the `unicode-range` per file) and the generated fallback at the top of
`theme-overrides.css`, point `--font-sans` (and `--font-display`) at the new
stack, and set the preloads as above. `Zero/usa` (one family, one file) and
`Zero/forge` (two families, static cuts per weight) are the two shapes to
copy. Check with the fonts blocked (DevTools > Network > block `*.woff2`):
the page must look the same size, nothing moves.

### Type sizes

Tailwind's scale (`--text-xs` 0.75rem, `--text-sm` 0.875rem, `--text-base`
1rem, `--text-lg` 1.125rem, `--text-xl` 1.25rem, … `--text-9xl`) plus:

| Token | Zero/default | Used for |
| --- | --- | --- |
| `--text-3xs` | 0.6875rem | Stock / SKU row on the product page |
| `--text-2xs` | 0.6875rem | Eyebrow labels, badges |
| `--text-label` | 0.8125rem | Form labels, meta text, review actions |
| `--text-md` | 0.9375rem | Review body |
| `--text-card-name` | 0.9rem | Product name on cards |
| `--text-card-price` | 1.05rem | Price on cards |
| `--text-section-title` | 1.75rem | Related / up-sell headings, product Details |
| `--text-page-title` | 1.875rem | Page titles (phones) |
| `--text-page-title-md` | 2.25rem | Page titles (from 48rem) |
| `--text-product-title` | 2.5rem | Product name (phones) |
| `--text-product-title-lg` | 3.5rem | Product name (large screens) |
| `--text-product-price` | 1.875rem | Price on the product page |

Everything is in `rem`, so `html { font-size: 112.5%; }` scales the whole
storefront at once.

## 3. Override templates and layouts

Same as any Magento theme: put the file at the same path under the child,
e.g. `Magento_Theme/templates/html/header/logo.phtml` (the example does this).
Start from the parent's copy in `Zero/default` and keep its docblock notes:
they record why the markup is the way it is (no RequireJS/Knockout, HTMX
fragments, cache-safe shells).

If the new markup only uses classes already present in Zero's templates, it
is styled by the parent's `app.css` as is.

## 4. New Tailwind classes (build step)

Tailwind only emits classes it finds in the templates it scans. When a child
template uses a class Zero doesn't (say `bg-accent/30`), build the child's
own stylesheet:

```bash
cd app/design/frontend/Zero/default && npm ci   # once: the toolchain lives in the parent
cd ../example && npm run build:css             # writes web/css/app.css
```

`src/styles/app.css` imports the parent's entry file (tokens, plugins, all of
Zero's templates) and adds `@source "../../**/*.phtml"` for the child's. The
result, `web/css/app.css` in the child, replaces the parent's file by theme
fallback, so it is the complete stylesheet, not an addition.

`web/css/app.css` is git-ignored like the parent's: add the build to your CI
next to the parent's, after it:

```bash
(cd app/design/frontend/Zero/default && npm ci && npm run build)
(cd app/design/frontend/Acme/store && npm run build:css)
```

Then deploy static content as usual (`setup:static-content:deploy`).

## Right-to-left

`Zero/rtl` (next to this theme) is a complete right-to-left child: it sets
`dir="rtl"`, adds an Arabic font and an example Arabic translation. The
mirroring itself comes from this parent theme, which uses logical directions
only. Keep it that way in parent templates and CSS, and in child themes that
should work both ways:

| Use | Not |
| --- | --- |
| `ms-` `me-` `ps-` `pe-` `start-` `end-` `text-start` `text-end` `border-s` `rounded-s` | `ml-` `mr-` `pl-` `pr-` `left-` `right-` `text-left` `text-right` `border-l` `rounded-l` |
| `margin-inline-start`, `inset-inline-end`, `text-align: start`, `float: inline-start` | `margin-left`, `right`, `text-align: left`, `float: left` |
| `rtl:-scale-x-100` on a directional icon (arrows) | a hard-coded arrow direction |
| `getComputedStyle(el).direction` before `scrollBy({ left })` / reading `scrollLeft` (negative in RTL) | assuming scrollLeft grows to the right |

The mobile e2e suite checks key pages with `dir="rtl"` set on this theme, so a
physical direction that breaks the mirrored layout fails CI.

## Widgets are per theme

Widget instances (Content → Widgets) belong to one theme. The homepage hero
(a "Zero Banner" widget) and any other widget placed for Zero/default don't
appear under the child theme: add them again with the child selected as
Design Theme, or change their theme. Banners and slides themselves (Content →
Banners) are shared; only the placement is per theme.

## What not to override

- `Zero_Core::session/bootstrap.phtml` and the `hx-*` attributes on shells
  (account menu, mini-cart, compare/wish list sidebars, recently viewed): they
  load per-visitor content into full-page-cached pages.
- Form `form_key` inputs and the `zero:session-ready` triggers.
- Payment form containers in checkout (`zero.checkout.payment.*`): payment
  modules plug into them.
