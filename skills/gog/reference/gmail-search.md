# Gmail search syntax for `gog gmail search`

`gog gmail search '<query>'` accepts the **same operators as the Gmail web UI**. The full reference is at https://support.google.com/mail/answer/7190; this file collects the operators that matter for scripting.

## Core operators

| Operator | Meaning | Example |
|---|---|---|
| `from:` | Sender | `from:alice@example.com` |
| `to:` | Recipient | `to:bob@example.com` |
| `cc:` / `bcc:` | CC / BCC | `cc:team@example.com` |
| `subject:` | Subject contains | `subject:invoice` |
| `label:` | Has label | `label:work` |
| `category:` | Inbox category | `category:promotions` |
| `is:` | Status flag | `is:unread`, `is:starred`, `is:important` |
| `in:` | Folder/location | `in:inbox`, `in:sent`, `in:trash`, `in:spam`, `in:anywhere` |
| `has:` | Has attribute | `has:attachment`, `has:nouserlabels`, `has:drive` |
| `filename:` | Attachment filename / extension | `filename:pdf`, `filename:report.xlsx` |
| `larger:` / `smaller:` | Size | `larger:10M`, `smaller:500K` |
| `newer_than:` / `older_than:` | Relative date (`d`/`m`/`y`) | `newer_than:7d`, `older_than:1y` |
| `after:` / `before:` | Absolute date (YYYY/MM/DD) | `after:2026/01/01`, `before:2026/06/30` |
| `list:` | Mailing list | `list:devs@example.com` |
| `OR` | Boolean OR (UPPERCASE) | `from:alice@x.com OR from:bob@y.com` |
| `AND` | Implicit; just use space | `from:alice subject:meeting` |
| `-` | Negate (prefix) | `-from:newsletter@example.com` |
| `( )` | Group | `(from:alice OR from:bob) is:unread` |
| `"…"` | Exact phrase | `"quarterly review"` |

## Common patterns

### Unread mail in the last week

```bash
gog gmail search 'is:unread newer_than:7d' --max 50
```

### Anything from a specific sender, ever

```bash
gog gmail search 'from:invoice@vendor.example' --max 100
```

### Mail with PDF attachments in the last 30 days

```bash
gog gmail search 'has:attachment filename:pdf newer_than:30d' --max 20
```

### Search a specific label, excluding promotions

```bash
gog gmail search 'label:projects -category:promotions newer_than:14d' --max 50
```

### Sent mail to a specific person this month

```bash
gog gmail search 'in:sent to:client@example.com after:2026/01/01' --max 50
```

### Unsubscribe candidates — newsletters cluttering the inbox

```bash
gog gmail search 'in:inbox is:unread (subject:newsletter OR list:*) older_than:30d' --max 100
```

### Big files

```bash
gog gmail search 'larger:25M' --max 20
```

### Starred + important

```bash
gog gmail search 'is:starred is:important newer_than:90d' --max 50
```

### Search exact phrase in subject

```bash
gog gmail search 'subject:"Q4 board deck"' --max 10
```

## Output

By default the output is human-friendly. For scripting, add `--json`:

```bash
gog gmail search 'is:unread newer_than:1d' --max 50 --json | jq '.[] | {from: .from, subject: .subject}'
```

## Multi-account

```bash
gog gmail search 'is:unread' --max 10 --account you-work@gmail.com
gog gmail search 'is:unread' --max 10 --account you-personal@gmail.com
```

Or:

```bash
export GOG_ACCOUNT=you-work@gmail.com
gog gmail search 'is:unread' --max 10
```

## Gotchas

- **`OR` and `AND` are case-sensitive** — lowercase `or` is treated as a literal word.
- **Date operators use `YYYY/MM/DD`** (slashes), not ISO 8601.
- **`newer_than:`/`older_than:`** units are `d` (days), `m` (months), `y` (years). No spaces.
- **Quotes for phrases** — without them, `quarterly review` searches for *both* words anywhere, not the phrase.
- **`-` is for negation only when it prefixes an operator or word** — it's not subtraction. Use `-from:x@y.com` to exclude.
