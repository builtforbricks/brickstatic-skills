---
name: brst-server-deploy
description: "Use when a Brickstatic static export goes to the owner's own server over SFTP, FTPS or FTP: the Your server card (protocol, host, port, user, password or private key, folder), Test connection and accepting the SFTP server's key, a folder that already holds files, what an upload sends and removes, the manifest, failures and retries, automatic updates to the server, and the WP-CLI server commands."
---

# Brickstatic: the Your server target

Read `brst-start-here` first. Since Brickstatic 1.3.0 the Your server target uploads each export to a folder on the
owner's own hosting over SFTP, FTPS or FTP. Only new and changed files go up. ZIP stays available for every run.

**Every upload and every settings change needs the owner's word for that one action.** Never ask for the password, the
private key or its passphrase in a chat, never read them, never store them. Accepting a server's key is the owner's
call: they compare it with the key their host shows.

## The Your server card

On **Brickstatic > Export**, the **Settings** tab, beside GitHub. Option `brst_server`.

| Label | Stored as | Notes |
|---|---|---|
| Protocol | `protocol` | `sftp` (recommended), `ftps` (FTP over TLS) or `ftp` (not encrypted: the card warns) |
| Host | `host` | a name or an address; a pasted `sftp://user@host:port/folder` is reduced to its parts |
| Port | `port` | 22 for SFTP, 21 for FTP and FTPS unless the host says otherwise |
| User | `user` | the account's user name |
| Sign in with | `auth` | SFTP only: `password` or `key` |
| Password | `password_enc` | write-only; "A password is stored, encrypted. Leave the field empty to keep it." |
| Private key, Key passphrase | `key_enc`, `passphrase_enc` | write-only; OpenSSH, PEM or PuTTY format; a key made for this site alone |
| Folder on the server | `path` | relative to where the account opens (for example `public_html`), or a full path from `/`; empty for the account's own folder |
| Site address | `site_url` | optional: only the link after an upload |

Then **Save server settings** and **Test connection**. The Export card's target picker offers Your server only once the
card is complete (and, for SFTP, the server's key is accepted); until then it shows the reason and a Set up your server link.

The password, the key and the passphrase are stored encrypted with the site's secret keys, like the GitHub token, and
never reach the browser, a log or the run record. Changing the host, port, protocol or user forgets the accepted key and
the folder confirmation.

## Test connection and the server's key

- **SFTP, first test**: it reads the server's key and sends nothing else. The flash shows it as
  `ssh-ed25519 SHA256:...`; the owner compares it with the key the host shows (control panel or support) and clicks
  **Accept this key**, then tests again. From then on every connection compares the key before the password is sent or
  the key signs anything (`fingerprint`, `key_type` on the card).
- A **changed key** stops every upload with "The server's key changed..." and nothing is sent. Only the owner decides to
  accept the new one, after checking with the host.
- The test then logs in, checks the folder, writes and removes a small test file, and says what the folder holds: empty,
  the last upload (`manifest_files`), or files Brickstatic did not upload.
- **FTPS** encrypts the login and the files, but PHP does not check the server's certificate: SFTP is the safer choice
  where the host offers it. A host without FTPS answers with a sentence that offers SFTP or plain FTP; nothing is sent in
  the clear by itself.

## What an upload does

After the archive is built:

- It compares the export with `.brickstatic-manifest.json` in the folder, the list of what Brickstatic uploaded (with a
  copy on the site, option `brst_server_manifest`). The server's files are never hashed. Without the list on the server
  (the folder emptied or restored from a backup), every file goes up again.
- New and changed files go up: files first, then pages, the home page last. An unchanged site sends nothing ("Nothing
  changed since the last upload to ...").
- Files on the list that the export no longer has are removed, and the folders that leaves empty. **Nothing else in the
  folder is ever touched**: the owner's own files stay, and an edited list cannot point outside the folder, at a dot
  file or at PHP. An export in which pages failed removes nothing (a page that failed is not gone from the site), and a
  file the server refuses to delete stays listed with a warning; the next export tries again.
- Files above 4 MB go up in 4 MB pieces, so a slow link never outlasts the host's time limit; a dropped connection
  carries on where the piece stopped.
- A page update (**Update one page only** or the builder's window, choice **Your server: upload the changed files**, picked
  when GitHub is not set up; or `--only`) uploads what changed for it and removes nothing.
- The result row: "Uploaded to sftp://user@host/folder: N files new or changed, M removed." with **View the site** when
  the card has a site address. The ZIP stays in History.

**A folder that already holds files** and no list: the test says how many, and the first upload stops with "The folder
... already holds N files and folders that Brickstatic did not upload..." until the owner clicks **Use this folder for
the static site** (or `--confirm-folder`). Same-named files are then replaced; the rest stays. A file that was there
before and got replaced is listed under `keep` and never removed. The folder should be the static site's own.

**A copy of the site** (a staging clone of the database) does not upload: the settings remember the site address they
were saved on, and the target says so until they are saved again on the copy.

## Failures

- A dropped connection or a transfer the server may accept on another try waits 5, 10, 15... seconds and tries again.
  Five failures of the same step in a row stop the run with "The server failed the same step 5 times in a row..."; the
  next export carries on and does not send the files already uploaded.
- At once, with the reason: a refused password or key (`brst_server_login`), a changed key (`brst_server_hostkey`), a
  folder that does not exist (`brst_server_folder`), a server that refuses to write (`brst_server_write`, a permission or
  a full disk), FTPS refused (`brst_server_tls`), a host that does not answer (`brst_server_connect`).
- "A file of the SFTP library is missing...": a security scanner on the host may have removed a file of the bundled
  library. SFTP is off until the plugin is reinstalled; FTP and the rest of the plugin keep working.

## Automatic updates

**Settings > Automatic updates > Where updates go**: GitHub, Your server or A ZIP in History (`target`). An update to the
server sends only the changed files. Until the owner changes the select, `target` stays empty and the default rule applies
(GitHub once set up, else a ZIP). A choice that is not set up falls back to a ZIP, and the card says so.

## WP-CLI

| Command | Does |
|---|---|
| `wp brickstatic server set [--protocol=<protocol>] [--host=<host>] [--port=<port>] [--login=<login>] [--folder=<folder>] [--auth=<auth>] [--site-url=<url>]` | saves the card's settings; options left out keep their value. `--login` and `--folder`, because WP-CLI takes `--user` and `--path` itself |
| `wp brickstatic server set-secret <which> [--remove]` | `password`, `key` or `passphrase`, read from STDIN (a key to the end of STDIN); never an argument. Run `server set` with the host first: on a card with no host it says so and stores nothing (1.3.1) |
| `wp brickstatic server test [--accept-key=<fingerprint>] [--confirm-folder] [--format=table\|json]` | the test; `--accept-key` takes only the fingerprint the last test printed |
| `wp brickstatic server show` | the settings, never a secret |
| `wp brickstatic export --target=server` | the export, then the upload |

The owner runs `server set-secret` in their own terminal. You may suggest the command; never run it with a password or
a key you were given.

## Not measured

At 1.3.0 the uploads were measured against local SFTP and FTP servers and the OpenSSH server of a Mac, not against a
hosting company's servers. Say so if the owner asks how a given host behaves.
