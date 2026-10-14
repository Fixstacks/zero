# Zero CMS

Pages and blocks built from components in the admin. There's no HTML and no
page builder markup. Each component is a theme template with a few typed
fields. The result is as fast and accessible as the rest of the theme,
works right to left, and every store view can have its own version.

Module: `Zero_Cms`. Demo page: `/zero-cms`.

## For editors

1. **Content > Pages** (or **Blocks**), open a page, then click **Edit with Zero CMS**.
2. Left: the components in order. Drag them, or use the ↑ ↓ buttons from
   the keyboard. **Add a component** inserts one after the selected component. The selected
   component's fields are below the list.
3. Right: the storefront preview of your unsaved changes, on desktop or
   mobile. It updates about a second after you stop typing.
4. **Save** publishes the content and clears the page from the page cache
   (built-in or Varnish). The previous content is kept: **Versions** lists the
   last 30 and loads any of them into the editor. Save again to publish it.

**Store views.** *All Store Views* is the default content. Pick a store
view in the toolbar to give it its own version, for example a translation. Until
you save there, it shows the default content. **Use All Store Views
content** removes its own version.

A page with Zero CMS content shows that content instead of its HTML
content. The HTML content stays as it was and comes back if the module is
disabled.

### Components

| Component | Use it for |
| --- | --- |
| Hero | The top of a page: heading, text, image, button |
| Text | Headings and paragraphs (a blank line starts a new paragraph) |
| Article | Long text in Markdown: `## Heading`, `### Subheading`, `- list`, `1. list`, `> quote`, `**bold**`, `*italic*`, `[link](/path)`, `![image](https://…)`. Headings get anchors; in a blog post they make the table of contents |
| Image | One image with a caption, text width or full width |
| Quote | A pull quote and who said it |
| HTML | HTML from elsewhere, e.g. posts imported into the blog. Cleaned when saved: no scripts, styles, event handlers or embeds |
| Image and text | An image beside a heading, text and button |
| Tiles | A grid of linked tiles: categories, collections |
| Product slider | Products from a category or a list of SKUs, as product cards |
| FAQ | Questions and answers, with FAQPage structured data for search engines |
| Countdown | A live countdown to a moment: a flash sale starting or ending, new prices, a launch. When it's over it shows its own message, or hides if there is none |
| Call to action | A short message with a button on a colored band |
| Spacer | Extra space |
| Banner | A slider from **Content > Banners** (`Zero_Banner`) |

The countdown's time is store time (the editor shows the time zone under
the field). It counts on the visitor's clock, so a page served from the
page cache stays correct, and it switches by itself at the deadline. To
change prices at that moment, set up a catalog price rule or special
prices for the same time; the countdown only shows the time.

Links accept store paths (`/women.html`), full `https://` URLs, `#anchors`,
`mailto:` and `tel:`. Images are uploaded from the field and stored under
`pub/media/zero/cms/`.

### Sharing in social media

Under **Sharing in social media** (CMS pages): the image, title and text a
shared link to the page shows on Facebook, LinkedIn, X, Telegram, WhatsApp
and the like. Empty: the page's meta title and description, and the first
image among its components. 1200 × 630 px images work everywhere.

The tags themselves (Open Graph, Twitter cards) are on every page of the
store: Stores > Configuration > Zero > Theme > **Sharing in Social Media**
turns them on or off (on by default) and sets the image for pages without
one and the store's X account. Products use their name, description, main
image, price and stock; categories their name, description and image.

### AI

When AI is on, the editor has three more tools:

