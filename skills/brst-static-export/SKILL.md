---
name: brst-static-export
description: "Use when exporting a Bricks site as static HTML with Brickstatic, or explaining what an export does: export mode, which URLs are exported or left out, the export options and the destination, the ZIP and GitHub targets, runs and retention, what the ZIP holds, site search, redirects, sitemaps, extra pages and files, updating a page from Bricks, and the WP-CLI commands."
---

# Brickstatic: how an export works

Read `brst-start-here` first. Brickstatic lives on **Brickstatic > Export**, in three tabs:
**Export** holds the three steps in order (**1. Check the site**, the preflight; **2. Choose where it goes**, ZIP or
GitHub and the address of the static site; **3. Export**, the run, its progress and the last export's ZIP, with
**Update one page only** folded under it); **History** holds **Exports** (the runs) and, once GitHub is set up,
**Releases**; **Settings** holds **What goes in** (the options), **Address and forms**, and **GitHub**.

## A run, step by step

1. **Preflight.** Every export runs it first; a blocker stops the start (`brst_preflight_blocked`). See
   `brst-preflight-findings`.
2. **Discovery** lists the jobs from the database, not by following links:
   - published posts of public post types (not attachments, not Bricks templates); password-protected posts are left
     out and listed;
   - term archives with their pagination, post type archives, author and date archives (both on by default);
   - the front page, the posts page, a 404 page, `robots.txt`, `favicon.ico`, and the sitemaps `robots.txt` announces:
     an SEO plugin's index (Yoast SEO, Rank Math, SEOPress, All in One SEO, The SEO Framework) or WordPress' own
     `wp-sitemap.xml`, each with its children and stylesheet;
   - with site search on and a Search element on the site, the search results page (`/search/`, or `/site-search/`
     when the site exports a page, an archive or an Extra page of its own at `/search/`);
   - the owner's **Extra pages** (addresses nothing links to), marked `manual`; one WordPress answers with a 404 or a
     redirect is skipped with a run warning, and a file typed there is not a page (it belongs under Extra files);
   - the pagination pages of every paginated loop in Bricks data; a loop in a template is expanded once per post the
     template applies to; a loop inside a component or global element is reported, not expanded;
   - for load more and infinite scroll past 100 items, the extra pages under `/<page>/brst/<loop>/<n>/`;
   - on a multilingual site, every translation at its own language address (Polylang and plugins like it), or every
     page once per language under the language's folder (TranslatePress); the preflight's `languages` row lists them.
   Left out: search results, login, `wp-admin`, the REST API, WooCommerce cart, checkout and account with every
   address under them, author archives of authors without a user, drafts, and the owner's **Leave out** patterns.
   `wp brickstatic discover` prints the job list.
3. **Pages** are fetched in export mode: anonymously (no admin bar, no edit links), each fetch carrying
   `?brst-nocache=<run id>` so page caches let it through and never keep it (WordPress never sees the argument).
   Each page proves export mode by its answer headers or, when a host strips those, by a proof inside the page made
   with the run's token. A stored copy from an earlier export is refused. When three pages in a row come back without
   export mode, or a cache answers the export's fetches before the first page, the run stops with one sentence that
   names the cause, instead of failing page after page.
   **The link safety net:** after the listed pages, links on them to this site that no job covers (no query string,
   not left out, not a file) are fetched too, at most 500 in two passes, and listed as found by link only.
   A page or file larger than the host accepts in one request goes up in parts. A page WordPress answers with a redirect
   (an archive an SEO plugin switched off, a moved page) is skipped, not failed.
4. **Rewriting.** Links to this site become relative, so the export works from any folder; `404.html` keeps
   root-relative links; sitemaps, `robots.txt`, `canonical`, `hreflang` links and `og:url` use the destination base
   URL.
