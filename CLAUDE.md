# CLAUDE.md

Guidance for working in this repository.

## What this is

Personal website + blog for Ryan Abrams.

- **Generator:** Jekyll (v3.10, via the `github-pages` gem)
- **Hosting:** GitHub Pages, built automatically on push
- **Domain:** `ryanabrams.com` (apex), DNS + proxy via **Cloudflare**
- **Deploy branch:** `main` — pushing to `main` triggers the Pages build
- **Plugins:** `jekyll-feed`, `jekyll-sitemap`, `jekyll-seo-tag`

## Local build / preview

The build **requires a UTF-8 locale**. The SCSS comments contain `──`
box-drawing characters; with an empty/POSIX locale the old Ruby `sass`
gem reads files as US-ASCII and the build fails with
`Invalid US-ASCII character "\xE2"`. GitHub Pages uses UTF-8, so this is
only a local concern.

```bash
bundle install
LANG=C.UTF-8 LC_ALL=C.UTF-8 bundle exec jekyll build      # or: serve
```

If `bundle exec jekyll` reports "command not found", invoke the gem's
exe directly:
`LANG=C.UTF-8 bundle exec ruby "$(find / -path '*/gems/jekyll-*/exe/jekyll' | head -1)" build`

Output goes to `_site/` (git-ignored). Always build before pushing
non-trivial changes to confirm it compiles.

## Caching (important)

Cloudflare sits in front of GitHub Pages and caches aggressively.

- `assets/css/main.css` is cache-busted with `?v={{ site.time }}` in
  `_includes/head.html`, so CSS refreshes on each build.
- HTML can still be served stale. After a change that looks like it
  "didn't deploy," do a Cloudflare **Purge Everything** and hard-refresh.
- The sandbox cannot fetch the live domain (`host_not_allowed`); verify
  changes via a local build instead.

## Structure

- `_config.yml` — site config, `url`, plugins, avatar
- `_layouts/` — `default.html` (shell + all inline JS), `post.html`
- `_includes/` — `head.html`, `nav.html`, `footer.html`, `post-share.html`
- `_sass/` — partials; `_variables.scss` holds the theme tokens
- `assets/css/main.scss` — imports the partials (has Jekyll front matter)
- `assets/images/` — `avatar.jpg`; `blog/` for post images
- favicons + `site.webmanifest` live at the repo root

## Theme system (light/dark)

- CSS custom properties defined in `_sass/_variables.scss`: `:root` is
  light, `[data-theme="dark"]` overrides.
- `--invert` / `--invert-text` are for high-contrast pill buttons (dark
  pill on light mode, light pill on dark) — use these, not `--body`, for
  button backgrounds.
- Text tiers: `--body` > `--text-2` > `--text-3` > `--muted`.
- Theme is set **before paint** by an inline script in `head.html`
  (reads `localStorage.theme`, falls back to OS preference).
- The theme toggle, mobile hamburger menu, copy-link, and auto
  copyright-year scripts all live at the bottom of `_layouts/default.html`.

## Content conventions

- **Posts:** `_posts/YYYY-MM-DD-slug.md`, `layout: post`. Front matter:
  `title`, `date`, `categories`, `excerpt`, `read_time`, optional `image`.
- **Permalink** is `/blog/:title/` — the date affects ordering only, not
  the URL.
- **Project posts:** include `Projects` in `categories`. They show on
  `/projects/` and are excluded from the blog index and homepage feed.
- **Drafts:** add `published: false` (omitting the line = published).
  Works on any page or post.
- **Search visibility:** site is indexable by default. Use
  `noindex: true` to keep a page out of search and `sitemap: false` to
  keep it out of `sitemap.xml`. Both are set on `privacy-policy`,
  `thanks`, and `thanks-booking`.
- **Featured images:** 16:9 (e.g. 1600×900). Compress to **WebP**
  (~60 KB) and store in `assets/images/blog/`. Cards/hero use
  `object-fit: cover`, so keep the subject centered.

## Conventions for changes

- Develop on `main` (per the owner's instruction) and push when asked.
- Keep edits consistent with the existing style; reuse the theme tokens.
- Don't commit `_site/`, caches, `Gemfile.lock`, or
  `.claude/settings.local.json` (see `.gitignore`).
