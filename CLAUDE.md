# CLAUDE.md

Guidance for working in this repository.

## Start here (the rules that matter most)

1. **Work on `dev`, not `main`.** Develop and commit on `dev`; push to
   `origin/dev` freely to save work.
2. **Never push to `main` without explicit approval** — that deploys the
   live site. Only when the owner says "push."
3. **New posts default to `published: false`** (draft). Never publish
   (`published: true`) unless explicitly told.
4. **Sync first, rebase always.** Before starting work, make sure `dev`
   isn't behind `main`; before any push, `git pull --rebase`.
5. **Build before you trust it.** Ephemeral sandbox with a UTF-8 build
   quirk — see Local build. Always build (and `grep` `_site/`) to
   confirm a change before committing.

Detail for each of these is in the sections below.

## What this is

Personal website + blog for Ryan Abrams.

- **Generator:** Jekyll (v3.10, via the `github-pages` gem)
- **Hosting:** GitHub Pages, built automatically on push
- **Domain:** `ryanabrams.com` (apex), DNS + proxy via **Cloudflare**
- **Deploy branch:** `main` — pushing to `main` triggers the Pages build
- **Working branch:** `dev` — day-to-day changes happen here (see
  Branch workflow below)
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
non-trivial changes to confirm it compiles, then `grep` the relevant
file under `_site/` to confirm the change actually rendered. Remember
that **drafts (`published: false`) are excluded from the build**, so a
draft's absence from `_site/` is expected, not a failure.

## Caching (important)

Cloudflare sits in front of GitHub Pages and caches aggressively.

- `assets/css/main.css` is cache-busted with
  `?v={{ site.time | date: '%s' }}` in `_includes/head.html`, so CSS
  refreshes on each build.
- HTML can be served stale. **Cache purging is automated:**
  `.github/workflows/cloudflare-purge.yml` fires on a successful
  `github-pages` deployment and purges the Cloudflare cache via the API
  (needs repo secrets `CLOUDFLARE_API_TOKEN` + `CLOUDFLARE_ZONE_ID`; the
  token only needs Zone · Cache Purge). If something still looks stale,
  check that workflow's run in the Actions tab, or do a manual Cloudflare
  **Purge Everything** + hard-refresh.
- The sandbox cannot fetch the live domain (`host_not_allowed`); verify
  changes via a local build instead.

## Structure

- `_config.yml` — site config, `url`, plugins, avatar
- `_layouts/` — `default.html` (shell + all inline JS), `post.html`
- `_includes/` — `head.html`, `nav.html`, `footer.html`, `post-share.html`
- `_sass/` — partials; `_variables.scss` holds the theme tokens
- `assets/css/main.scss` — imports the partials (has Jekyll front matter)
- `assets/images/` — `avatar.jpg`, `social-card.jpg` (default OG image);
  `blog/` for post/hero images
- favicons + `site.webmanifest` live at the repo root
- `index.html` (homepage), `blog/index.html`, `projects/index.html`,
  `contact/`, `privacy-policy/`, `thanks/`, `thanks-booking/`

## Pages & display logic

- **The `Projects` category is the switch.** A post whose `categories`
  contain `Projects` shows on `/projects/` and is **excluded** from the
  blog index and homepage feed. Everything else is a blog post.
- **Homepage (`index.html`):** hero (avatar + social links, each shown
  only if its matching `site.*_url` is set), then a "Latest from the
  blog" feed — non-`Projects` posts, **capped at 3**, followed by a
  permanent "More posts →" button to `/blog/`.
- **Blog index (`/blog/`):** all non-`Projects` posts, with category
  **filter pills** auto-generated from the posts' categories (client-side
  JS filtering via `data-category` / `data-filter`).
- **Projects (`/projects/`):** only `Projects` posts, same card style.
- Cards use the post's `image` if set, otherwise fall back to a cycling
  CSS gradient placeholder.

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

## JavaScript & dependencies

- **All interactive features are vanilla, inline JS** in
  `_layouts/default.html` (theme toggle, hamburger, copyright year,
  copy-link, cookie consent). No build step, no framework, no
  client-side package manager — keep it that way.
- **Don't add third-party JS for features.** Share buttons use plain
  intent URLs (LinkedIn/X/mailto) and the clipboard API with an
  `execCommand` fallback — no SDKs. New features should follow suit.