5. **Assets**: the files the pages use (images, `srcset`, CSS `url()` and `@import`, icon fonts, Bricks' CSS files),
   copied from disk or fetched, and the owner's **Extra files** (copied from disk only). Then, before the check:
   redirects (`_redirects` and redirect pages), sitemaps taken down to the pages the export has, and the search index,
   in steps of a few seconds where it can be long.
6. **Google Fonts** are copied into the export by default, linked ones and ones a stylesheet imports, so the static
   site never calls Google.
7. **Verify** (in steps of about four seconds) checks the tree: every internal link exists, every referenced file resolves, no reference to the
   WordPress site is left, every page carried the export header. Failures are listed, never rounded away.
8. **The archive**, `site.zip`, is built in a fixed order with its sha256 recorded. For the GitHub target the push
   follows (`brst-github-deploy`).

## What export mode changes in a page

Only while a page renders for the export; Bricks' theme files and settings are never edited.

| Feature | In the export |
|---|---|
| Lazy load | turned off, with `loading="lazy"` on images unless the image sets its own loading or high fetch priority |
| Several loops on a page | the paginated loop moves to its page; every other loop stays on page 1 |
| AJAX pagination | plain links to pre-rendered pages |
| Load more, infinite scroll | up to 100 items pre-rendered and revealed in batches by a small script; past that a "See all" link to separately exported pages |
| AJAX popups | rendered into the page, so they open without a request |
| Query filters, live search | not rendered; the loop exports unfiltered with plain pagination |
| Search element | kept, with the static search script (site search on); left out when site search is off |
| Comments | the whole comments area removed; existing comments are not exported |
| Relative dates | rendered once at export time, then they stand still |
| WordPress runtime | `ajaxUrl`, REST URL and nonces blanked (kept only where a form sends to this WordPress site); REST discovery, oEmbed, feed links, emoji script and generator removed |
| Forms | as each form's Brickstatic group says (`brst-forms-routes`) |
| WooCommerce | a catalogue: products, the shop and categories keep prices and descriptions; add-to-cart buttons and forms, quantity fields, the mini cart, the sort menu and the scripts that call the cart are removed; menu items to cart, checkout and account are left out |

## The options

Option `brst_static_export`, saved from the screen:

| Settings card, label | Key | Default |
|---|---|---|
| What goes in, Google Fonts local | `google_fonts_local` | on |
| What goes in, Author archives | `include_author` | on |
| What goes in, Date archives | `include_date` | on |
| What goes in, Site search | `search` | on |
| What goes in, Redirects | `redirects` | on |
| What goes in, Leave out (one path per line; a plain path leaves out that page and everything under it; `=/about/` that page only; `*` and `?` match like a shell pattern) | `exclude_patterns` | none |
| What goes in, Extra pages (addresses of this site nothing links to) | `extra_pages` | none |
| What goes in, Extra files (files or folders under the install: copied from disk; never PHP, dot files but `.well-known` (whose files may have no extension), the runs folder, the files the export writes itself (`robots.txt`, `favicon.ico`, `index.html`, `404.html`), the whole site, `wp-content` or uploads folder, or a folder closed to visitors by `.htaccess` or `web.config`; at most 5,000) | `extra_files` | none |
| What goes in, Parallel requests (pages fetched at once, 1 to 6) | `parallel` | 3 |
| What goes in, Keep files of the last N runs (1 to 10) | `retention_runs` | 3 |
| Address and forms, Address of the static site (the base URL) | `base_url` | none |
| Address and forms, More addresses the forms accept (static origins) | `static_origins` | none |
| Address and forms, WordPress address for forms | `forms_origin_url` | this site |

Without a base URL, sitemaps and canonical links keep this site's address. Set it to the static site's address.

A page's own **Page settings → Brickstatic** in Bricks: **Leave this page out of the static export** (key
`brstExportExclude`, an exact leave-out pattern for that page) and **Leave this page out of the site search**
(`brstSearchExclude`, still exported).

## Targets

