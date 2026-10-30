---
name: generate-post-posters
description: Generate a 4-slide Instagram poster carousel (cover, concept, flow, recap) for a blog post as local PNG files. Use this right after a new post is added/published under src/content/posts/, or whenever asked to create, regenerate, or check for missing Instagram posts/posters for an article.
---

# Generate Instagram posters for a post

Every published post gets a 4-image Instagram carousel, rendered locally as
PNGs — never via a published Claude Artifact / claude.ai (the user has
explicitly said these must stay local, not uploaded to Claude's cloud).

## When to run this

- Immediately after a new post is merged/added to `src/content/posts/`, as
  the natural next step of publishing it — don't wait to be asked.
- When the user asks to check whether posters exist for an article, or to
  generate/regenerate them.

## 0. Check what already exists

List `social-media/instagram/` (gitignored — it may not show up in a fresh
checkout, that's expected). Each post that has posters gets its own
subfolder named after the post's slug, containing exactly 4 files:
`poster1-cover.png`, `poster2-<topic-word>.png`, `poster3-flow.png`,
`poster4-recap.png`. Only generate what's missing unless told to
regenerate everything. Never touch another post's existing poster set
without being asked — if its content or theme looks stale/outdated, flag
it and ask before changing it.

## 1. Pull CURRENT brand tokens — never hardcode or copy from old posters

Read these two files fresh every time, since they can change between
posts (they have before — the site was rebranded and re-themed after
some posters already existed, and those posters are now stale on
purpose, left alone):

- `src/config/site.js` → `SITE_NAME` is the wordmark text for the badge/
  footer/CTA. There is no custom domain (no `public/CNAME`), so don't
  invent one — just use the plain name, no fake TLD.
- `src/styles/global.css` `:root` block → `--bg`, `--surface`,
  `--surface-badge`, `--border`, `--border-strong`, `--text`,
  `--text-dim`, `--text-faint`, `--accent`, `--font-sans` (Inter),
  `--font-mono` (JetBrains Mono). Use these values directly as the
  poster palette — do not invent a separate "Instagram theme" darker or
  lighter than the site.
- The logo badge SVG is `src/components/LogoMark.jsx` — copy its exact
  `<rect>`/`<path>` markup (squircle + chevron + cursor block), fill
  with `--accent`.

## 2. The 4-slide pattern (1080×1080px each)

1. **Cover** — logo badge + wordmark top-left; "NEW ARTICLE" eyebrow;
   post title (2 lines); excerpt from frontmatter; tag pill (from
   `src/lib/tags.js` vocabulary) + "N min read" pill; "Swipe →" bottom
   right. Estimate reading time from word count at ~200 wpm.
2. **Concept** — eyebrow + 2-line headline naming the post's core idea;
   5 numbered rows (term + short description) in bordered cards, pulled
   from the post's actual section headers/definitions — don't invent
   terms the post doesn't cover, and don't drop a term the post spends a
   whole section on.
3. **Flow** — eyebrow + headline; a small vertical diagram (boxes +
   arrows with short labels + one dashed grouping + a final pill)
   illustrating one concrete mechanism from the post. Keep it to 2-3
   boxes/groups so it fits comfortably in the 936×936 usable area
   (1080 minus 72px padding each side).
4. **Recap** — "RECAP" eyebrow + headline; the same 5 terms as the
   Concept slide (same terms, can reword the descriptions) so the two
   slides don't drift apart; CTA bar: "Read the full article" +
   `SITE_NAME`.

Keep terminology and the tag/read-time labels consistent across all 4
slides in a set.

## 3. Build each slide as standalone HTML, then render locally

Don't use the `design` skill / Claude Artifacts for the final output —
that publishes to claude.ai, which this project doesn't want. Instead:

1. Write each slide as a self-contained `.html` file: `<!doctype html>`,
   a Google Fonts `@import` for Inter + JetBrains Mono, a single root
   `<div>` sized exactly `width:1080px;height:1080px;overflow:hidden`
   with `padding:72px`, everything else as inline styles. No external
   assets besides the Google Fonts request.
2. Render to PNG with a locally installed browser in headless mode —
   check for Chrome/Edge first, e.g.:
   `"/c/Program Files/Google/Chrome/Application/chrome.exe"`. Gotchas
   learned the hard way (Windows + Git Bash):
   - Pass `--user-data-dir=<some writable temp folder>` — without it
     Chrome fails to write the screenshot at all (`Access is denied`).
   - Build the `file://` URL from a Windows-style path
     (`cygpath -m "$dir"`, giving `C:/Users/...`), not a POSIX path
     (`/c/Users/...`) — a POSIX-style `file://` URL silently loads a
     Chrome "file not found" error page instead of your HTML, and the
     screenshot "succeeds" with a small, wrong file. Always sanity-check
     output file size against a known-good render before trusting a
     batch.
   - Full command shape:
     ```
     chrome.exe --headless=new --disable-gpu --hide-scrollbars \
       --force-device-scale-factor=1 --window-size=1080,1080 \
       --user-data-dir=<writable temp dir> \
       --screenshot=<absolute-dest-path.png> \
       "file:///<Windows-style-path>/<slide>.html"
     ```
3. Save PNGs directly to
   `social-media/instagram/<post-slug>/poster{1-4}-<name>.png`
   (create the folder if it doesn't exist). No git commit needed —
   `social-media/` is gitignored (see `.gitignore`).
4. Read back at least the cover and one diagram-heavy slide with the
   Read tool to visually confirm nothing clipped and the theme/branding
   is correct before calling it done.

## 4. Don't forget

- No PR, no commit — this output is intentionally untracked.
- Don't publish a Claude Artifact for the final deliverable.
- If brand tokens (name, colors, fonts) look different from what a
  previous poster set used, trust the current source files, not the
  older images — call out the drift rather than silently matching
  stale posters.
