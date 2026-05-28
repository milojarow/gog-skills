# gog-skills

**Drive [gog](https://gogcli.sh), the Google Workspace CLI, from Claude Code.**

## What is this?

A Claude Code marketplace with one skill, `gog`. It teaches Claude how to use the `gog` CLI to read and modify Google Workspace data from the terminal — Gmail, Calendar, Drive, Contacts, Sheets, Docs — including the OAuth setup, multi-account selection, and the keyring/env-source gotcha that bites every new shell.

### Why this skill exists

- **`gog` doesn't auto-source its keyring env file.** Every fresh shell needs the env exported (or `gog` won't find credentials). The skill makes this the first thing it tells you.
- **OAuth client setup is a one-time Google Cloud Console dance** — easy to forget the steps a year later. The install reference captures the exact path: create Desktop client → download `client_secret.json` → `gog auth credentials` → `gog auth add`.
- **Sheets `--input USER_ENTERED` vs `RAW`** changes whether `=SUM(A1:A10)` is a formula or literal text. The skill calls this out where it bites people.
- **Gmail search uses the same operators as the Gmail web UI** (`newer_than:`, `from:`, `label:`, …) — the reference file collects the useful ones for scripting.
- **Docs supports export/cat but NOT in-place edit** — important to know before you try to script a doc update.

## The skill

| Skill | Description |
|-------|-------------|
| **gog** | Drive `gog` for Google Workspace — Gmail/Calendar/Drive/Contacts/Sheets/Docs. Covers install, OAuth setup, multi-account, scripting flags, Gmail search syntax and Sheets value semantics. |

## Installation

Add this marketplace in Claude Code:

```
/plugin → Marketplaces → Add Marketplace → milojarow/gog-skills
```

Then install:

```
/plugin → Discover → gog-skills → Install
```

## Requirements

- The **gog** CLI on `$PATH` (see `skills/gog/reference/installation.md` — install via Homebrew or prebuilt binary, then OAuth setup against Google Cloud Console).
- A Google account with the services you want to use enabled.
- OAuth credentials (Desktop application client) from Google Cloud Console — one-time setup.

## License

MIT
