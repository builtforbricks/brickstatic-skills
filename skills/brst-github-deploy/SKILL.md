---
name: brst-github-deploy
description: "Use when a Brickstatic static export goes to GitHub: the GitHub card (repository, branch, folder, token, Tag every push), what the token needs, how a push commits only changed files and never force-pushes, the files it keeps, releases and roll back, a page re-export, and the WP-CLI github and releases commands."
---

# Brickstatic: the GitHub target

Read `brst-start-here` first. The GitHub target pushes each export to one branch of one repository as a single commit;
a host that builds from the repository publishes it from there. ZIP stays available for every run.

**Every push, roll back and settings change needs the owner's word for that one action.** Never ask for the token in
a chat, never read it, never store it.

## The GitHub card

On **Brickstatic > Export**, the **Settings** tab, beside Address and forms. Option `brst_github`. The
releases are under the **History** tab once GitHub is set up.

| Label | Stored as | Notes |
|---|---|---|
| Repository | `owner`, `repo` | owner/repo, for example acme/website; the repository's github.com address works too |
| Branch | `branch` | the branch each export is committed to; main by default |
| Folder in the repository | `path` | empty for the root of the repository |
| Token | `token_enc` | write-only; "A token is stored. Leave the field empty to keep it." or "No token is stored."; Remove the stored token |
| Tag every push | `tag_on_publish` | adds the tag `export-<run id>` to each commit |

Then **Save GitHub settings** and **Test connection** (enabled once a repository and a token are stored). The Export
card's target picker offers GitHub only when the card is complete; until then it shows the reason.

**The token**: a fine-grained personal access token for this one repository with Contents: read and write, and
nothing else. On a repository that belongs to an organisation, the owner approves the token first. It is stored
encrypted with the site's secret keys and never shown again; it never reaches the browser. Changing the site's secret
keys means pasting the token again. Without the sodium extension or the secret keys the card shows why and the token
field is disabled.

## What a push does

After the archive is built, the push runs in steps:

- It lists the branch's files once and uploads only the files that are new or changed under the folder.
- The files travel the way `git push` sends them: one request for most sites, packs of about 8 MB for a large one,
  parked on the temporary ref `refs/brst/export-<run id>` until the last request moves the branch and removes it. No
  REST request per file, so GitHub's limit on content-creating requests (500 an hour) does not stop a large first push.
- One commit, "Static export <date>, <n> pages" (a page re-export names the page), then the branch moves forward.
  **Never a force-push.** If the branch moved while the export ran, the run fails with a sentence that says to export
  again; nothing is overwritten.
- When nothing changed, no commit is made; the release is recorded as unchanged ("Nothing changed since the last
  release; the branch stays at <commit>.") and no tag is added.
- Files under the folder that the export no longer holds are deleted, except host files a buyer keeps there: the
  `.github/` folder, `CNAME`, `.nojekyll`, `README.md`, `README`, `LICENSE`, `.gitignore`, `.gitattributes`,
  `_redirects`, `_headers` and the hosts' own configuration files. The export overwrites one only when it holds it.
- `_redirects` is shared. The export's rules are a block from the line `# Brickstatic redirects (do not edit
  this block; the lines above it are yours)` to the end of the file; the buyer's lines above it stay on every push, a
  `_redirects` without that line keeps all its lines with the block added after them, and when the export has no
  redirects left (or Redirects is off) the block goes, the file only when nothing else is in it. A page update does this
  too.
- Files outside the folder are never touched.
- A file above 100 MB is skipped with a warning (GitHub refuses it).
- Rate limits and server errors make the step wait and try again; a refused token stops the run at once.
- **Test connection** checks the repository, the branch and git's push address with the token.
- An empty repository is fine: the first export creates the branch. A missing branch in a repository that has
  commits is refused: create it on GitHub first.

The result row says "Pushed commit <short sha> to owner/repo, branch <branch>." with **View the commit**; the CLI
prints a `release` row. A failed push keeps the ZIP.

## Releases and roll back

The **Releases** card lists every push and every roll back, newest first (the last 50, option `brst_releases`): When,
Kind (Export, Page, Roll back, "no change"), Pages, Commit, Tag, By.

**Roll back** (on rows whose files differ from the newest) makes a new commit with the files of that earlier export,
so the host publishes that state again; nothing is removed from the history. With a folder, only the folder is
restored. It opens a confirm dialog, "Roll back to this export?". A roll back to the state already on the branch makes
no commit.

## A page re-export

**Re-export this page** under step 3's **Update one page only** (or `export-start` with `only`, or `wp brickstatic export
--target=github --only=<url>`) re-renders one page and the files it uses and commits only what changed for it.

## WP-CLI

| Command | Does |
|---|---|
| `wp brickstatic github set [--repo=<repo>] [--branch=<branch>] [--folder=<folder>] [--tag\|--no-tag]` | saves the card's settings; options left out keep their value. `--folder`, because WP-CLI takes `--path` itself |
| `wp brickstatic github set-token` | the token on STDIN, one line (`pbpaste \| wp brickstatic github set-token`); a `--token` argument is refused because it would stay in the shell history |
| `wp brickstatic github remove-token` | removes the stored token |
| `wp brickstatic github test [--format=table\|json]` | the repository, the push permission and the branch head |
| `wp brickstatic export --target=github` | the export, then the push |
| `wp brickstatic releases list [--format=table\|json]` | every push and roll back |
| `wp brickstatic releases rollback <id>` | rolls the branch back to a release, as a new commit |

The owner runs `github set-token` in their own terminal. You may suggest the command; never run it with a token you
were given.

## Errors worth knowing

- "GitHub refused the token" (`brst_github_unauthorized`): wrong, expired or revoked; the owner pastes a new one on the
  card.
- `brst_github_no_push`: the token cannot write to this repository; it needs Contents: read and write on it.
- `brst_github_not_found`: the repository or the branch does not exist, or the token cannot see it.
- `brst_github_not_fast_forward`: the branch moved during the run; export again.

## Not measured

At 1.0.0-rc.5 the push was measured against a local server that answers GitHub's API with real git repositories, not
against GitHub itself, and no host's build from the repository was measured. Say so if the owner asks whether a host
publishes the result.
