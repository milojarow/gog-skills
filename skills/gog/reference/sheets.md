# Google Sheets via `gog sheets`

`gog sheets` covers the read/write surface needed for most scripts. The full Sheets API is enormous; this file covers the operations exposed via `gog` and the semantics that bite.

## Operations

| Command | Purpose |
|---|---|
| `gog sheets get <sheetId> <range> [--json]` | Read a range |
| `gog sheets update <sheetId> <range> --values-json '<json>' --input <mode>` | Overwrite a range |
| `gog sheets append <sheetId> <range> --values-json '<json>' --insert <mode>` | Append rows |
| `gog sheets clear <sheetId> <range>` | Delete contents of a range |
| `gog sheets metadata <sheetId> [--json]` | Tabs, sheet IDs, dimensions |

## Range syntax (A1 notation)

| Range | Meaning |
|---|---|
| `Sheet1!A1` | Single cell |
| `Sheet1!A1:D10` | Rectangle, columns A-D, rows 1-10 |
| `Sheet1!A:A` | All of column A (use sparingly — can be huge) |
| `Sheet1!1:1` | Row 1 only |
| `Sheet1!A2:Z` | Columns A-Z from row 2 down to the bottom of data |
| `Sheet1` | Entire sheet (use only with `metadata` or when you're sure) |
| `'My Tab'!A1:B5` | Sheet name with spaces — wrap in single quotes |

**Sheet name must match exactly** (case-sensitive). Use `gog sheets metadata <sheetId>` to see the actual tab names.

## `--values-json` — the recommended way to pass values

`gog sheets update / append` takes `--values-json` with a JSON 2D array — outer array is rows, inner arrays are cells:

```bash
gog sheets update <sheetId> "Sheet1!A1:B2" \
  --values-json '[["Name","Age"],["Alice","30"]]' \
  --input USER_ENTERED
```

This writes:

```
       A         B
1     Name      Age
2     Alice     30
```

Mixed types are fine — wrap them all as strings; `USER_ENTERED` will type-coerce:

```bash
gog sheets append <sheetId> "Sheet1!A:C" \
  --values-json '[["2026-01-15","42","=A2*2"]]' \
  --insert INSERT_ROWS --input USER_ENTERED
```

…inserts a row where col A is parsed as a date, col B as a number, col C is a formula evaluating to 84.

## `--input USER_ENTERED` vs `RAW` — pick deliberately

| Mode | What it does |
|---|---|
| `USER_ENTERED` | Treats values as if a human typed them. `=SUM(A1:A10)` → formula. `2026-01-15` → date. `42%` → 0.42 with percentage format. **Default for most use cases.** |
| `RAW` | Stores values literally as strings. `=SUM(A1:A10)` → text "=SUM(A1:A10)". Useful when you're storing data you don't want re-interpreted. |

If you write a formula with `RAW`, it shows as text and doesn't compute. If you write `2026-01-15` with `RAW`, it stays a string instead of a date — sort/filter won't treat it as a date.

## `--insert` modes for append

| Mode | What it does |
|---|---|
| `INSERT_ROWS` | Inserts new rows, pushing existing rows down. |
| `OVERWRITE` | Overwrites rows at the append position (still detects the last row). |

`INSERT_ROWS` is usually what you want for log-style appends.

## Get with `--json`

For scripting, always `--json`:

```bash
gog sheets get <sheetId> "Sheet1!A1:D10" --json
```

The output is the same 2D array shape as `--values-json` takes — round-trippable.

## Common patterns

### Append a log row

```bash
gog sheets append <sheetId> "Log!A:D" \
  --values-json "[[\"$(date -Iseconds)\",\"user42\",\"login\",\"ok\"]]" \
  --insert INSERT_ROWS --input USER_ENTERED
```

### Clear a range without removing formatting

```bash
gog sheets clear <sheetId> "Sheet1!A2:Z"
```

(Only contents are cleared — formatting, column widths, frozen rows stay.)

### Discover tab names + sheet IDs

```bash
gog sheets metadata <sheetId> --json | jq '.sheets[] | {title: .properties.title, id: .properties.sheetId}'
```

### Read a range and process it

```bash
gog sheets get <sheetId> "Sheet1!A2:C" --json \
  | jq -r '.[] | @tsv' \
  | while IFS=$'\t' read -r col_a col_b col_c; do
      echo "Row: $col_a | $col_b | $col_c"
    done
```

### Write a small matrix in one call

```bash
gog sheets update <sheetId> "Summary!A1:B4" \
  --values-json '[["Metric","Value"],["Users","1234"],["Revenue","56789"],["Churn","2.3%"]]' \
  --input USER_ENTERED
```

## Gotchas

- **Sheet name capitalization matters.** `sheet1` ≠ `Sheet1`.
- **`A:A` (whole column)** can be very large and slow — bound the range when you can.
- **Forgetting `--input USER_ENTERED`** when you wanted formulas/dates/percentages parsed.
- **`append` writes BELOW the detected last row** — if your range is `A:C` it starts after the last row that has any value in any of A, B, C. Bounding to a single column can change where it appends.
- **`--values-json` is JSON** — escape inner quotes correctly. For Bash, single-quote the outer string and use double quotes inside.
- **No streaming** — `gog sheets get` materializes the full range. For huge ranges, page yourself with sub-ranges.