- **Analytics/advertising are the deliberate exception:** Google Tag
  Manager (`gtm.id` = `GTM-TCCWGMLK`) is the only third-party tag, and it
  now holds both Google Analytics **and** an X (Twitter) advertising
  pixel. So the site is **not** "script-free" or privacy-neutral; any
  additional tags live inside GTM, not the repo.
- **GTM is consent-gated via Consent Mode v2 — it does NOT auto-load.**
  `head.html` sets `gtag('consent','default', …)` with everything
  **denied**, then only *defines* `window.loadGTM()` (guarded by
  `__gtmLoaded`); nothing invokes it until the visitor grants a
  category. The consent script at the bottom of `default.html` pushes
  `gtag('consent','update', …)` per category and calls `loadGTM()` only
  when **analytics OR advertising** is granted. There is **no** GTM
  `<noscript>` iframe (it can't honor consent). Don't reintroduce an
  auto-loading GTM snippet.
- **Two consent categories → Consent Mode signals:** Analytics maps to
  `analytics_storage`; Advertising maps to `ad_storage` +
  `ad_user_data` + `ad_personalization`. **GTM-side requirement:** each
  tag must respect these — GA4 honors `analytics_storage` natively, but
  the X/ad pixel (a custom tag) needs an *Additional consent check*
  requiring `ad_storage` in GTM, or it'll fire regardless. The repo
  sends the signals; gating each tag is configured in the GTM UI.
- **Cookie consent UI:** `_includes/cookie-consent.html` (banner +
  settings modal with Necessary / Analytics / Advertising), styled in
  `_sass/_cookie.scss`, rendered only when `site.gtm_id` is set. Buttons
  carry `data-cc` actions (`accept` / `reject` / `settings` / `save` /
  `close`); the choice is saved in `localStorage` under `cookieConsent`
  (`{"analytics": bool, "advertising": bool}`) and honored on return
  visits. An element with `id="open-cookie-settings"` (anywhere — e.g.
  the privacy page) reopens the modal; the handler no-ops if absent.
  Note: revoking *after* granting takes effect on the next page load (a
  loaded GTM can't be unloaded mid-page).

## Content conventions

- **Posts:** `_posts/YYYY-MM-DD-slug.md`, `layout: post`. Front matter:
  `title`, `date`, `categories`, `excerpt`, `read_time`, optional `image`.
- **Permalink** is `/blog/:title/` — the date affects ordering only, not
  the URL (the URL comes from the filename slug).
- **Project posts:** include `Projects` in `categories` (see Pages &
  display logic for how that routes the post).
- **Drafts & publishing:** `published: false` keeps a post/page out of
  the build (not live, not in `_site/`); `published: true` (or omitting
  the line) publishes it. The owner frequently keeps **drafts** in the
  repo that aren't ready yet, so treat unfinished posts as drafts by
  default.
  - **When asked to create a new post, always set `published: false`.**
  - **Never set `published: true` unless the owner explicitly says to
    publish.** Adding finished content to a post is not the same as
    publishing it — leave the draft flag alone until told otherwise.
- **Search visibility:** site is indexable by default. Use
  `noindex: true` to keep a page out of search and `sitemap: false` to
  keep it out of `sitemap.xml`. Both are set on `privacy-policy`,
  `thanks`, and `thanks-booking`.
- **Sitemap:** a **custom `sitemap.xml`** at the repo root overrides
  `jekyll-sitemap` (the plugin defers when the file exists). It adds a
  `<lastmod>` to every URL: posts use `last_modified_at` or their `date`;
  pages fall back to **`site.time`** (the build timestamp), so page
  `lastmod` reflects the last *build*, not the last content change. Set
  `last_modified_at:` in a page's front matter to give it an accurate
  date. (GitHub Pages can't run `jekyll-last-modified-at`, so git-based
  per-page dates aren't available without moving to a custom Actions
  build.)
- **`llms.txt`:** an LLM-readable index of the site (llmstxt.org
  convention) at the repo root, **generated by Liquid at build time** —
  don't hand-edit the output. It lists blog posts, project posts, pages,
  and the RSS feed, and **honors the same visibility policy as search**:
  pages with `noindex: true` are filtered out, and drafts never reach
  the build. New posts/pages appear automatically. It carries
  `sitemap: false` (and, being a `.txt`, isn't in `site.html_pages`
  anyway, so the custom sitemap skips it). Note: this is *not* the same
  as Cloudflare's **Markdown for Agents** (`Accept: text/markdown`
  content negotiation) — that's an edge feature requiring a Cloudflare
  **paid plan** and can't be done from a static GitHub Pages origin.
- **Social meta (Open Graph / Twitter):** handled by `jekyll-seo-tag`
  (`{% seo title=false %}` in `head.html`). Its **`og:image` comes from
  `page.image` only** — *not* `site.image` or `logo` (`logo` feeds only
  the JSON-LD structured data). The site-wide default card lives at
  `assets/images/social-card.jpg` (1200×630 JPEG — JPEG, not WebP, for
  scraper compatibility) and is applied via a `_config.yml` `defaults`
  block **scoped to `type: pages`**. That scope is deliberate: posts must
  keep their own `image:` (or none), because the post cards key off
  `post.image` — broadening the default to posts would make every card
  show the social card instead of its gradient placeholder. A post with
  its own `image:` overrides the default for its own `og:image`.
- **Featured images:** 16:9 (e.g. 1600×900). Compress to **WebP**
  (~60 KB) and store in `assets/images/blog/`. Cards/hero use
  `object-fit: cover`, so keep the subject centered.
- **Inline post images** go in `assets/images/blog/` too; `.post-body
  img` makes them responsive. Convert/compress these to WebP as well.
- **Image tooling is available.** Python **Pillow** handles
  conversion/compression (resize, JPEG/PNG → WebP). It's installable in
  the sandbox via `pip install Pillow` (PyPI is reachable; arbitrary
  hosts are blocked). Don't assume image work is impossible — it's how
  the hero and inline images here were compressed.
- **Image-sizing gotcha (hit twice):** any raw `<img>` needs an explicit
  CSS rule (`max-width: 100%; height: auto`) or it overflows and breaks
  layout on mobile. The covered cases are `.post-body img` (inline) and
  `.feat-card .img img` (homepage cards). Before adding an image in a new
  context, confirm a sizing rule exists for it — don't assume.

### Creating a new post

1. Create `_posts/YYYY-MM-DD-slug.md`. The **date** sets ordering; the
   **slug** sets the URL (`/blog/slug/`).
2. Start from this front matter — note `published: false` by default:

   ```yaml
   ---
   layout: post
   title: "Post Title"
   date: 2026-06-09
   categories: [Marketing Ops]   # add "Projects" to route it to /projects/
   excerpt: "One sentence for cards and social/search previews."
   read_time: 5
   # image: /assets/images/blog/slug.webp   # optional 16:9 WebP hero
   published: false
   ---
   ```

3. Write the body in Markdown; use `##` / `###` for section headings.
   Inline images: optimize to WebP into `assets/images/blog/` first.
4. Build locally to confirm. Leave `published: false` until the owner
   explicitly asks to publish.

## Branch workflow

- **Before doing any work on `dev`, check it against `main`.** If `dev`
  is behind `main`, sync it first (bring `main` into `dev`) before
  making changes — start every task from a `dev` that matches `main`.
- **Develop on `dev`.** Make and commit all changes on `dev`.
- **`main` is the deploy branch.** Work on `dev` is NOT live until it
  reaches `main` — GitHub Pages only builds `main`.
- **Never push `dev` → `main` without explicit approval.** Pushing to
  `main` deploys to the live site, so it only happens when the owner
  explicitly says to.
- **When the owner says "push," push to `main`.** That means: merge
  `dev` → `main` and push `main` (this triggers the deploy). Then keep
  `dev` and `main` in sync so they don't drift.
- `dev` and `main` should hold identical content; only diverge while a
  set of changes is in progress on `dev`.
- **Rebase before pushing.** The remote is often ahead — the owner
  pushes commits directly. Run `git pull --rebase origin <branch>`
  before any push to avoid rejected pushes.
- **The sandbox is ephemeral.** The container is cloned fresh each
  session and reclaimed afterward; uncommitted/unpushed work is lost.
  Commit and push to `origin/dev` to persist work across sessions —
  that's just saving, not a deploy, and needs no approval.

## Conventions for changes

- Keep edits consistent with the existing style; reuse the theme tokens.
- Don't commit `_site/`, caches, `Gemfile.lock`, or
  `.claude/settings.local.json` (see `.gitignore`).
- **Keep this file current.** If anything surfaces during work that
  belongs here — a new convention, a recurring gotcha, a workflow rule,
  a non-obvious build/deploy detail — point it out and ask whether to
  document it in CLAUDE.md. Don't silently update this file; flag it and
  let the owner decide.
