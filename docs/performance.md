# Performance: keeping the pages fast

The Zero demo scores 95–99 on PageSpeed Insights (mobile) with 29 KB of
JavaScript per page. That is not a property of the theme that survives on
its own: a carousel that autoplays, a font that swaps, a chat widget pasted
into a CMS block, each takes points back. This page says what the score
rests on, what to keep to when adding a module, a theme or content, and how
to check before a deploy.

## What the numbers measure

| Metric | What moves it on a Zero store |
| --- | --- |
| First Contentful Paint (FCP) | Time to the first text or image: the HTML's time to first byte plus the one CSS file. |
| Largest Contentful Paint (LCP) | The hero image or the product photo: its size, its format, whether it is discovered early. |
| Speed Index (SI) | How fast the screen stops changing. Anything that repaints after the first paint — a slide change, a font swap, a banner sliding in — pushes it up, even when FCP is fine. |
| Total Blocking Time (TBT) | JavaScript on the main thread. Zero's own scripts are small; third-party tags are what blocks. |
| Cumulative Layout Shift (CLS) | Content that moves: an image without a size, a font whose fallback is wider, a bar inserted above content. |

Google scores from a simulated mid-range phone on slow 4G, with a 4× slower
CPU. A change invisible on a laptop costs seconds there.

## What the score rests on

1. **Magento renders, the cache serves.** Pages are cacheable; the
   per-visitor pieces (cart counter, account links, messages) arrive in one
   request after the page. A block that reads the session or the customer
   makes the whole page uncacheable: that belongs in a private fragment
   (`Zero\Core\Controller\PrivateFragmentActionInterface`, see extensions.md).
2. **One stylesheet, render-blocking on purpose.** `app.css` is ~40 KB
   minified, built from the theme and every enabled module (build.md). It is
   not preloaded and swapped in later: measured, that gave CLS 1 on mobile.
   Child themes add one more file, `theme-overrides.css`, no build.
3. **Little JavaScript, all deferred, none inline.** HTMX (16 KB) plus small
   per-template files, every `<script>` with `defer`, settings in
   `<script type="application/json">`. The one exception is `consent.js`,
   loaded in the head without `defer` so the cookie banner's visibility is
   settled by the first paint. No RequireJS, jQuery or Knockout anywhere.
4. **Fonts that never swap.** Self-hosted subset files, `font-display:
   optional`, a metric-matched fallback per family, preloads of only the
   first-screen files. The whole standard is in child-themes.md, Fonts.
5. **Images sized and in WebP.** Every product and banner image has a width
   and height (or an aspect ratio) in the markup, so the page is laid out
   before the images load; `srcset` and `sizes` for the screen; WebP from
   the image module; lazy loading below the fold only.
6. **Nothing changes after the first paint.** The cookie banner is in the
   first paint (not slid in after load). The sticky Add to Cart bar is
   hidden until the visitor scrolls past the form. Carousels do not autoplay
   until the visitor touches the page. No fade-in-on-load animations, no
   content inserted above what is already shown.
7. **Third parties only after consent, and only through Zero_Consent.**
   Google Analytics, the marketing pixels and anything else load when the
   visitor has accepted, from a static file, never from a tag pasted into the
   head. A tag manager is the single biggest way to lose 20 points.

## Budgets

The demo's home page, mobile; a store with a bigger catalog or more
modules may run a little above, but a jump means something is wrong.

| | Budget | Demo |
| --- | --- | --- |
| Page weight (home, compressed) | ≤ 400 KB | 345 KB |
| JavaScript | ≤ 40 KB | 29 KB |
| CSS | ≤ 60 KB | 40 KB |
| Requests to paint the home page | ≤ 30 | 29 |
| LCP | ≤ 2.5 s | 2.0 s |
| Speed Index | FCP + 0.5 s at most | 2.3 s |
| TBT | ≤ 50 ms | 0 ms |
| CLS | 0 | 0 |

## When you add something

**A module**

- Styles in the module's `view/frontend/zero/tailwind.css`, built into the
  theme's file (build.md); no `<style>` blocks, no second stylesheet.
- Behaviour in HTML and HTMX first; JavaScript only when it can't be done
  without, as one static file per template, loaded with `defer`, settings in
  a JSON script. Never inline (the CSP forbids it anyway).
- A block that depends on the visitor renders as a private fragment, so the
  page stays cacheable. Product cards are cached one by one: a badge or a
  button on the card must not depend on the visitor.
