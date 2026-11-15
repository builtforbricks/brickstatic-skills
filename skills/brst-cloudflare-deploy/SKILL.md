---
name: brst-cloudflare-deploy
description: "Use when a Brickstatic static export is published straight to the owner's Cloudflare account: the Cloudflare card (account ID, Worker name, API token, site address), Test connection, a Worker that already exists, what a publish sends and removes, the workers.dev address and a custom domain, redirects and the 404 page, Cloudflare's limits, failures and retries, automatic updates to Cloudflare, and the WP-CLI cloudflare commands."
---

# Brickstatic: the Cloudflare target

Read `brst-start-here` first. Since Brickstatic 1.9.0 the Cloudflare target publishes each export to a Worker on the
owner's own Cloudflare account, with no repository in between. The Worker holds only the export's files (static
assets, no code). Only new and changed files are sent. ZIP stays available for every run.

**Every publish and every settings change needs the owner's word for that one action.** Never ask for the API token in
a chat, never read it, never store it. Confirming a Worker that existed before Brickstatic is the owner's call: a
publish replaces what it serves, its code included.

## The Cloudflare card

On **Brickstatic > Export**, the **Settings** tab, beside Your server. Option `brst_cloudflare`.

| Label | Stored as | Notes |
|---|---|---|
| Account ID | `account` | the 32 characters Cloudflare's dashboard shows as Account ID (Workers & Pages); a pasted dashboard address is reduced to it |
| Worker name | `worker` | lowercase letters, digits and dashes, 63 at most; the first export creates the Worker; part of its workers.dev address |
| API token | `token_enc` | write-only; "A token is stored, encrypted. Leave the field empty to keep it." One permission is enough: Account, Workers Scripts, Edit, for that account alone |
| Site address | `site_url` | optional: the owner's own domain once attached to the Worker; only the link after an upload |

Then **Save Cloudflare settings** and **Test connection**. The Export card's target picker offers Cloudflare once the
account ID, the Worker name and a token are saved; until then it shows the reason and a Set up Cloudflare link.

The token is stored encrypted with the site's secret keys, like the GitHub token, and never reaches the browser, a log
or the run record. Changing the account ID or the Worker name forgets the confirmation of an existing Worker.

## Test connection

It checks the token and the account, opens an upload session and leaves it (so a token that may only read is caught;
nothing is uploaded or deployed), and looks for the Worker:

- "No Worker named ... exists yet: the first export creates it." When the account has no workers.dev address yet, the
  test says so: the owner opens Workers & Pages in Cloudflare's dashboard once to get one, or attaches a domain later.
- "The Worker ... holds the last export.", with the addresses it answers at (its custom domains and its workers.dev
  address).
- "A Worker named ... already exists, and Brickstatic did not make it." An export then stops with the same sentence
  until the owner clicks **Use this Worker for the static site** (or `--confirm-worker`), or chooses another name.
- "Cloudflare refused the API token": the token is wrong, expired or revoked. "Cloudflare refused: the API token cannot
  reach this account, or it lacks the permission Account, Workers Scripts, Edit": the account ID or the token's
  permissions.

## What a publish does

After the archive is built:

- Every publish names the whole site's files to Cloudflare, which answers the ones it lacks; only those are uploaded.
  The new list replaces the old one in one step. An unchanged site sends nothing and makes no deployment ("Nothing
  changed since the last upload to Cloudflare, Worker ...").
- A page the export no longer has is gone as soon as the export is published. An export in which pages failed, and a
  page update that does not hold the whole site, keep every other file of the last upload: nothing is removed then.
- An address without its last slash is sent on to the one with it, an unknown address gets the export's `404.html`, and
  the export's `_redirects` is applied by Cloudflare (a `_headers` file in the export's root too). Neither is published
  as a file. Cloudflare takes 2,000 plain redirect rules and 100 with a wildcard; the run's log says when rules were
  left out.
- A new Worker's workers.dev address is switched on once, after its first publish:
  `https://<worker name>.<account's name>.workers.dev/`. A Worker that existed keeps its switch as the owner set it.
- The result row: "Published to Cloudflare, Worker ...: N files new or changed, M removed." with **View the site**.
  The ZIP stays in History. The Releases card stays GitHub's: there is no roll back on Cloudflare from Brickstatic.
- This site keeps a record of the last publish (option `brst_cloudflare_manifest`). When the Worker was deployed from
  somewhere else since, the next export publishes the site again.

**A custom domain** is attached by the owner in Cloudflare's dashboard: the Worker, Settings, Domains & Routes, Add,
Custom domain. The domain must be on Cloudflare (its DNS managed there). Brickstatic attaches nothing; the owner
enters the domain as **Site address** for the link.

**A copy of the site** (a staging clone of the database) does not publish: the settings remember the site address they
were saved on, and the target says so until they are saved again on the copy.

## Limits and failures

- Cloudflare's limits: 25 MB a file (an export with a larger file stops before anything is sent and names it), 20,000
  files in one site on the Free plan and 100,000 on the Workers Paid plan (above the plan's number Cloudflare refuses
  the upload, in its own words).
- A dropped connection, an error on Cloudflare's side or a rate limit waits and tries again. Five failures of the same
  step in a row stop the run with "Cloudflare failed the same step 5 times in a row..."; the next export carries on and
  does not send the files already uploaded.
- "This page update holds only its own pages, and this site has no record of a whole export on this Worker": run a
  whole-site export to Cloudflare first.

## Automatic updates

**Settings > Automatic updates > Where updates go**: GitHub, Your server, Cloudflare or A ZIP in History (`target`).
An update to Cloudflare sends only the changed files. A choice that is not set up falls back to a ZIP, and the card
says so.

## WP-CLI

| Command | Does |
|---|---|
| `wp brickstatic cloudflare set [--account=<id>] [--worker=<name>] [--site-url=<url>]` | saves the card's settings; options left out keep their value |
| `wp brickstatic cloudflare set-token [--remove]` | the API token, read from STDIN; never an argument. Run `cloudflare set` first |
| `wp brickstatic cloudflare test [--confirm-worker] [--format=table\|json]` | the test; `--confirm-worker` allows a Worker that existed before Brickstatic |
| `wp brickstatic cloudflare show` | the settings, never the token |
| `wp brickstatic export --target=cloudflare` | the export, then the publish |

The owner runs `cloudflare set-token` in their own terminal. You may suggest the command; never run it with a token
you were given.

## Measured, and not

At 1.9.0 the target was measured on a real Cloudflare account (a first publish, an unchanged export, changed and
removed pages, a page update, redirects, an existing Worker) and against a local server that answers as Cloudflare's
API was measured to. Not measured: a site near the plan's file limit, and a token that holds only the one permission
named above (the test token held one more).
