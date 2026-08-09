# Reading Gmail message bodies — flattened text vs raw HTML

When you only need *what a message says*, flattened text is fine. When you need the
**relationship between fields** — date ↔ amount, label ↔ value, row ↔ column — flattened
text is the wrong input. A flattener that loses structure does not lose data; it
**fabricates relationships**, which is much harder to catch because it leaves no gap.
It leaves something that looks like confirmation.

## The failure mode

An HTML message containing **two separate tables** gets flattened into one stream of
text. A standalone field from the first table ends up textually adjacent to values
from the second, and the adjacency reads as meaning:

```
payment day: Friday          <- a loose field from table 1
2026-08-01                   <- first row of table 2 (a different table)
2026-08-08
```

The natural reading — "payments fall on Friday the 1st, 8th, …" — is false. Those
dates are Saturdays, and the calendar table had **no weekday column at all**. Nobody
misread anything; the flattening invented the link.

## The tripwire that catches it

**A field that contradicts itself within one line.** Above: the weekday name says
Friday, the date is a Saturday. When the two halves of a single field disagree, do
not argue about which one wins — **go back to the raw HTML**. It almost always means
the two values came from different places in the document, or the template was
filled incorrectly.

Checking the weekday of a date costs one command:

```bash
date -d "2026-08-01" '+%A'      # Saturday
```

```bash
for d in 01 07 08 14 15; do echo "$d: $(date -d "2026-08-$d" '+%A')"; done
```

## Procedure

1. **If the relationship between fields matters, extract the HTML, not the plain
   text.** Before relying on a flattened body, confirm which body representations
   your read path actually exposes — check the subcommand's own `--help` rather
   than assuming a text body is the only option.
2. **Rebuild the table from its structure** — `<thead>` / `<tr>` / `<td>` — never
   from the position of text on the page.
3. **Confirm with a third, independent signal.** In the case above it was arithmetic
   cadence (first interval +8 days, then +7 exactly) reproducing every date. Three
   independent weak proofs beat one strong one.

## Do not trust template semantics

In the same message the `<thead>` `id` attributes were **crossed** relative to the
visible text:

```html
<th id="fecha">Importe</th>
<th id="monto">Fecha</th>
```

If the generator got that wrong, any attribute can be wrong. Trust the **values of
the visible cells**; never the `id`s or `class`es that claim to describe them.

## The general rule

A source that describes a **state** does not describe a **property**, and an extractor
that loses structure **fabricates relations**. That second failure is the dangerous
one — it produces something that looks like evidence.
