---
name: gog
description: Use when reading or modifying Google Workspace data from the terminal via the `gog` CLI — Gmail (search, send), Calendar (events), Drive (search), Contacts, Sheets (get/update/append/clear), Docs (export/cat) — including OAuth client setup, multi-account selection and JSON output for scripting.
---

# gog — Google Workspace CLI

Drive [gog](https://gogcli.sh) to read and modify Gmail, Calendar, Drive, Contacts, Sheets and Docs from the shell.

> **🌐 ACTIVE-SKILL MARKER:** While `gog` is active, begin every reply with 🌐 so the operator sees at a glance that this skill is engaged. Do not omit it.

## Overview

`gog` is a single Go binary that wraps the Google APIs for six services (Gmail, Calendar, Drive, Contacts, Sheets, Docs) behind a uniform subcommand interface. Auth is OAuth2 (your own Google Cloud client + per-account browser flow). One install can hold many Google accounts; commands take `--account <email>` (or a `GOG_ACCOUNT=` env var) to pick one.

## When to use

- Searching, sending Gmail from the terminal.
- Reading calendar events (per calendar, date range).
- Searching Drive; reading file metadata.
- Listing contacts.
- Reading and writing Google Sheets (get/update/append/clear, including `--values-json` payloads).
- Exporting / catting Google Docs.
- Building scripts that talk to Google Workspace without writing a full Google API client.

**Not for:** Google Workspace admin console (user provisioning, group management, audit logs) — those need the Admin SDK, which `gog` does not cover. This skill is for **per-user** Workspace data.

## Prerequisites

- The `gog` binary on `$PATH`. Verify with `gog --version`.
- OAuth credentials configured (one-time): a Desktop OAuth client from Google Cloud Console + `gog auth credentials` + `gog auth add <email>`.
- The Google services you want to use must be enabled on the account.

If `gog` isn't installed (or OAuth isn't set up yet), see [reference/installation.md](reference/installation.md) for the full one-time path.

## ⚠️ Auth setup gotcha — sourcing the keyring each shell

If you store the OAuth keyring path / encryption key in an env file (a common pattern is `~/.config/gogcli/keyring.env`), **that file is NOT auto-sourced**. Every fresh shell starts blind to it; running `gog gmail search …` will fail with `not authenticated` or similar.

**Fix:** before any `gog` command in a new shell, source the env file:

```bash
source ~/.config/gogcli/keyring.env
```

…or add the source line to your `~/.zshrc` / `~/.bashrc` so it's automatic. The skill assumes this is done; if you see auth errors, this is the first thing to check.

(If you didn't set up a keyring env file at all — i.e. you let `gog auth add` write to its default location — you may not need this at all. The gotcha bites only when you opted into an explicit keyring path.)

## Quick reference — one command per service

| Service | Command (typical) |
|---|---|
| Gmail — search | `gog gmail search 'newer_than:7d' --max 10` |
| Gmail — send | `gog gmail send --to a@b.com --subject "Hi" --body "Hello"` |
| Calendar — list events | `gog calendar events <calendarId> --from 2026-01-01T00:00:00Z --to 2026-01-31T23:59:59Z` |
| Drive — search | `gog drive search "query" --max 10` |
| Contacts — list | `gog contacts list --max 20` |
| Sheets — get | `gog sheets get <sheetId> "Tab!A1:D10" --json` |
| Sheets — update | `gog sheets update <sheetId> "Tab!A1:B2" --values-json '[["A","B"],["1","2"]]' --input USER_ENTERED` |
| Sheets — append | `gog sheets append <sheetId> "Tab!A:C" --values-json '[["x","y","z"]]' --insert INSERT_ROWS` |
| Sheets — clear | `gog sheets clear <sheetId> "Tab!A2:Z"` |
| Sheets — metadata | `gog sheets metadata <sheetId> --json` |
| Docs — export | `gog docs export <docId> --format txt --out /tmp/doc.txt` |
| Docs — cat | `gog docs cat <docId>` |
| Auth — list accounts | `gog auth list` |
| Auth — add account | `gog auth add you@gmail.com --services gmail,calendar,drive,contacts,sheets,docs` |

For Gmail's search query operators (`from:`, `label:`, `newer_than:`, `is:`, …): see [reference/gmail-search.md](reference/gmail-search.md).
For Sheets' range syntax, `--input` modes and batch patterns: see [reference/sheets.md](reference/sheets.md).

## Multiple accounts

If `gog auth add` has been run for several Google accounts, pick one per call:

```bash
gog gmail search 'is:unread' --max 10 --account you-personal@gmail.com
gog gmail search 'is:unread' --max 10 --account you-work@gmail.com
```

Or set the default for a whole shell:

```bash
export GOG_ACCOUNT=you-work@gmail.com
gog gmail search 'is:unread' --max 10   # uses you-work
```

`gog auth list` shows the configured accounts.

## Scripting tips

- **`--json`** — produce parseable structured output. Default human-readable output is for humans; pipe-through is `--json`.
- **`--no-input`** — non-interactive mode. Combine with `--json` for cron/scripts.
- For Sheets writes: prefer `--values-json '[["row1col1","row1col2"], …]'` over inline positional values — it's unambiguous and lets you pass typed values cleanly.

## Common mistakes

- **Forgetting to `source` the keyring env file in a new shell** — see the warning above. Symptom: `not authenticated`.
- **Sheets writes with `--input RAW` when you meant `USER_ENTERED`** — `RAW` stores `=SUM(A1:A10)` as the literal text `=SUM(A1:A10)`. `USER_ENTERED` evaluates it as a formula. Pick deliberately.
- **Trying to edit a Doc in place** — `gog docs` supports export, cat and copy, but NOT in-place edits. You need the Docs API directly (not in `gog`) for that.
- **Confusing calendar `<calendarId>` with the email address** — for a user's primary calendar it IS the email, but for shared calendars it's a long Google-assigned ID. Get it from `Settings → Integrate Calendar` in Calendar web UI, or via `gog calendar list` if available.
- **Not confirming destructive ops** — `gmail send` and creating calendar events are visible to others. Read the proposed command back to the user before invoking.
- **Assuming `--json` is enabled by default** — it's not. Without it, output is human-formatted and a pain to parse.
