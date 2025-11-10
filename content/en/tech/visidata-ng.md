---
title: Visidata NG — my Visidata fork with pragmatic improvements
date: 2025-11-10T12:26:00
Description: "I forked Visidata — Visidata NG — to implement, use, and share improvements faster: exit confirmation, a quick percent-of-total column, currency formatting, flexible date formats, and more. In this post, I explain how I arrived at this, list the current changes, and show how to try them."
tags:
  - visidata
  - visidata-ng
  - bash
  - linux
  - macos
  - review
Categories: 
featured_image: "/tech/visidata-ng/visidata-ng.jpg"
featured_image_caption: ""
summary: "I forked Visidata — Visidata NG — to implement, use, and share improvements faster: exit confirmation, a quick percent-of-total column, currency formatting, flexible date formats, and more. In this post, I explain how I arrived at this, list the current changes, and show how to try them."
DisableComments: true
draft: false
---

![](/tech/visidata-ng/visidata-ng.jpg)

I’m a long-time fan of [Visidata](https://github.com/saulpw/visidata). I initially planned to contribute directly upstream, but getting changes accepted can take time, while I wanted to use them myself and share sooner. That’s why I created a fork called Visidata NG (new generation). I’m developing it independently and proposing the most promising changes upstream in parallel.

At this point, it’s not published to `pip`/`brew` or other registries — it’s for enthusiasts who want to try new features early.

### What I added

1. **Exit confirmation on the last stack level (`q`)**  
   When you’re at the last table in the stack and press `q`, Visidata NG now shows a confirmation dialog to prevent accidental exit and loss of unsaved changes. I used to lose my markup by mistake — not anymore.

2. **Quick “percent of total” via `g+f`**  
   Pressing `g+f` creates a new column based on the current one (if its type is `int` or `float`) and calculates the percent of the total. Frequency tables do show percentages by default (as a histogram), but there was no one-shot hotkey to create a dedicated column — now there is.

3. **Currency formatting on `$` (column type `currency`)**  
   On `$`, a currency picker pops up, and the column is formatted according to the selected currency standard (prefix + value + suffix). Handy when you need quick stats and a neat Markdown table.

4. **Flexible date formats on `@` (column type `date`)**  
   On `@`, you get a format picker with `datetime`, `date`, `month`, and `year`. This makes ad‑hoc exploration faster — no long expressions, just a couple of keystrokes.

### Roadmap
Next, I’m considering going deeper into data storage/processing internals: vectorized operations could significantly speed up computations. For now, I focused on the practical tweaks I miss the most day-to-day.

### How to try Visidata NG

1. Clone the repository:
   ```bash
   git clone https://github.com/denisotree/visidata-ng
   ```
2. Switch to the active development branch:
   ```bash
   cd visidata-ng
   git checkout develop-ng
   ```
3. Install in editable mode:
   ```bash
   pip install -e .
   ```
4. The executable will be installed into the `bin` directory of your current Python environment.

#### Installation caveats
- If you installed Visidata outside of `pip` (e.g., `brew`, `apt`, `pacman`, etc.), versions may conflict — you might need to remove the system-level package.
- If Visidata was installed via `pip`, my fork will replace the original since it shares the same package metadata.

### Feedback
Have ideas for improvements or found a bug? Message me (LinkedIn / Instagram / Telegram).
