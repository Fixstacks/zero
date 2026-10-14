# Zero Blog

Module: `Zero_Blog`. A blog for the Zero storefront: posts written in the
Zero CMS editor (with AI), categories, tags, authors, products in posts and
posts on product pages, comments with moderation, an RSS feed, and the SEO
search engines look for. No JavaScript on its pages; everything is cached
by the full page cache or Varnish and refreshed by cache tags when you save.

## For editors

**Content > Blog**

| Section | |
| --- | --- |
| Posts | Title, URL key (made from the title when empty), publish date (a future one schedules the post), author, store views, excerpt, featured image with alt text, categories, tags (typed, comma separated: new ones are created), products (SKUs), SEO fields, comments on or off |
| Categories | Title, URL key, description, position, SEO fields |
| Tags | The same, without position |
| Authors | Name, job title, bio, photo, website, LinkedIn and X links |
| Comments | Approve, reject or delete. A comment shows under its post once approved |
| Settings | Stores > Configuration > Zero > Blog |

**The text of a post** is written in the Zero CMS editor: **Save and Edit
Content** on the post form, or **Edit Content** in the grid. Posts use the
same components as pages, plus:

- **Article**: long text in Markdown. `## ` and `### ` headings become the
  table of contents beside the post (three or more `##` headings).
- **Image**, **Quote**, and **HTML** (for imported posts).

The editor's AI drafts a post from a description, rewrites a text field,
and translates a post. A post with no content yet shows its excerpt.

**Products**: list SKUs on the post. They show under the post as product
cards, and the post shows under **From the blog** on their product pages
(up to three posts per product). Products can also go inside the text
with the Product slider component.

**Languages**: a post belongs to store views. A post in another language
is its own post, assigned to that store view; it can use the same URL key.

## URLs

| | |
| --- | --- |
| `/blog` | The newest posts (`?p=2`, `?p=3`… for older pages) |
| `/blog/<url-key>` | A post |
| `/blog/category/<url-key>` | A category |
| `/blog/tag/<url-key>` | A tag |
| `/blog/author/<url-key>` | An author with their posts |
| `/blog/rss.xml` | RSS 2.0, the 20 newest posts |

`blog` is **Blog Path** in the settings. When a published post's URL key
changes, its old URL redirects (301) to the new one. Redirects are URL
rewrites of type `zero-blog` (Marketing > URL Rewrites).

## Search engines

Every blog page has:

- a title, meta description and canonical URL (a post can point its
  canonical at the place it was first published, and set meta robots);
- Open Graph and Twitter card tags (`og:type` article for posts, with
  `article:published_time`, `article:author`, `article:section`, `article:tag`),
  following the store's Sharing in Social Media setting (on, off, X account,
  default image);
- structured data (JSON-LD): `BlogPosting` with author, publisher, dates,
  image, word count and reading time; `Blog` on the index;
  `CollectionPage` for categories and tags; `ProfilePage` with `Person`
  (and `sameAs` profile links) for authors; `BreadcrumbList` everywhere;
- breadcrumbs: Home > Blog > Category > Post;
- semantic HTML: one `h1`, `article`, `time datetime`, `rel="author"`,
  `rel="tag"`, `rel="prev"`/`"next"` on page links;
- the featured image as WebP in several sizes, loaded eagerly with
  `fetchpriority="high"` on a post (it is the largest paint), lazily in lists.

Posts, categories and authors with posts are added to Magento's XML
sitemap (Marketing > SEO & Search > Site Map) with their dates and the
featured image. Priority and frequency: settings > XML Sitemap.

AMP isn't supported on purpose: Google stopped favouring AMP pages in 2021,
and Zero pages are already fast.

## Comments

Readers leave a name, email and comment. Every comment waits for approval;
the email is never shown. Spam protection without a captcha: a hidden field
that bots fill (their comments are dropped, they get the normal answer),
five comments per visitor in ten minutes, no more than three links. Turn
comments off for the store (settings) or for one post.

## Importing

```bash
bin/magento zero:blog:import wordpress --file=export.xml
bin/magento zero:blog:import magefan
bin/magento zero:blog:import amasty
```

| Source | What is read |
| --- | --- |
| WordPress | The export file (Tools > Export > Posts). Published, scheduled and draft posts, authors, categories, tags, featured images, Yoast or Rank Math SEO fields. WordPress's line breaks become paragraphs; shortcodes are dropped (captions keep their image and text) |
| Magefan Blog | Its tables in this database (still there after the extension is disabled): posts, categories, tags, store views, featured images. Authors from its author table or the admin users who wrote posts |
| Amasty Blog | Its tables in this database: posts (status published or scheduled), categories, tags, authors, store views, thumbnails |

A post's text becomes one HTML component (cleaned: no scripts, styles or
embeds). Magento directives such as `{{media url="…"}}` become real URLs.
Featured images are copied into `pub/media/zero_blog/import` (downloaded
for WordPress). Each post's old address redirects (301) to the new one:
WordPress uses the post's real link, Magefan `blog/post/{key}`, Amasty
`blog/{key}.html`; change it with `--old-pattern=news/{key}`.

Options: `--dry-run` (show what would happen), `--no-images`,
`--no-redirects`, `-v` (one line per post). Posts whose URL key already
exists are skipped, so it can run again. Clean the page cache afterwards.

## For developers

| Table | |
| --- | --- |
| `zero_blog_post` | Posts (`published_at` in UTC, `reading_time` counted when the text is saved) |
| `zero_blog_post_store`, `_category`, `_tag`, `_product` | A post's links |
| `zero_blog_category`, `zero_blog_tag`, `zero_blog_author` | |
| `zero_blog_comment` | Comments (`status`: 0 pending, 1 approved, 2 rejected) |

A post's text is Zero CMS content of type `zero_blog_post`
(`Model\Cms\PostTarget` makes it editable). `Controller\Router` maps the
URLs above; `Model\Page\Meta` sets each page's head and breadcrumbs,
`Block\Head` adds Open Graph and JSON-LD and carries the page's cache tags.

Cache tags: every blog page has `zero_blog`; a post `zero_blog_post_<id>`;
the "From the blog" block `zero_blog_product_<product id>`, which a post
carries for every product it links to (or linked to). Saving a post
refreshes its page, the lists, and only those product pages. A new pending
comment refreshes nothing; approving one refreshes its post.

Templates are in the module (`view/frontend/templates`), styled with the
theme's Tailwind classes; override them in a child theme as usual.

### Tests

`dev/tests/zero-e2e/tests/blog.spec.ts` (storefront: lists, a post's head
tags and structured data, table of contents, product links, RSS, comments)
and `blog-admin.spec.ts` (admin: a post written and published, its content
edited, a comment approved). Unit tests in `Test/Unit`.

## Disabling or removing

Its data lives in its own tables (`zero_blog_author`, `zero_blog_post`, `zero_blog_post_store`, `zero_blog_category`, `zero_blog_post_category`, `zero_blog_tag`, …). With the module's code still in
place, `bin/magento module:disable Zero_Blog` followed by `setup:upgrade`
**drops those tables** (Magento's declarative schema), and enabling the module
again brings them back empty. To keep the data, switch the feature off in
Stores > Configuration where it has a setting, or back up first:
`bin/magento setup:upgrade --safe-mode=1` dumps the data before anything is
dropped (`--data-restore=1` puts it back). Deleting the module's code instead
leaves the tables in place.
