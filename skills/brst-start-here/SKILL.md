---
name: brst-start-here
description: "Load first in any session that checks, exports, deploys or explains a site with Brickstatic, the Bricks Builder plugin by Built for Bricks (a static HTML export of the site, the brickstatic/* abilities, the `wp brickstatic` commands): how the plugin fits together, the owner's switch, the five abilities, the workflow and what never to touch."
---

# Brickstatic: start here

Brickstatic publishes the Bricks site as static HTML that keeps its forms working, as a ZIP or pushed to a
GitHub repository. It renders every page the way a visitor sees it, in a request-scoped export mode, and never edits a
Bricks file or setting on the site. It runs while the plugin is active; there is no other switch. Load Bricks' own
`bricks-start-here` first when it exists. Every other `brst-*` skill assumes you have read this one; how an export works
is in `brst-static-export`.

The licence keeps updates and support coming; the export works without it. Nothing an agent does needs it.

## The owner's switch

The abilities exist only while the owner has switched on **Brickstatic > Settings > AI agents** (the switch
"Agent abilities", stored as `brst_settings.agent_abilities`, off by default). The admin menu can sit under Bricks or
Tools instead of its own item (Settings > Admin menu); the screens are the same. While the switch is off, no category
and no ability is registered and nothing is listed. The Brickstatic section appended to Bricks' start-here text
says which state the switch is in; it is there either way.

**When the abilities are missing**, the switch is off or WordPress is older than 6.9. Tell the owner where the switch
is and stop. Do not work around it with other tools (Bricks' PHP execution, options, database writes).

## The abilities

Category `brickstatic`. Administrators only: anyone else gets `brst_forbidden` ("Only an administrator can use
the Brickstatic abilities."). They write nothing but the plugin's own run records, the stored preflight report
and the export token.

| Ability | Input | What it does |
|---|---|---|
| `brickstatic/preflight` | none | Read-only for the site (it stores the report, as the screen does). Answers `ok`, `counts` (`blocker`, `warn`, `note`), the `rows` (`id`, `level` as `blocker`, `warn` or `note`, `feature`, `where` as text with titles and addresses, `consequence`, `fix`), `bricks`, `plugin_version` and `static_export_enabled`. Read each row with `brst-preflight-findings`. |
| `brickstatic/export-start` | `target` (`zip` by default, `github`, or `server` since 1.3.0), `only` (page URLs or paths, for a page re-export), `force` | Starts a run like the screen's Start export, fetches nothing yet, and answers `run_id`, `jobs`, `parallel` (the free workers less one, so the site keeps one for visitors), `partial`, `target`, the `preflight` counts and `workers` (`free`, `probed`). Refuses as the screen does: a blocker in the preflight (`brst_preflight_blocked`), a run still in progress (`brst_run_live`), a site that does not meet the requirements (`brst_requirements`), a target that is not set up (`brst_target_unavailable`). The export token stays on the server. |
| `brickstatic/export-step` | `run_id`, `pages` (1 to 20, default 10) | One bounded step: up to `pages` pages fetched by the server, else one step of assets, fonts, finish or deploy. Answers `done`, `status`, `phase` (`pages`, `assets`, `fonts`, `finish`, `deploy`), `counters`, `fetched`, `warnings_tail`, `download_url`, `release` and `wait`. Call it until `done`; when `wait` is above 0 (GitHub asked for a pause), wait that many seconds first. |
| `brickstatic/export-status` | `run_id` | Read-only. The run's status, counters, last warnings and log lines, the verify result, the archive and the release, with `download_url` (the ZIP's address for a signed-in administrator). |
| `brickstatic/runs-list` | none | Read-only. The remembered runs, newest first, and the run in progress (`current`). |

**The worker rule.** `export-start` first measures how many PHP workers are free (about two seconds). When fewer
than two are free it refuses with `brst_no_free_workers` before any run exists: "This server has fewer than two free
PHP workers, so an export driven through abilities would wait on itself. Run it from Brickstatic in WordPress, or
with wp brickstatic export." The request your call arrives in holds one worker itself, so a host with one or two
workers always refuses. The measurement is stored in the run; `export-step` refuses with the same code when it finds
fewer than two there, and then cancels a run an ability started so it does not block the next export. Say so to the
owner and point at the two routes the sentence names; do not retry. Drive only the run your own `export-start`
answered: never call `export-step` on a run the owner started.

## Workflow

1. `brickstatic/preflight`. Say the plugin version, whether Brickstatic is on, and the counts.
2. Explain the blockers first, then the warnings, with `brst-preflight-findings`. Notes say what the export does on
   its own; mention the ones that change what visitors see (comments, relative dates, frozen code output).
3. **Show the owner every fix before making it.** Fixes happen in Bricks or WordPress (a page's settings, a menu, a
   form's Brickstatic group), through Bricks' abilities, and only on the owner's word.
4. Exporting: the default is the owner's browser on **Brickstatic > Export** (works on any host). For a
   large site the owner runs `wp brickstatic export`. Through the abilities: `export-start`, then `export-step`
   until `done`, then `export-status`. Never start an export while the owner's own export is running.
5. After an export, report the verify result and the warnings, and where the ZIP is (Download ZIP under step 3 of the
   Brickstatic screen, or History, Exports). The owner checks the static site from the address it is served at; you cannot.

## Forms, in one rule

Each Bricks form on an exported page chooses, in its **Brickstatic** group in the builder, where it sends from the
static site: **This WordPress site** (Bricks runs the form actions; WordPress stays online) or **Another service**
(an endpoint URL; WordPress is not involved). See `brst-forms-routes`.

## Never

- Switch the abilities on for the owner, or tell them it is harmless to; the choice is theirs.
- Touch the licence, by any route.
- Push to GitHub (`export-start` with `target: github`), roll back a release, or change the GitHub settings without
  the owner's word for that push. Never ask for the GitHub token in a chat, read it or store it. See
  `brst-github-deploy`.
- Upload to the owner's server (`export-start` with `target: server`), change the Your server settings, accept a
  server's key or confirm a folder without the owner's word for that one action. Never ask for the server password, the
  private key or its passphrase in a chat, read them or store them. See `brst-server-deploy`.
- Pass `force: true` to `export-start` on your own: it cancels the run in progress, which may be the owner's.
- Switch automatic updates on or off, change their schedule, or run `wp brickstatic auto run`, without the owner's word:
  with GitHub or your server chosen, every automatic update is a push or an upload. An update that runs on its own shows as an export in progress
  with the context `auto`; leave it to finish (the owner stops it from the Export screen). See `brst-static-export`.
- Edit Bricks' theme files, regenerate CSS files or change Bricks settings to make an export pass, without the
  owner's word.
- Say a static copy works, or that its forms send, because the export finished: the verify result checks links and
  files, not behaviour.

## Honest limits to keep in mind

Not static, whatever the export does (the preflight lists each): query filters, live search, the comments form,
logins, carts and checkout, and anything that calls `admin-ajax` or the REST API from the page. Load more and infinite
scroll pre-render up to 100 items per loop. Forms to this WordPress site need WordPress online.
Site search works for the Bricks Search element and, since 1.3.1, for another plugin's search element that plugin names
with `brickstatic/search/elements`. Since 1.4.0 a search form that is not an element (a WordPress widget, a shortcode,
the theme's) works once the owner names it with a CSS selector under **Other search forms**; without one, only a form
with an input named `s` sent to the site's home works, and only on pages where a search element renders.
The plugin's own checks drive the abilities through WordPress' REST routes
and WP-CLI, not through an MCP client. Check the pack against the plugin with `brst-update`.
