# Changelog

Each release of this pack names the Brickstatic version it was checked against (`verifiedUpTo` in
`skills/brst-update/references/index.json`). Skills and plugin are released together; a pack never describes a
plugin that has not shipped.

## 0.3.5, 2026-10-24

Verified up to Brickstatic 1.2.5 (GitHub pushes with identical files, History relists lost runs, form shortcodes in page
content). It goes out with Brickstatic 1.2.5.

- `brst-preflight-findings`: `forms_unhandled` also lists form shortcodes and blocks in ordinary page content; what a
  form plugin's own form does from a static copy (measured).
- `brst-forms-routes`: shortcodes in page content; a CRM fed through a Bricks Form's Webhook action (measured).

## 0.3.4, 2026-10-24

Verified up to Brickstatic 1.2.4 (Windows paths, the files count while files are fetched). It goes out with
Brickstatic 1.2.4.

- `brst-static-export`: Windows, and what an export made there before 1.2.4 may lack.

## 0.3.3, 2026-10-24

Verified up to Brickstatic 1.2.3 (no PHP warning on a fresh install). No skill changed: nothing the skills describe
moved. It goes out with Brickstatic 1.2.3.

## 0.3.2, 2026-10-23

Verified up to Brickstatic 1.2.2 (the ZIP without the PHP zip extension, a host that blocks the browser). It goes out
with Brickstatic 1.2.2.

- `brst-preflight-findings`: the note `zip_builtin` (a server without PHP's zip extension; Brickstatic builds the ZIP
  itself).
- `brst-static-export`: the ZIP without the zip extension, the reason when no ZIP is built, and the export pausing on a
  host that blocks the browser.

## 0.3.1, 2026-10-22

Verified up to Brickstatic 1.2.1 (two admin screen fixes: the GitHub Token field shows again, the export result's
spacing). No skill changed: nothing the skills describe moved. It goes out with Brickstatic 1.2.1.

## 0.3.0, 2026-10-20

Verified up to Brickstatic 1.2.0 (BFB Advanced Forms on the static site). It goes out with Brickstatic 1.2.0.

- `brst-forms-routes`: BFB Advanced Forms (always to this WordPress site, the Error message only, payments and Save for
  later back on the static site, what cannot work), and form elements the preflight lists as not handled.
- `brst-preflight-findings`: the rows `forms_advanced_cart`, `forms_advanced_login`, `forms_advanced_pay_address`,
  `forms_unhandled` (warnings) and `forms_advanced_pay` (a note); the `forms` row counts Advanced Forms.

## 0.2.0, 2026-10-20

Verified up to Brickstatic 1.1.0. It goes out with Brickstatic 1.1.0, and is the pack's first published release
(0.1.0 was prepared for 1.0.0 and never published).

- `brst-static-export`: automatic updates (the card, what starts one, where it goes, the background requests and the
  server cron line, the `auto` context, the failure email) and `wp brickstatic auto status|set|run`.
- `brst-start-here`: never switch automatic updates on or off, change their schedule or run one without the owner's
  word; leave a running one to finish.

## 0.1.0, 2026-10-12

Verified up to Brickstatic 1.0.0. It goes out with Brickstatic 1.0.0.

- First pack: `brst-start-here`, `brst-static-export`, `brst-preflight-findings`, `brst-forms-routes`,
  `brst-github-deploy` and `brst-update`.
- 2026-10-15: the plugin was "Bricks Extensions" with a second extension, Breakpoint Canvas, until this date; the canvas
  moved to BFB Elements Pro and its skill to the BFB Elements pack, and every name here moved to Bricks Static Export
  (`bricks-static-export`, skills `bse-*`).
- 2026-10-19: the plugin is named **Brickstatic**; every name moved again: the pack `brickstatic-skills` (repository `builtforbricks/brickstatic-skills`), the skills `brst-*`, the upgrade script `brst-skills-upgrade`, and every option, command and screen name the skills quote.
- Brought up to the release candidate: the header probe's cases, the
  Woo, language and JSON-with-a-file rows in `brst-preflight-findings`; caches, the link safety net, languages and the
  Woo catalogue in `brst-static-export`; popups, loops, components, file fields and `{{field}}` tokens in
  `brst-forms-routes`.
- Brought up to rc.5: the Brickstatic screen's three tabs and three steps in every skill that names a place on it;
  the download link on the screen's own address, the http:// note, the from-disk link script and the Google hints in
  `brst-static-export`.
- Brought up to rc.7 (the 1.0 additions, 2026-10-16): updating a page from Bricks and the changed files ZIP, site
  search, redirects, extra pages and files, SEO plugin sitemaps, redirected pages skipped, in `brst-static-export`; the
  `search_forms`, `redirects`, `extras` and `sitemaps` rows in `brst-preflight-findings`.
- Corrected after the review of rc.7 (2026-10-17): site search serves the Bricks Search element only, and its 3 MB
  holds up to about 7,500 pages; redirects come from renamed posts, not pages, a prefix redirect works only on a host
  that reads `_redirects`, and on GitHub the export's lines are a marked block below the owner's own; a page update
  fetches the sitemaps again, refuses after the address or archive settings changed, and removes pages now left out
  (a changed files ZIP lists them for deletion by hand); Extra pages and files name their new refusals (the whole
  site, wp-content or uploads folder, folders closed to visitors, the export's own files, a file under Extra pages,
  404 and redirected pages), and files in `.well-known` may have no extension; the new `search_engines` row and the
  no-sitemap note in `brst-preflight-findings`.
