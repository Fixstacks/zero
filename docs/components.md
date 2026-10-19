# Component library

Zero's building blocks, live, each with its markup to copy into a template:
**Content → Design → UI Components** in the admin (permission: Zero UI
Components). Pick a store view to see the components with its theme,
language and direction (the Arabic store shows them right-to-left).

The admin page shows the storefront's `/zero/ui/` in a frame through a
signed link valid for an hour, so the page itself stays closed to the
public. To open `/zero/ui/` to everyone (a demo, a design review without
admin access), turn on **Stores → Configuration → Zero → Theme → Component
Library → Public at /zero/ui/**. It is never cached or indexed. If the admin
runs on another host than the storefront, frames are refused: use "Open in
a new tab" there.

| Component | What it is |
| --- | --- |
| Buttons | DaisyUI `btn` variants; icon-only buttons with `aria-label` |
| Form fields | Labelled inputs, select, textarea, checkbox, radio group; required and error states |
| Messages | Magento's `div.message` success / info / warning / error |
| Badges | Product badges (New and Sale; the product's own, Only a few left and Free shipping with Zero_ProductLabels; each on or off in Stores > Configuration > Zero > Theme > Product Badges) and DaisyUI badges |
| Accordion | Native `<details>`; a shared `name` keeps one open |
| Tabs | DaisyUI radio tabs, no JavaScript |
| Dialog | Native `<dialog>`, opened by `data-dialog-open`, closed by `<form method="dialog">` |
| Slider | Scroll-snap track with `data-scroller-prev/next` buttons, RTL-aware |
| Quantity stepper | Number input with `data-qty-step` buttons |
| Steps | DaisyUI `steps` with `aria-current="step"` |
| Loading (HTMX) | `htmx-indicator` shown while a request runs |

All of it follows the theme's rules: Tailwind/DaisyUI classes, native HTML
first, behaviour through data attributes handled in `web/js/zero.js` (no
inline scripts, so it runs under the strict Content Security Policy), texts
through `__()`, and WCAG 2.2 AA (the e2e test `ui.spec.ts` checks the page
with axe-core).

## How it's built

Each component is one template in `Zero_Core::ui/components/`. The page
renders it and shows the same file as code, so the example and the code
can't drift apart. Add your own: a template, plus an entry in
`Zero\Core\ViewModel\UiComponents` (`components` argument, extendable from
any module's `di.xml`).