- Nothing fetched on load (`hx-trigger="load"`) above the fold unless the
  content truly can't be in the page. Each such fragment is a request and a
  repaint.
- Third-party scripts go through Zero_Consent's purposes (`analytics`,
  `marketing`) and load after the choice.

**A theme**

- Fonts per the standard (child-themes.md, Fonts): subset, `optional`,
  fallback, preloads trimmed.
- The hero image: one `<picture>` with WebP sources and `sizes`, not a CSS
  background; under 100 KB for the phone size. The banner module makes
  these from the uploaded picture.
- No `backdrop-filter` on `.page-header` itself (it breaks the mobile
  menu's positioning; a blurred `::before` behind it is the pattern), and no
  animation on load: a hero that fades or rises in is a repaint after the
  first paint, and an LCP image that starts transparent counts late.
- Check every page at phone width with the fonts blocked: nothing may move.

**Content (in the admin)**

- Banners: one picture per slide, uploaded once; the module resizes and
  converts it. Autoplay is fine: it starts on the first touch.
- CMS pages and blocks: no pasted `<script>` or `<iframe>` widgets (chat,
  reviews, maps) above the fold. A map or a video goes behind a click, as
  the store locator's map and the blog's embeds do.
- Product images: upload the large original; the image module makes the
  sizes. Don't paste images into descriptions by URL from another site.

## How to check, before a deploy

1. **PageSpeed Insights** on the phone for the home page, a category and a
   product: `https://pagespeed.web.dev/`. Mobile is the number that matters.
2. **Lighthouse locally with real throttling**, for the filmstrip:

   ```sh
   npx lighthouse https://zero.fixstacks.com/ --only-categories=performance --throttling-method=devtools --view
   ```

   In the report open "View Treemap" for the JavaScript and the filmstrip
   under Speed Index: after the frame with the LCP image, no frame may
   differ. If one does, something repaints late: find it before deploying.
3. **The visual audit** (`ZERO_E2E_AUDIT=1 npx playwright test visual-audit`
   in `dev/tests/zero-e2e`) for layout shifts, elements outside the screen
   and tap targets at four sizes; **the e2e suite** for the rest.

## What PageSpeed Insights cannot tell you

Lighthouse (and so PageSpeed Insights) loads the page in a headless tab whose
frames are never shown. About 0.43 s after navigation Chrome's compositor
sends a burst of empty frames for the still-empty document; the display
never draws them, and from then on the renderer gets one frame opportunity
per second (viz throttles clients with undrawn frames). A page whose first
content frame is ready before that burst paints at ~0.45 s; one that is
still waiting for its stylesheet paints at the next opportunity, ~1.5 s, and
at ~2.5 s if that first frame cannot be laid out, painted, committed and
rasterised within one vsync. Lighthouse's simulated Speed Index is 1.4 × the
observed one, so these steps show up as Speed Index ≈ 2.3 s or ≈ 4.2 s with
FCP, LCP, TBT and CLS unchanged.

This is a property of the measuring tab, not of the page: the same Chrome
driven by Playwright paints the Zero home page at 0.5–0.6 s. Every site is
measured this way; sites with inline critical CSS pay it in Speed Index
rather than FCP. Before changing the theme for a Speed Index jump, read
`observedFirstPaint` in the Lighthouse JSON (`--output=json`): a value of
~1.5 s or ~2.5 s with `load` at ~0.6 s is this artefact. Real-user data
(CrUX, or WebPageTest on a real device) is the number to trust for Speed
Index; the rest of the PSI report stays meaningful.

## When a number drops

| Symptom | Look at |
| --- | --- |
| Speed Index up, FCP unchanged | First `observedFirstPaint` in the Lighthouse JSON (see above). If it is not ~1.5/2.5 s: something repaints after the first paint — a carousel advancing, a font swapping (`font-display` not `optional`, or a face without a fallback), a banner or bar inserted after load, a fragment fetched on load above the fold, an image without dimensions loading in. |
| LCP up | The hero or product image: too big, not WebP, discovered late (a CSS background, a lazy attribute on the first image), or a preload missing. |
| TBT up | A new script: open the treemap; a third-party tag loaded before consent is the usual find. |
| CLS above 0 | An image or embed without a size; a font without a metric-matched fallback; content inserted above the fold. |
| FCP up | Time to first byte (the page not cached, a slow block) or something render-blocking added to the head: a second stylesheet, a synchronous script, a Google Fonts link. |
| Requests up | A module adding files per page; images without `srcset` loaded at full size; icons as separate files instead of inline SVG. |
