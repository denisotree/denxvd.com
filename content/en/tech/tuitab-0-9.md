---
title: tuitab 0.9 — MCP Server, Editable Databases, and Window Functions
date: 2026-08-18T12:00:00
Description: "In two months tuitab went from 0.4 to 0.9: an MCP server, SQLite and DuckDB tables you can edit and write back from the terminal, JSON/YAML/TOML opened as a real document instead of a flattened table, window functions, and a form for adding a row. Here’s what changed and why each piece matters in everyday work."
tags:
  - tuitab
  - mcp
  - rust
  - sqlite
  - duckdb
  - bash
  - linux
  - macos
  - review
Categories:
featured_image: "/tech/tuitab-0-9/tuitab-0-9.jpg"
featured_image_caption: ""
summary: "In two months tuitab went from 0.4 to 0.9: an MCP server, SQLite and DuckDB tables you can edit and write back from the terminal, JSON/YAML/TOML opened as a real document instead of a flattened table, window functions, and a form for adding a row. Here’s what changed and why each piece matters in everyday work."
DisableComments: true
draft: false
---

![tuitab: a running total per region, computed by a window function right in the table](/tech/tuitab-0-9/tuitab-0-9.jpg)

[tuitab](https://github.com/denisotree/tuitab) is a fast browser and editor for tabular data, written in Rust and running in the terminal: CSV, TSV, JSON, Parquet, Excel, SQLite, and DuckDB, with filters, pivots, JOINs, and charts all in one place. Two months ago I [wrote about it]({{< relref "tuitab" >}}) at roughly version `0.4.x` and ended by promising an MCP server — so you could plug an agent into your data and have tuitab’s fast local engine do all the counting. I closed that one by `0.6.0`, and spent four more releases building out and fixing whatever I tripped over in my own work. `0.9.4` is out now, and the distance from what I showed in that article adds up to another article. Here’s what changed: the MCP server, editable databases, documents instead of flattened tables, window functions, and a form for the new row.

P.S. This continues the [first article]({{< relref "tuitab" >}}) — if you haven’t met tuitab yet, start there: install, navigation, and the basics.

---

### 1. The MCP Server: tuitab as an Engine for a Language Model

This was the big idea from the last article, and it’s still my favorite part. A model is bad at arithmetic, and over a hundred-thousand-row file it simply makes numbers up. So the engine should count and the model should ask the question. `tuitab --mcp` starts an [MCP](https://modelcontextprotocol.io) server on top of exactly the same code the terminal interface runs.

```bash
# Register tuitab as an MCP server in Claude Code
claude mcp add tuitab -- tuitab --mcp
```

The agent gets four tools: `tuitab_inspect` (which sheets and tables a file holds, the real column names, their types, the row count), `tuitab_query` (a pipeline of `filter`, `select`, `sort`, `compute`, `group_by`, `frequency`, `pivot`, `join`, `limit`), `tuitab_describe` (the per-column profile behind the `I` key), and `tuitab_jq` (jq programs over nested JSON, JSONL, YAML, and TOML).

What matters here is that **MCP carries no SQL and no arbitrary code**. The model sends structured operations, each mapping onto a function tuitab already had, and gets back numbers computed by the Polars engine (all the maths runs on it). There is nothing for it to invent — it can only ask badly, and that shows up in the answer.

In `0.9.2` a source may also be a glob pattern, and markdown with frontmatter became a table: a page is one row, its frontmatter fields are columns, and the page text is a `body` column. Together that means my whole blog answers a single call.

```json
{"source": {"path": "content/**/*.md"},
 "ops": [{"filter": [{"col": "tags", "op": "contains", "value": "visidata"}]},
         {"select": ["file", "title"]}]}
```

While writing this article that call came back in a fraction of a second: eight pages, four Russian and four English.

Patterns live in the MCP server for now — they arrive in the terminal in `0.9.5`. By the time you read this it has most likely shipped.

`0.8.0` added writing tools too, behind a dedicated flag and a handshake. Start the server with `--mcp-write` and `tuitab_write` tells you exactly what would change: the SQL, the affected rows as they stand, and a plan id. It **writes nothing**. `tuitab_write_apply` writes — that plan, in one transaction — or refuses, if the table changed between the two calls. Without the flag those tools don’t exist at all. That is what keeps a change consistent.

---

### 2. SQLite and DuckDB Tables Became Editable

A database used to be a one-way street: you could look, not fix. The sheet took your edits and there was nowhere to put them. Now `Ctrl+S` onto the file the table came from derives the change set — `UPDATE` for edited cells, `DELETE` for removed rows, `INSERT` for pasted ones — shows every statement in full, and runs them in one transaction only after you confirm.

![Editing a SQLite table in tuitab: the confirmation shows the exact SQL before anything is written](/tech/tuitab-0-9/database.gif)

And then the parts I actually built this for:

1. **Rows are addressed by `rowid`, not by position.**
   Before anything is written, the affected rows are re-read inside the same transaction and compared against the values as loaded. If the table changed underneath you — another tool, another tuitab sheet — the save is refused with what differs, and nothing is written.
2. **Schema changes reach the database too.**
   Adding a column (`zi`, `=`, `zx`), dropping one (`zd`), renaming one (`ze`) all become `ALTER TABLE` statements in the same confirmation and the same transaction. Any of those used to block the save outright.
3. **A database can be built from nothing.**
   `tuitab inventory.sqlite` on a file that doesn’t exist opens a blank sheet: add columns with `zi`, give them types with `t`, type the rows, and `Ctrl+S` creates a real typed table. Works for `.duckdb` too, which had no writer at all before.
4. **NULL became visible.**
   The database loaders used to turn NULL into an empty string, which made the two indistinguishable on screen and quietly converted one into the other on the way back. A NULL cell now renders as a dim italic `NULL`, opening it for editing shows `\N`, and typing `\N` means SQL NULL.
5. **The `.db` extension is no longer guessed at.**
   A file used to be tried as SQLite and then as DuckDB. Every reader and writer now asks the file’s own header, so a DuckDB database called `data.db` opens in the terminal and over MCP alike.

Errors are written in sentences, not in error codes. If an operation made an in-place write impossible — a pivot, a JOIN, a transpose — the save is refused with the reason and a pointer to saving elsewhere. Saving into a different `.db` copies the whole database first (`VACUUM INTO` for SQLite) and applies the changes to the copy.

---

### 3. JSON, YAML, and TOML Are a Document, Not a Flat Table

Since `0.5.0` the structured formats open as a table **over the real document**, not over a flattened copy of it. Editing a cell writes into the tree, saving re-serialises it, and nesting, key order, and TOML datetimes all survive. A TOML file saved back as TOML keeps its comments and layout as well — it’s written through its own source rather than rebuilt.

![JSON, YAML, and TOML in tuitab: diving into a document node and converting between formats](/tech/tuitab-0-9/tree.gif)

`Enter` dives into the node of the current row, `q` comes back, and sheets in one dive chain share a single document — an edit three levels down is still there when you return, and `U` undoes it at any level. `m` cycles how a node is projected: records, key/value, scalars — an object reads far better as pairs than as one very wide row.

Converting is just a different extension on save: `config.toml` saved as `config.yaml`. If the conversion can’t carry everything, tuitab says so in the status line instead of silently dropping it. Arrow / Feather (`.arrow`, `.feather`, `.ipc`) arrived along the way, for reading and writing.

---

### 4. Window Functions, Compound Sort, and `and` / `or` in Expressions

`zw` adds a column computed from the rows around each row: `row_number`, `rank`, `dense_rank`, `cum_sum`, `lag`, `lead`, a group’s `sum` / `avg` / `min` / `max` / `count` repeated on its rows, and `pct_of_total`.

![Window functions in tuitab: a running total by date, scoped to each region](/tech/tuitab-0-9/window.gif)

There’s one detail here I reworked on purpose. A running total asks **which column orders the rows** — and the table itself is not re-sorted. Totalling by date used to mean sorting by date first, a change to the table nobody asked for. Now the rows are put in that order to compute the column, and the answer comes back where the rows already are.

Also worth having in expressions:

1. **`and`, `or`, and `not`**
   `department == "HR" or department == "Marketing"` works wherever an expression does — including the `|` row filter. Plus `contains(col, regex)` for matching a substring by regex.
2. **Compound sort with `z[` / `z]`**
   Sort by the first column as usual, move to the second, and add it as a further key. Chaining two plain sorts does not do this: the second is free to reorder rows that tie on its own key.
3. **Backticks for names with spaces**
   `` `Amount due` * 1 `` — such a column could not be referred to at all before, and double quotes *parsed*, silently producing a column of nulls, because a quoted string is a text literal.

---

### 5. A New Row Goes In Through a Type-Checked Form

`0.8.0` brought `o` and `O` — an empty row below and above the cursor, vim-style; until then the only way to add a row was pasting from the clipboard. And in `0.9.3` I turned `O` into a form: one field per column, generated from the sheet itself and labelled with the column’s type. `o` is unchanged — a blank row below the cursor.

![The new-row form in tuitab: one field per column, each value checked against its column](/tech/tuitab-0-9/rowform.gif)

Each value is checked against its column as it’s typed: a value the column cannot hold is marked red with the reason under the fields, and `Enter` refuses to insert until it’s fixed. This is the first insert path that asks the question at all — editing a cell with `e` used to answer an impossible value by turning the whole column into text and saying “Cell updated”, so an `Int64` column loaded from Parquet could become strings without a word.

A field left blank is NULL rather than an empty string. And because a blank field more often means “I’m still filling this in” than a deliberate NULL, the first `Enter` says how many are blank and waits for a second one.

---

### 6. Excel Finally Does Arithmetic

Not a feature, a repair — but it unlocked a whole workflow that simply didn’t work before.

1. **Every number in a spreadsheet was text.**
   Each cell reached tuitab through its string rendering, so a column of money came out `string` and `sum` over it answered “the column is string, and sum needs a numeric one”. An `.xlsx` source was unusable for the exact arithmetic the tool exists to do. A column is now offered to `Int64` and then `Float64`, and keeps its text only when some cell is not a number. Whole numbers stay whole: an id doesn’t come back as `1.00`.
2. **A date came back as its serial number.**
   Excel stores a date as a count of days with a format hung on it, and tuitab read the count: the 29th of January 2026 arrived as `46051`. It’s now written the way every other reader here writes a date — `2026-01-29`, or `2026-01-29 14:30:00` when the cell carries a time.
3. **A NULL saved to `.xlsx` was written as the word `null`.**
   It read back as a perfectly good label and turned the whole column into text. It’s a blank cell now.

---

### 7. Other Small Things I Found Along the Way

1. **Copying left the clipboard empty on Linux.**
   On X11 and Wayland the text isn’t stored anywhere — it’s served by the process that set it, over that very connection. tuitab opened a connection per copy and closed it on the way out, so the status line cheerfully said “copied” and there was nothing to paste, in tuitab or anywhere else. It now holds one clipboard for the life of the process. (macOS was never affected.)
2. **`y c` copied the cell as it is drawn.**
   A float column is shown to two decimals, so `1234.5678` yielded `1234.57` — invisible until you pasted the value back into a table. The single-cell copy takes the raw value now.
3. **Pinning a column with `!` no longer moves it.**
   Pinning used to walk the column to the front one swap at a time. Two consequences: unpinning restores the original order, and keeping a column in sight while scrolling is no longer indistinguishable from asking for your columns to be reordered.
4. **A negative currency couldn’t be typed or pasted back.**
   It’s displayed in brackets — `($5.00)` — and was read back through a filter keeping only digits, a dot, and a minus, so the brackets went and the value came back positive.
5. **Keyboard shortcuts didn’t work on a non-Latin layout.**
   Layout translation lived in a fallback at the bottom of the key handler and never saw the second key of a shortcut at all, so `zw`, `gb`, `z[` and `z]` could not be typed. Translation now happens once, before anything reads the key.

---

### How to Update

```bash
# Cargo: rebuild and install over the old one
cargo install tuitab --force

# macOS / Linux via Homebrew
brew upgrade tuitab

# Arch Linux
yay -Syu tuitab
```

Pre-built binaries for Linux and macOS (x86_64 / aarch64) are on the [releases page](https://github.com/denisotree/tuitab/releases). The documentation now lives in [`docs/`](https://github.com/denisotree/tuitab/tree/master/docs): keybindings, expressions, working with databases, and a page on MCP, in English and Russian.

---

### Post Scriptum

The overall principle hasn’t changed: tuitab keeps data in memory. On a modern machine that’s comfortable for files of a few gigabytes, and if you’re dealing with tens of them, that’s a job for scripts with distributed lazy evaluation. And yes, this is still a young project: `0.9.4` is not `1.0`, and something somewhere will certainly behave differently than you expect. Which is why I need your help to make tuitab better — I’ll be endlessly grateful for bug reports, pull requests, and simply for you giving tuitab a try in your own work.

Next up: separate articles walking through the scenarios tuitab is for. Analysing data together with an agent over MCP, say, and working with SQLite / DuckDB databases straight from the terminal.

### Feedback

If you have ideas for improvements or you found a bug — open an [issue on GitHub](https://github.com/denisotree/tuitab/issues) or message me (LinkedIn / Instagram / Telegram).
