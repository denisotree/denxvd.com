# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Personal site / tech blog for [denxvd.com](https://denxvd.com) (Denis Vdovin — data engineer). Built with the [Hugo](https://gohugo.io) static site generator and the [M10C theme](https://github.com/vaga/hugo-theme-m10c). Bilingual: English + Russian.

## Commands

```bash
hugo server -D            # local dev server with drafts (draft: true) rendered
hugo server               # local dev server, published content only
hugo --minify             # production build into public/ (git-ignored)
hugo new content/en/tech/<slug>.md   # scaffold a new post from archetypes/default.md
```

The theme is a **git submodule**. After a fresh clone: `git submodule update --init --recursive`. Local Hugo is `extended` (required — the build compiles SCSS).

## Deployment

Push to `master` → GitHub Actions (`.github/workflows/hugo.yml`) builds with **Hugo 0.128.0 extended** and deploys to GitHub Pages. There is no manual deploy step and no preview environment; merging to `master` publishes. The CI Hugo version is pinned independently of your local version — keep features compatible with 0.128.0.

## Writing voice

When writing or translating blog posts, use the project skill **`denxvd-writing-style`** (`.claude/skills/denxvd-writing-style/`) — the author's voice "mask" (frontmatter invariants, article skeleton, tone rules, RU↔EN conventions). It auto-triggers on blog-writing requests; invoke it explicitly if it doesn't. Reproduce the voice, not the typos present in older published posts.

## Content architecture

- Bilingual via Hugo's multilingual mode (`hugo.toml` `[languages]`). Content lives in **`content/en/`** and **`content/ru/`**, which mirror each other. Each language has its own menu definitions in `hugo.toml`.
- A post and its translation must share the **same file path/slug** across `content/en/` and `content/ru/` — that path match is what links them for the language switcher in the header. Adding a post in one language without the matching translation leaves the switcher with nothing to point to.
- Posts live under `content/<lang>/tech/`. `_index.md` files define section/list pages. The home page is `content/_index.md`.
- Frontmatter fields that matter (see `archetypes/default.md`): `draft` (set `false` to publish), `featured_image` (absolute path like `/tech/<slug>/<img>.jpg`, used as the list thumbnail by `layouts/_default/list.html`), `summary` (shown in the post list), `date`, `tags`.

## Images

Static assets are served from `static/` at the site root. Per-post images go in **`static/tech/<slug>/`** and are referenced by absolute path (`/tech/<slug>/...`). The `featured_image` frontmatter value and in-body `![](...)` embeds both use this root-absolute form.

## Authoring in Obsidian

This repo doubles as an Obsidian vault (`.obsidian/` is git-ignored). `archetypes/obsidian_template.md` is the Obsidian-side new-note template mirroring the Hugo archetype. Be aware: Obsidian's default attachment naming produces random filenames (e.g. `NaRzhgOTl6H2peqBxHcvQRu769.png`) that may land in `static/tech/` — rename these to meaningful names and move them into the post's `static/tech/<slug>/` folder.

## Theme customization

The theme is consumed read-only as a submodule. Customizations are layered on top from the project root (Hugo resolves project files before theme files):

- **`layouts/`** — overrides of theme templates. `_default/baseof.html` (full page shell + language switcher + analytics include), `_default/list.html` (post-list rendering), `_default/_markup/render-link.html` (opens external `http(s)` links in a new tab), `partials/analytics.html` (Yandex.Metrika).
- **`assets/css/_extra.scss`** — the theme's `main.scss` ends with `@import 'extra'`, and Hugo's asset pipeline picks up this project file to override it. This is where Fira Code, the color palette overrides, table styling, and the responsive sidebar layout live. Theme colors are also driven by `[params.style]` in `hugo.toml`, which feed SCSS variables in the theme's `main.scss`.

Edit these project-root files rather than the submodule. Changes inside `themes/hugo-theme-m10c/` would be lost on a submodule update and are not tracked by this repo.