- **ZIP**: always available. **Download ZIP** under step 3 (the run's result, or the last export) or on History,
  Exports. The link takes the screen's own address, so a site behind a proxy that ends HTTPS still downloads it. On a
  plain `http://` site the browser warns about every download; the screen says so there and the owner chooses Keep.
- **GitHub**: available once a repository and a token are saved under Settings, GitHub; until then step 2 shows it
  disabled with its reason and a Set up GitHub link. See `brst-github-deploy`.

## Runs and retention

A run is a folder under the uploads folder holding `run.json`, the tree and `site.zip`. History, Exports lists the
last 20 runs (Started, Pages, Size, Status, Check; the newest five in view, the rest folded); the files of the newest N finished runs are kept (setting
above), older ones keep only their summary. A run left for more than six hours cannot be resumed. On the screen a run
in progress is resumed after a reload, or discarded. Delete removes a run with its files; a run in progress cannot be
deleted, unless it is abandoned (not the current run, or silent for more than six hours): then Delete cancels it first.

## What the ZIP holds

Each page as `<path>/index.html`, `404.html` at the root, `robots.txt`, `favicon.ico`, the sitemaps when exposed, the
files the pages use at their WordPress paths, localised Google Fonts under `assets/fonts/google/`, and when they apply:
`brst-search-index.js` and `search/index.html` (site search), `_redirects` and a redirect page at each old address,
the owner's Extra files. It never holds `wp-config.php`, the theme folder, a PHP file, or any file no page referenced
and the owner did not list.

A server without PHP's zip extension still gets its ZIP (1.2.2): Brickstatic writes it itself, at most 65,535 files or
4 GB (preflight note `zip_builtin`). When no ZIP can be built, the screen's log says why after "No ZIP could be built",
and the files are in the run's folder on the server.

## Site search

The Search element (plain or overlay) works on the static site: results as you type in a list under the field, Enter
opens the results page with every match, twenty at a time. The index is built from the exported singular pages' main
text (not archives, headers or footers), about 3 MB of page text up to about 7,500 pages; accents and case are ignored,
a `post_type` parameter on the element narrows the results, and on a site in several languages results follow the
page's language. It works from a folder opened on disk too. A search form that is not a Bricks Search element (a
WordPress widget, a shortcode) gets the script only on a site that also has a Search element, and only on pages where
one renders. Colours: the CSS variables `--brst-search-bg`, `-fg`, `-muted`, `-border`, `-hover`, `-mark`.

## Redirects

Redirection (enabled URL redirects), Rank Math (active; exact, starts with, a regex that is a plain prefix), Yoast
Premium's stored redirects and WordPress' renamed post slugs (`_wp_old_slug`; WordPress keeps no old address for a
renamed page) go into `_redirects` (the file several static hosts read) and, for every exact old address that is a
folder or an `.html` name, a redirect page (any host, and from disk). A prefix redirect has no redirect page: it works
only on a host that reads `_redirects`, and the preflight row says how many there are. What a static host cannot do
(other patterns, contains or ends with, conditions, query strings, 410 or 404 answers, a target on another site that
is not an http or https address) is listed in the preflight row `redirects` and the run's warnings, never approximated. A redirect page
never replaces a page or a file of the export. On GitHub the export's lines are a marked block in the repository's
`_redirects`, below the owner's own lines (`brst-github-deploy`).

## Sitemaps

The sitemaps `robots.txt` announces go in with the base URL; an entry whose page the export does not have is taken out,
and a child sitemap left empty leaves the index. A leaked export token exposes only what a visitor
could already see.

Every page carries one small script (`brst-disk-links`): only when the page is opened as a file, a click on a folder
link opens that folder's `index.html`, so an unzipped export can be looked at by opening `index.html`. On a host it does
nothing. With Google Fonts local, a page that loads no Google stylesheet keeps no preconnect or dns-prefetch to Google.

## Updating one page

Re-exports one page and the files it uses, into a copy of the last complete export:

- **From Bricks**: **Update static site** in the builder's toolbar (just before Save) or in **Page settings → Static
  Export** opens a small window over the builder; the page must be saved and published. Also **Update static site**
  under a page's title in the Pages list and in the admin bar.
- **On the screen**: **Update one page only** under step 3 (**Re-export this page**); `export-start` with `only`
  (and `changed_only`); `wp brickstatic export --only=<url> [--changed-only]`.

Where it goes, GitHub first: **GitHub: commit the changed files** (picked once GitHub is set up), **ZIP: changed files
only** (just what the update changed, to unzip over the live copy; when nothing changed there is no ZIP and the screen
says so), **ZIP: whole site** (the last export with the page replaced). The search index and `_redirects` are rebuilt
from the whole tree, and the sitemaps are fetched from WordPress again and taken down to the pages of the whole tree,
so they travel in the changed files when they changed.

The copy comes from the newest complete export still on disk (a full export, or a page update copied from one).
Without one, the update holds that page alone and the window says to run a full export first. When the static site's
address or the author or date archive switches changed since that export, the update is refused (`brst_settings_changed`):
run a full export first. A copied page that is now left out, or whose old address now redirects to a page the site
has (a renamed post), is removed from the copy; GitHub deletes it on the branch. A changed files ZIP cannot remove a
file: the run's warnings name what to delete on the host by hand (removed pages and old redirect pages).

## Where an export runs