- **Build the page with AI**: describe the page ("a summer sale page for
  running gear: 20% off shoes until August 31, best sellers, free returns,
  delivery FAQ"). You get a full set of components in the store view's language,
  with links and product sliders pointing at real categories. Images are
  left for you to add. The image alt text describes what to use.
- **Write / Improve / Shorten** under every text field. Write uses the
  page title and the component's other fields as context.
- **Translate into** [language]: translates every text on the page into
  any language Magento has a locale for. The store view's own language is
  preselected. Links, images, products and options stay as they were.
  Typical use: open a store view, translate, check it, save. That store view
  now has its own version.

Nothing the AI returns is published by itself. Answers load into the
editor as unsaved changes, so you check them in the preview and press Save.

## Setup

**Stores > Configuration > Zero > CMS > AI**

| Setting | |
| --- | --- |
| Enabled | Off by default |
| Provider | Anthropic (Claude). Others can be added, see below |
| Anthropic API Key | From [console.anthropic.com](https://console.anthropic.com/settings/keys). Stored encrypted |
| Model | `claude-opus-5-5` (default, best writing), `claude-sonnet-5-5` (faster, cheaper), `claude-haiku-4-5-20251001` (fastest) |

Or from the command line (the key goes to `env.php`, not the database):

```bash
bin/magento config:set zero_cms/ai/enabled 1
bin/magento config:set --lock-env zero_cms/ai/api_key sk-ant-...
```

**Permissions** (System > User Roles, under Content > Elements):

- *Zero CMS Editor*: the editor.
- *Use AI*: the AI tools. This is a separate permission because every request is
  paid for. Without it, the editor shows no AI buttons and the endpoints
  answer 403.
- The settings section has its own permission: *Stores > Settings >
  Configuration > Zero CMS*.

Requests go from the server to the provider. The key never reaches the
browser. Each request releases the admin session before calling the model.
An answer can take up to a minute, and the editor keeps working
meanwhile.

## For developers

### A component

Any module can add components in `etc/zero_cms/components.json`. All
enabled modules' files are merged in module order, so a later module can
also change an existing component (for example, its template). The result is cached with
the config cache. Run `bin/magento cache:clean config` after changing a file.

```json
{
    "testimonial": {
        "label": "Testimonial",
        "description": "A customer quote with their name.",
        "template": "Acme_Cms::components/testimonial.phtml",
        "fields": {
            "quote": {"type": "textarea", "label": "Quote", "required": true, "max": 600},
            "name": {"type": "text", "label": "Name", "max": 80},
            "photo": {"type": "image", "label": "Photo"},
            "style": {"type": "select", "label": "Style", "options": {"plain": "Plain", "card": "Card"}, "default": "plain"}
        }
    }
}
```

The code (`testimonial`) is lowercase letters, digits and `_`. `label`
and `description` show in the editor and are translated with the module's
i18n CSV. The description is also what the AI reads to decide when to use
the component, so make it say what the component is for.

Field types:

| Type | Value in the template | Options |
| --- | --- | --- |
| `text` | string, one line | `max` (255) |
| `textarea` | string, line breaks kept | `max` (5000) |
| `link` | `['url' => …, 'label' => …]` | |
| `image` | `['path' => …, 'alt' => …]`, path under `pub/media` | |
| `select` | one of the `options` keys | `options`, `default` |
| `boolean` | bool | `default` |
| `number` | int | `min`, `max`, `default` |
| `datetime` | `"YYYY-MM-DDTHH:MM"` in store time (`$view->timestamp()` converts it) | |
| `products` | `['source' => 'category'\|'skus', 'category_id' => int, 'skus' => string[]]` | |
| `repeater` | list of items, each with its own `fields` | `fields`, `max` (20) |
| `markdown` | string as typed; `$view->markdown()` renders it (everything escaped, then the subset above) | `max` (40000) |
| `html` | HTML cleaned by HTML Purifier on save (`Model\Content\HtmlSanitizer`) | `max` (200000) |

Every field may be `required` (shown in the editor; Save warns when one is
empty). Values are cleaned on every preview and save
(`Model\Content\Validator`). Unknown fields are dropped, text loses markup
(except `markdown`, escaped when rendered, and `html`, cleaned) and is cut to `max`, URLs are limited to the schemes above, and image paths
can't leave `pub/media`. Templates still escape everything they print.

### The template

```php
<?php
/**
 * @var \Magento\Framework\View\Element\Template $block
 * @var \Magento\Framework\Escaper $escaper
 */
$v = (array) $block->getData('values');      // the cleaned fields, all present
/** @var \Zero\Cms\ViewModel\Component $view */
$view = $block->getData('view');
?>
<figure class="<?= $v['style'] === 'card' ? 'card bg-base-200 p-8' : '' ?> max-w-2xl mx-auto flex flex-col gap-4">
    <blockquote class="text-xl"><?= /* @noEscape */ $view->paragraphs($v['quote']) ?></blockquote>
    <figcaption class="flex items-center gap-3">
        <?php if ($v['photo']['path'] !== ''): ?>
            <img src="<?= $escaper->escapeUrl($view->mediaUrl($v['photo']['path'])) ?>"
                 alt="<?= $escaper->escapeHtmlAttr($v['photo']['alt']) ?>"
                 class="size-12 rounded-full object-cover" loading="lazy" width="48" height="48">
        <?php endif; ?>
        <span><?= $escaper->escapeHtml($v['name']) ?></span>
    </figcaption>
</figure>
```

`$block->getData('uid')` is the component's stable id. `$view` has
`mediaUrl()`, `paragraphs()` (escaped `<p>`s from a textarea),
`productsHtml()` (a `products` field as the theme's product slider), and
`timestamp()` / `formatDateTime()` / `isoDateTime()` for `datetime` fields.

Each component is wrapped in `<div class="zero-cms-component zero-cms-{code}">`
inside a vertical stack with the theme's section spacing, so components
don't add outer margins of their own.

**Styles.** For Tailwind to see the template's classes, the module needs
`view/frontend/zero/tailwind.css` (see [extensions.md](extensions.md), §1):

```css
@source "../templates/**/*.phtml";
```

**Scripts.** Storefront pages run under a strict Content Security Policy:
no inline scripts and no `on*` attributes. A component that needs
JavaScript loads a static file from its template. See the slider and FAQ
for examples, and [components.md](components.md) for the theme's
data-attribute behaviors that need no script at all.

### An AI provider

`Model\Ai\ClientInterface` has two methods. `text()` returns a plain answer,
and `structured()` returns JSON matching a JSON Schema (Anthropic does this with a forced
tool call). Implement it and register it in `di.xml`:

```xml
<type name="Zero\Cms\Model\Ai\Assistant">
    <arguments>
        <argument name="clients" xsi:type="array">
            <item name="acme" xsi:type="object">Acme\Ai\Model\Client\Proxy</item>
        </argument>
    </arguments>
</type>
<type name="Zero\Cms\Model\Config\Source\AiProvider">
    <arguments>
        <argument name="providers" xsi:type="array">
            <item name="acme" xsi:type="string">Acme AI</item>
        </argument>
    </arguments>
</type>
```

The page schema the model fills is built from the component registry
(`Model\Ai\Schema`), so new components work with AI without extra code.
Whatever a provider returns goes through the same Validator as the
editor's input.

### Storage

| Table | |
| --- | --- |
| `zero_cms_content` | One row per page, block or blog post and store view (`store_id` 0 = All Store Views): the content as JSON |
| `zero_cms_revision` | Earlier versions, the last 30 per page, block or post and store view |

Other modules make their own things editable: implement
`Model\Content\Target\TypeInterface` (load, title, storefront URL, admin
Back link, menu) and add it to `Model\Content\Target\Pool` in `di.xml`.
`Zero_Blog` does this for posts (`zero_blog_post`).

On the storefront, `Plugin\Cms\RenderZeroContent` renders the content in
place of the page's or block's HTML (CMS pages, blocks and the CMS block
widget). Unsaved previews are stored in the cache for an hour and rendered
through a signed link (`zero/preview/index`) that only the editor creates.

## Disabling or removing

Its data lives in its own tables (`zero_cms_content`, `zero_cms_revision`). With the module's code still in
place, `bin/magento module:disable Zero_Cms` followed by `setup:upgrade`
**drops those tables** (Magento's declarative schema), and enabling the module
again brings them back empty. To keep the data, switch the feature off in
Stores > Configuration where it has a setting, or back up first:
`bin/magento setup:upgrade --safe-mode=1` dumps the data before anything is
dropped (`--data-restore=1` puts it back). Deleting the module's code instead
leaves the tables in place.
