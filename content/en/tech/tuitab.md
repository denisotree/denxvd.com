---
title: tuitab — My Terminal Tool for Tables, a VisiData Alternative
date: 2026-06-19T12:00:00
Description: "I built tuitab — a fast terminal explorer for tabular data, written in Rust: it opens CSV, JSON, Parquet, Excel, SQLite, and DuckDB, and does filtering, sorting, pivot tables, JOINs, computed columns, and charts right in the terminal. I explain why I moved on from VisiData to my own tool, what tuitab can do, and how to install it."
tags:
  - tuitab
  - visidata
  - rust
  - bash
  - linux
  - macos
  - review
Categories:
featured_image: "/tech/tuitab/tuitab.jpg"
featured_image_caption: ""
summary: "I built tuitab — a fast terminal explorer for tabular data, written in Rust: it opens CSV, JSON, Parquet, Excel, SQLite, and DuckDB, and does filtering, sorting, pivot tables, JOINs, computed columns, and charts right in the terminal. I explain why I moved on from VisiData to my own tool, what tuitab can do, and how to install it."
DisableComments: true
draft: false
---

![tuitab showing a CSV table in the terminal with column type icons](/tech/tuitab/tuitab.jpg)

[tuitab](https://github.com/denisotree/tuitab) — a fast, powerful browser for tabular data that runs right in your terminal. Written in Rust. It works with CSV, TSV, JSON, Parquet, Excel, SQLite, and DuckDB out of the box (and I can write new connectors on request) and lets you filter, sort, build pivot tables, run JOINs, compute new columns, and draw charts without taking your hands off the keyboard. I built it first and foremost for myself, because I love data analysis and I love the terminal. I love being able to dive into data fast, look at every segment I care about, and test simple hypotheses — without spinning up Jupyter or writing a million SQL queries.

I’ve already [written]({{< relref "visidata" >}}) that VisiData is one of my favorite tools, and I even [made my own fork]({{< relref "visidata-ng" >}}) to add the features I was missing faster. But at some point I hit a ceiling and decided to build my own from scratch. In this article I’ll explain why that happened, what tuitab can do, and how you can give it a quick try.

---

### Why I Didn’t Stop at a VisiData Fork

In my [visidata-ng]({{< relref "visidata-ng" >}}) fork I bolted on the things I was missing: exit confirmation, a quick “percent of total,” currency formatting, and flexible date formats. But the longer I patched someone else’s code, the clearer two things became:

1. **The Python environment is a pain.** To open Parquet you need `pyarrow`, Excel needs a couple more packages, and all of it tends to clash between `pip`, `brew`, and the system Python. I’ve [written about these install caveats]({{< relref "visidata-install" >}}) myself — sometimes setup turned into a quest of its own.
2. **Speed was capped by the architecture.** I wanted a columnar engine under the hood, not add-ons layered on top of an existing app.

So I built tuitab in Rust on top of [Polars](https://www.pola.rs) — which gave me both speed and a single self-contained binary with zero external dependencies. What used to be hacks in the fork became part of the core here.

---

### What tuitab Can Do

1. **Six formats out of the box**  
   CSV/TSV (delimiter auto-detected), JSON, Parquet, Excel, SQLite, and DuckDB. Open a single file, browse a whole folder, or pipe data in — without a single extra package.

   ```bash
   # Open a file
   tuitab data.csv

   # Browse several files as a list
   tuitab orders.csv customers.csv products.parquet

   # Read data from a pipe (format set with -t)
   psql -c "SELECT * FROM orders" --csv | tuitab -t csv
   ```

2. **A self-contained binary — no Python**  
   No virtual environments and no version conflicts: it installs with one command, and DuckDB and SQLite are compiled right into the binary. Alongside `tuitab` you also get the shorter aliases `ttab` and `ttb` — to type less.

3. **Vim-style navigation and pinned columns**  
   Move with `h` `j` `k` `l`, jump to the start/end with `gg` / `G`, page through with `Ctrl+B` / `Ctrl+F`. Sort the current column: `[` ascending, `]` descending, `r` to reset. You can pin a column to the left with `!` so it doesn’t scroll away horizontally.

4. **Instant analysis: statistics, frequencies, charts**  
   Press `I` for a per-column summary (count, nulls, unique, min/max, mean, median, mode, standard deviation, and quantiles). `F` builds a frequency table, and `V` draws a chart right in the terminal: numeric columns get a smart histogram (Freedman–Diaconis rule), categorical ones get a frequency bar chart. Pin a date or category column first, and you get line and grouped charts.

   ![Drawing charts in tuitab: histogram and bar chart rendered in the terminal](/tech/tuitab/charts.gif)

5. **Pivot tables, JOINs, and computed columns**  
   This is the stuff you’d usually open Python for. In tuitab a pivot works like this: pin the columns to group by, put the cursor on the column you want to spread across, press `W`, and type an aggregation formula such as `sum(revenue)`.

   ![Building a pivot table in tuitab with a sum(revenue) aggregation](/tech/tuitab/pivot.gif)

   `J` launches a step-by-step JOIN wizard: pick the second file (or an already-open sheet), the type `INNER` / `LEFT` / `RIGHT` / `OUTER`, and the key columns.

   ![JOIN wizard in tuitab: picking the file, join type, and key columns](/tech/tuitab/join.gif)

   And with `=` you can add a computed column from an expression — arithmetic, string and date functions, and conditionals are all supported. The new column appears right next to the cursor.

   ![Adding a computed column from an expression in tuitab](/tech/tuitab/compute.gif)

6. **Column types: currency, percentages, dates**  
   With `t` you can set a column’s type: string, integer, float, date, datetime, boolean, percentage, currency, or file size. For currency it immediately offers `USD · EUR · GBP · JPY` and formats the values with the right symbol. Interestingly, these are the exact things I once added to the visidata fork by hand — here I made them built-in, along with a quick “percent of total” via `zf`.

7. **Export anywhere, and the clipboard**  
   Save with `Ctrl+S` — the format is picked from the extension (`.csv`, `.tsv`, `.parquet`, `.json`, `.xlsx`, `.sqlite`). And you can copy selected rows to the clipboard with `yr` straight into TSV, CSV, JSON, or Markdown — handy when you need to drop a neat table into a document or a message.

8. **Non-QWERTY layouts work too**  
   Layouts other than QWERTY (ЙЦУКЕН, QWERTZ, AZERTY) are transparently remapped — the hotkeys fire no matter which language you’re currently typing in. A small thing for most, but endlessly convenient for those who work with both Latin and Cyrillic scripts and constantly switch between them.

---

### How to Install

The simplest way is via Cargo:

```bash
# Builds and installs tuitab (plus ttab and ttb)
cargo install tuitab
```

Since DuckDB is compiled from source, the first build takes a few minutes — that’s normal. If you’d rather not wait, there are pre-built options.

```bash
# macOS / Linux via Homebrew
brew tap denisotree/tuitab
brew install tuitab
```

```bash
# Arch Linux — pre-built binary from the AUR
yay -S tuitab
```

```bash
# Debian / Ubuntu — download the .deb from the releases page and install it
sudo dpkg -i tuitab_*_amd64.deb
```

Pre-built binaries for Linux and macOS (x86_64 / aarch64) are on the [releases page](https://github.com/denisotree/tuitab/releases).

---

### Quick Start

- Open a file: `tuitab data.csv`, move with `h` `j` `k` `l`, jump with `gg` / `G`.
- Sort a column: `[` ascending, `]` descending, `r` to reset.
- Chart it — `V`, per-column stats — `I`, frequencies — `F`.
- Select rows by an expression: `|!=amount > 1000`.
- Add a column: `=`, then e.g. `revenue / units`.
- Save the result — `Ctrl+S`, quit — `q`, help at any time — `?`.

---

### Where tuitab Isn’t the Right Fit

I’ll be honest: this isn’t a “VisiData killer.” VisiData is older, more powerful, and has grown a huge plugin ecosystem — if you need a rare format or a tricky integration, chances are it’s already there. I built tuitab for something else: speed and zero install hassle.

One more thing — like VisiData, tuitab keeps data in memory. On a modern machine that’s comfortable for files of a few gigabytes, but if you’re dealing with tens of gigabytes, that’s a job for [Polars](https://www.pola.rs) or [DuckDB](https://duckdb.org) scripts directly. And the project is still young (version `0.4.x` as of this article), so something might not behave perfectly here and there.

---

### Plans and Wrap-up

tuitab has already become my main tool for quick ad-hoc exploration — the same thing I used to do in VisiData, only faster and without the dance around environments.

The next big thing I want to build is an [MCP](https://modelcontextprotocol.io) server to connect it to an LLM. The idea is to use tuitab as a powerful data-analysis engine driven by the model: the LLM opens the file, builds pivots, computes columns, filters, and reads the result, while you simply describe the task in words. In effect, it turns fast manual exploration in the terminal into a conversation with your data in plain language — and it all runs on a fast local engine, without shipping your data off to the cloud.

The project is open source (Apache-2.0), so any ideas and bug reports are welcome. In future articles I’ll dig into specific use cases in more detail: pivot tables, JOINs, and the expression language on real data.

### Feedback

If you have ideas for improvements or you found a bug — open an [issue on GitHub](https://github.com/denisotree/tuitab/issues) or message me (LinkedIn / Instagram / Telegram).
