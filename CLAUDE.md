# CLAUDE.md

This file provides guidance to Claude Code when working with this repository.

## Project Overview

This is the **gog-skills** repository — a Claude Code marketplace that ships one skill for driving the [gog](https://gogcli.sh) Google Workspace CLI.

**Repository**: https://github.com/milojarow/gog-skills

## Repository Structure

```
gog-skills/
├── .claude-plugin/          # marketplace.json + plugin.json
├── CLAUDE.md                # This file
├── README.md                # Project overview
├── LICENSE                  # MIT
├── evaluations/             # Test scenarios for the skill
└── skills/
    └── gog/
        ├── SKILL.md          # Entry point (lean)
        └── reference/        # installation.md, gmail-search.md, gmail-message-bodies.md, sheets.md
```

## The skill

### gog
Drive the `gog` CLI for Google Workspace. `SKILL.md` is the lean entry point with the keyring-source gotcha, a 6-service quick-reference table, and cross-links to:
- `reference/installation.md` — install paths + the OAuth client setup against Google Cloud Console.
- `reference/gmail-search.md` — Gmail query operators (`newer_than:`, `from:`, `label:`, …) and common patterns.
- `reference/gmail-message-bodies.md` — flattened text vs raw HTML when field relationships matter; the self-contradicting-field tripwire; don't trust template `id`/`class` attributes.
- `reference/sheets.md` — range syntax, `--values-json` vs inline rows, `USER_ENTERED` vs `RAW`, batch operations.

## Skill Activation

Activates when reading or modifying Google Workspace data from the terminal via the `gog` CLI — Gmail search/send, Calendar events, Drive search, Contacts, Sheets CRUD, Docs export.

## Known gap: how to get a RAW HTML body out of `gog gmail`

`reference/gmail-message-bodies.md` teaches *why* you must read the raw HTML when the
relationship between fields matters, but it deliberately does **not** name a `gog` flag
for obtaining it — the exact subcommand/flag was never verified against the binary, and
the skill currently documents no message-read command at all (only `search` and `send`).

To close this: run `gog gmail --help` and the read subcommand's `--help`, confirm which
body representations are exposed (text / HTML / raw MIME), then add the concrete
invocation to step 1 of that reference. **Do not write a flag into the skill until it has
been confirmed against the installed binary** — a documented flag that doesn't exist
teaches every future session to run a failing command.

## Updating this skill

After any session that discovers a new wall or pattern. Keep entries **generic** — placeholder addresses and IDs only; never real accounts, sheet IDs, or doc IDs. The git log of this repo is the diary.
