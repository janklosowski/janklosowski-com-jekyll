# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

The personal website and blog for Jan Klosowski at **janklosowski.com** (custom domain via `CNAME`). It's a Jekyll 4.1 static site — no app server, no build pipeline beyond Jekyll itself. There is no CI workflow in the repo; `_site/` is gitignored and not committed.

## Commands

```bash
bundle install                  # install gems (first time / after Gemfile change)
bundle exec jekyll serve        # dev server with live reload at http://localhost:4000
bundle exec jekyll build        # build static site into _site/
```

There are no tests, linters, or formatters. Verify changes by running `jekyll serve` and viewing the page, or inspecting the built file under `_site/`.

## Architecture

**Two parallel layout trees.** The homepage and the rest of the site do NOT share a base layout — this is the single most important thing to know:

- **Homepage** — `index.html` → `base-homepage` → `head-homepage.html` + `foot-homepage.html`. Forces `light-mode` and has **no dark-mode toggle**.
- **Everything else** — `post`/`page`/`blog` content → `base` → `head.html` + `foot.html`. `foot.html` contains the dark-mode toggle logic (localStorage key `theme`, falls back to `prefers-color-scheme`).

`head.html` and `head-homepage.html` are **near-identical copies** of the SEO block (Open Graph, Twitter Card, JSON-LD, canonical). When editing meta tags, fonts, or analytics, **change both** or they drift.

**Layout chain quirk:** `_layouts/page.html` and `_layouts/blog.html` declare `layout: default`, but **there is no `_layouts/default.html`**. This is intentional and harmless — those layouts already pull in `head.html`/`foot.html` via `{% include %}`, so the missing parent layout is a no-op. Do not add a `default.html` expecting it to wrap pages; it won't.

**Layouts:** `base`/`base-homepage` (head+content+foot), `post` (article + date + "Comment on 𝕏" button using `page.xlink`), `page` (generic wrapper, used by `ux.html`), `blog` (post-list loop — currently not referenced by any page), `redirect` (meta-refresh to `page.redirect_url`).

## Posts and content

Posts live in `_posts/YYYY-MM-DD-slug.markdown` (kramdown). Front matter fields in use:

- `layout: post`, `title`, optional `subtitle`, `date`, `category`
- `permalink` — **custom per post** (e.g. `/ai-kills-open-internet/`), not date-derived
- `excerpt` — drives the meta description AND the blog-list summary
- `xlink` — X/Twitter post URL; renders the "Comment on 𝕏" button in `post.html`
- `thumblink` — optional OG/Twitter image path; defaults to `/img/thumbnail.png` if absent

**Two independent ways a post is kept out of the live build** (both are used in this repo):
1. **Underscore-prefix the filename** (`_posts/_YYYY-MM-DD-slug.markdown`) — Jekyll ignores it. This repo uses this instead of a `_drafts/` folder. All the `redirect`-layout Facebook-redirect posts are parked this way.
2. **Future-date the post** (e.g. `date: 3000-...`). `_config.yml` does not set `future: true`, so future-dated posts are excluded. Two old posts (`teaching-ux`, `go-home-ico`) are hidden this way.

Net result: only the three real-dated, non-prefixed posts currently build (`dca-fire-pivot`, `email-on-your-domain`, `ai-kills-open-internet`). To publish a parked post, rename off the underscore and/or set a real past date.

`ux.html` is the standalone portfolio page (`/ux`, `noindex: true`, `sitemap: false`) with a JS-obfuscated contact email.

## Styling

Sass under `_sass/`: `_base.sass` (Sass **indented** syntax, not SCSS) and `_syntax.scss`. Both are imported by `css/main.scss`, which is the only file with front matter (so Jekyll compiles it to `/css/main.css`). Fonts: Inria Sans/Serif via Google Fonts + local Roboto Mono in `/fonts`.

## Other

- Plugins: `jekyll-feed` (`/feed.xml`) and `jekyll-sitemap` (`/sitemap.xml`).
- Analytics: Ahrefs script in the `<head>` includes.
- `welcomments` (self-hosted comments) — `_includes/welcomments/` + `_data/welcomments/`. Currently **commented out** in `post.html`; not active.
