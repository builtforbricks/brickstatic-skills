# Brickstatic skills

Agent skills for [Brickstatic](https://builtforbricks.com), the Bricks Builder plugin by Built for Bricks that
publishes a Bricks site as static HTML that keeps its forms working, as a ZIP or pushed to GitHub. The skills tell a
client how an export works, what every preflight finding means and how to fix it, how forms reach a destination from a
static page, and what a push to GitHub does and never does. Checked against the plugin before every release.

**Status: released with Brickstatic 1.1.0 (pack 0.2.0).** Each release names the Brickstatic version it was checked
against (`verifiedUpTo` in `skills/brst-update/references/index.json`) and goes out with that plugin version.

## What a skill is

A folder with a `SKILL.md`: a short front matter (`name`, `description`) and Markdown an AI client loads when the task
matches. It does not talk to the site; the connection does. Bricks 2.4 or later connects a client to the site through
the WordPress MCP Adapter, set up under **Bricks > AI**, and the plugin adds five abilities of its own once the site's
owner switches on **Brickstatic > Settings > AI agents**: a preflight, and an export driven from the server in
small steps. The skills tell the client what the abilities cannot: what export mode changes in a page, what each
finding costs the static copy, which route each form takes and what the push keeps.

## Requirements

- Brickstatic 1.0.0 or later, on WordPress 6.9 or later for its abilities.
- Bricks 2.4 or later with its abilities enabled, for a client that works on the site.
- A client that loads skills: Claude Code, Codex, Cursor, GitHub Copilot, or any client that reads `SKILL.md` folders.

## Install

Copy the prompt from the **AI agents** tile on the Brickstatic dashboard in WordPress (shown once the switch is
on) into your client, or do it by hand.

**Claude Code, as a plugin** (updates with `/plugin marketplace update brickstatic-skills`):

```
/plugin marketplace add builtforbricks/brickstatic-skills
/plugin install brickstatic@brickstatic-skills
```

**Claude Code, Codex, Cursor and others, as a checkout** (one checkout, symlinked, so an upgrade moves every skill):

```sh
git clone https://github.com/builtforbricks/brickstatic-skills.git ~/.brickstatic/skills/brickstatic-skills
~/.brickstatic/skills/brickstatic-skills/scripts/brst-skills-upgrade   # pins the checkout to the latest release

# Pick the directory your client scans:
SKILLS_DIR="$HOME/.claude/skills"      # Claude Code, every project    (a project alone: .claude/skills)
# SKILLS_DIR="$HOME/.agents/skills"    # Codex and other agents         (Codex alone: ~/.codex/skills)
# SKILLS_DIR=".cursor/skills"          # Cursor, this project

mkdir -p "$SKILLS_DIR"
for skill in "$HOME"/.brickstatic/skills/brickstatic-skills/skills/brst-*; do
  ln -sfn "$skill" "$SKILLS_DIR/$(basename "$skill")"
done
```

Then start a new chat and ask: *List the loaded skills whose names start with `brst-`.*

## What is included

| Skill | Covers |
|---|---|
| `brst-start-here` | Load first: what the plugin does, the owner's switch, the five abilities, the workflow, never do |
| `brst-static-export` | How an export works and what it changes: export mode, discovery, the targets, the destination, runs and retention, what the ZIP holds, the WP-CLI commands |
| `brst-preflight-findings` | Every preflight row by id, with its level, its consequence and its fix in the plugin's own words |
| `brst-forms-routes` | The per-form choice (This WordPress site or Another service), the form's Brickstatic group, static origins, Check forms, what each route needs and cannot do |
| `brst-github-deploy` | The GitHub card, the token's scope, the folder, releases and roll back, a page re-export, what the push never does |
| `brst-update` | Compares the pack with the plugin on the site; updates on request |

## What these skills never do

They never switch the abilities on for the owner, never touch the licence, and never push to GitHub, roll back a
release or change the GitHub settings without the owner's word for that push. They never read, store or ask for the
GitHub token in a chat: the owner pastes it under Settings, GitHub on the Brickstatic screen or pipes it to WP-CLI. They never present a static copy
as tested: the owner checks the static site, and its forms, from the address it is served at.

## How it is made, and why that matters

- Every ability, WP-CLI command, setting and preflight row a skill quotes is checked against the plugin's code before a
  release; the check fails on a name the plugin does not have. `verifiedUpTo` is the plugin version they were checked
  against, never typed.
- The plugin and the pack are released together, the same day. `scripts/brst-skills-upgrade` pins a checkout to the
  latest release; `--check` only reports.

## Updating

Ask your client to "check for Brickstatic skills updates" (the `brst-update` skill), or run
`~/.brickstatic/skills/brickstatic-skills/scripts/brst-skills-upgrade`. Start a new chat afterwards: most
clients load skills once, at the start of a session.

## Licence

GPL-2.0-or-later, the plugin's. Built for Bricks, https://builtforbricks.com.