- **The owner's browser**, from Start export: the default, and it works on any host because PHP never waits on
  itself. The tab must stay open (Pause and Cancel are on the panel). A host that blocks an address after many quick
  requests (1.2.2): five pages in a row refused or unanswered pause the export on its own with a sentence saying so, no
  page counted as failed; after Resume it fetches one page at a time with a gap. Already blocked: wait, set Parallel
  requests to 1 under Settings, reload the screen and resume the run.
- **WP-CLI**, server-side: `wp brickstatic export [--target=zip|github] [--parallel=<n>] [--only=<url>]
  [--changed-only] [--base-url=<url>] [--skip-fonts] [--force] [--format=table|json]`. A CLI process is not a PHP worker, so it suits
  large sites. `--parallel` is 1 to 6, default 1; only a host with free PHP workers gains from more: above 1 the
  worker probe runs first and the number is capped at the free workers it saw (a line says so).
- **The abilities**, server-side in steps, only where at least two PHP workers are free (`brst-start-here`).
- **Automatic updates** (1.1.0), on the server in the background, when the owner switches them on (below).

## Automatic updates (1.1.0)

**Brickstatic > Export > Settings > Automatic updates**, off until the owner switches it on. The whole site is exported
again on its own: **When content changes** (a page, post or other exported post type, a Bricks template or a menu
published, updated, unpublished or deleted; Bricks' site-wide settings saved; the builder's Save counts; drafts,
autosaves and revisions never do) and **On a schedule** (every day or every week at a time on the site's clock).
Where it goes: GitHub when it is set up (only the changed files are pushed; an unchanged site pushes nothing), else a
ZIP kept in History.

- An update starts right after the change; changes made while one runs give one more update when it ends.
- An export started by hand goes first: the automatic update waits for it and starts when it ends.
- Each background request runs about 20 seconds of server-side steps and hands over to the next. That needs the site
  able to send a request to itself and one free PHP worker while it runs; the card checks both when it is saved. On a
  host that cannot, the card shows a server cron job that runs `wp cron event run --due-now` through WP-CLI every five
  minutes, which runs updates on any host (WP-CLI is not a PHP worker) and keeps schedules on time.
- A run started this way has the context `auto` and shows on the Export tab with **Stop the automatic update**;
  `export/<run>/resume` refuses it (`brst_auto_live`): the server drives it.
- A failure is shown on the card and sends one email to the site's admin address, and none again until an update
  works; the filter `brickstatic/auto/failure_email` changes or empties the address.
- They count towards **Keep files of the last N runs** like any export.

## WP-CLI commands

| Command | Does |
|---|---|
| `wp brickstatic status` | the plugin, Bricks and licence state, and whether the export can run |
| `wp brickstatic preflight [--format=table\|json]` | runs and stores the preflight; exit 1 on a blocker |
| `wp brickstatic discover [--format] [--kind=<kind>]` | the job list; kinds page, post, archive, pagination, reveal, 404, search, sitemap, robots, favicon |
| `wp brickstatic regenerate-css` | Bricks' CSS files regenerated with Bricks' own generator, then the preflight again |
| `wp brickstatic export-token issue [--ttl=<seconds>]` | prints a new export token once, for a crawl driven from outside the site |
| `wp brickstatic export-token revoke` | revokes it |
| `wp brickstatic export ...` | the export (above) |
| `wp brickstatic runs list [--format]` | the remembered runs |
| `wp brickstatic runs delete <id>` | deletes a run's files and summary |
| `wp brickstatic workers [--max=<n>] [--timeout=<seconds>] [--format]` | how many PHP workers answer the probe right now; from WP-CLI every worker of the pool can answer, inside a web request its own worker is busy |
| `wp brickstatic abilities list [--format]` | the five abilities and whether they are registered (the AI agents switch) |
| `wp brickstatic abilities run <name> [--input=<json>] --user=<administrator>` | runs one ability as that user and prints its answer as JSON |
| `wp brickstatic auto status [--format=table\|json]` | the automatic updates: settings, where an update goes, the run under way, the last result, the next scheduled one, the server check, the last decisions |
| `wp brickstatic auto set [--on-change=on\|off] [--schedule=off\|daily\|weekly] [--time=HH:MM] [--day=0-6]` | the same as the card; only on the owner's word |
| `wp brickstatic auto run` | an automatic update now, in this process, waiting for it; only on the owner's word |

The GitHub and release commands are in `brst-github-deploy`. Ask the owner before deleting a
run, issuing a token or changing automatic updates: an export token switches export mode on for any request that
carries it until it expires, and an automatic update with GitHub set up is a push.
