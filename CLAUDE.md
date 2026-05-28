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
        └── reference/        # installation.md, gmail-search.md, sheets.md
```

## The skill

### gog
Drive the `gog` CLI for Google Workspace. `SKILL.md` is the lean entry point with the keyring-source gotcha, a 6-service quick-reference table, and cross-links to:
- `reference/installation.md` — install paths + the OAuth client setup against Google Cloud Console.
- `reference/gmail-search.md` — Gmail query operators (`newer_than:`, `from:`, `label:`, …) and common patterns.
- `reference/sheets.md` — range syntax, `--values-json` vs inline rows, `USER_ENTERED` vs `RAW`, batch operations.

## Skill Activation

Activates when reading or modifying Google Workspace data from the terminal via the `gog` CLI — Gmail search/send, Calendar events, Drive search, Contacts, Sheets CRUD, Docs export.

## Updating this skill

After any session that discovers a new wall or pattern. Keep entries **generic** — placeholder addresses and IDs only; never real accounts, sheet IDs, or doc IDs. The git log of this repo is the diary.
