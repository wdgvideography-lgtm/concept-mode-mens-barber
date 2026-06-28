# CLAUDE.md

Guidance for AI assistants (and humans) working in this repository.

## What this is

A **single-page static marketing website** — a design concept built by WDG for
**AMC Transport Solutions Ltd**, a UK drainage, waste-management, tanker, jetting
and bulk-haulage company. It is a self-contained brochure site: one HTML file with
all CSS and JavaScript inline, plus image/video assets and a Vercel deploy config.

> **Note on the repo name:** the repository is named `concept-mode-mens-barber`,
> but the actual content is the AMC Transport concept site. The name is a leftover
> from a reused template/slot — trust the file contents, not the repo name. Don't
> "fix" the site to be about a barber.

There is **no build system, no framework, no package manager, and no dependencies**.
Everything ships exactly as it sits in the repo.

## Repository layout

```
.
├── index.html              # The entire site — markup + inline <style> + inline <script>
├── vercel.json             # Static hosting config (clean URLs, asset cache headers)
└── assets/
    ├── placeholder.svg     # Branded navy/amber truck graphic shown when a photo is missing
    ├── photos/             # JPG photos referenced by exact filename (see photos/README.txt)
    │   └── README.txt      # Authoritative list of expected photo filenames + what each slot is
    └── video/              # Hero background clips (hero-1.mp4, hero-2.mp4) + hero-poster.jpg
```

`index.html` is ~700 lines and is the source of truth for the whole site. When asked
to change anything visual or behavioral, you are almost always editing `index.html`.

## How to run / preview

No build step. Serve the folder statically and open it:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000
```

Opening `index.html` directly via `file://` mostly works, but use a local server so
the hero `<video>` clips and relative asset paths behave like production.

## Deployment

Hosted as a static site on **Vercel** (`vercel.json`):
- `cleanUrls: true`, `trailingSlash: false`
- `assets/photos/*` served with `Cache-Control: public, max-age=86400`

There is no build command — Vercel publishes the files as-is. To deploy, push to the
branch wired to the Vercel project (or use the Vercel MCP tools if asked).

## Architecture & conventions

### One file, three layers
`index.html` contains, in order:
1. `<head>` — meta/OpenGraph tags, Google Fonts (Oswald + Inter), and a
   `LocalBusiness` JSON-LD block with the real business NAP (name/address/phone).
2. `<style>` — all CSS, driven by **CSS custom properties** declared in `:root`.
3. `<body>` — semantic `<section>`s, then a single `<script>` at the end.

### Design tokens (CSS variables in `:root`)
The palette and look are centralized. Prefer these over hard-coded colors:
- `--navy`, `--navy2..4`, `--ink` — dark backgrounds
- `--amber`, `--amber-light`, `--amber-pale` — brand accent (CTAs, highlights)
- `--blue`, `--green`, `--steel`, `--light`, `--white`, `--grey`, `--line`
- Fonts: **Oswald** for headings/`.display`, **Inter** for body.

### Page sections (each `<section>` has an `id` used by nav anchors)
`#hero` → trust strip → `#services` → stats → `#about` → values → `#fleet`
→ `#gallery` → CTA → `#contact` → footer. Nav links and footer links are
in-page smooth-scroll anchors (`href="#..."`).

### JavaScript (vanilla, no libraries — bottom of `index.html`)
- **Sticky nav:** toggles `.scrolled` on `#navbar` past 50px scroll.
- **Hero video cycling:** swaps between `hero-1.mp4` / `hero-2.mp4` on `ended`.
- **Reveal-on-scroll:** `IntersectionObserver` adds `.visible` to `.reveal` elements
  (stagger with `.reveal-d1`/`-d2`/`-d3`). Use these classes for new animated blocks.
- **Stat count-up:** elements with `data-count` (and optional `data-suffix`) animate.
- **Smooth anchor scroll** for `a[href^="#"]`.
- **`prefers-reduced-motion`** is respected — video autoplay is disabled and count-ups
  jump straight to final values. Keep new motion gated the same way.

### Images & the placeholder fallback
This is the most important convention to preserve:
- Every `<img>` points at a real path under `assets/photos/` and carries
  `onerror="this.style.display='none'"`.
- Each image sits over a `.ph` element (or has a gradient sibling). When the photo
  file is absent, the `<img>` hides itself and the branded `placeholder.svg` /
  gradient shows through — so the page looks complete with or without photos.
- **`assets/photos/README.txt` is the canonical list of expected filenames** and what
  each slot should contain. To add/swap a photo, drop a file with the exact matching
  name; no code change needed. When adding a *new* image slot, replicate the
  `<img ... onerror=...>` + `.ph` pattern so the fallback keeps working.

## Business facts embedded in the page (keep consistent everywhere)
If you change any of these, update **all** occurrences — the JSON-LD, the contact
section, the footer, and `tel:`/`mailto:` links must agree:
- Phone: `07543 564442` (`tel:07543564442`) — advertised 24/7
- Email: `enquiries.amctransport@outlook.com`
- Address: Foreclose Farm, New Road, Heage, Derby, DE56 2BA, Derbyshire, GB
- Tagline: "No job too big nor too small" · "24/7, 365 days a year"
- Footer credit: "Website concept by WDG"

## Working in this repo

- **Edit `index.html` directly.** Match the existing inline style/structure — don't
  introduce a framework, bundler, external CSS/JS files, or npm dependencies unless
  the user explicitly asks to restructure the project.
- **Reuse the CSS variables and existing utility classes** (`.btn-amber`, `.btn-ghost`,
  `.h2`, `.eyebrow`, `.lead`, `.reveal`, `.center`) instead of inventing one-offs.
- **Keep accessibility/motion behavior intact** — `alt` text on images, the reduced-motion
  guards, and semantic section structure.
- There are no tests, linters, or CI in this repo; verify changes by eye in a browser.

## Git workflow

- Active development branch for current work: **`claude/claude-md-docs-pfyjme`**.
- Develop on the designated branch, commit with clear messages, and
  `git push -u origin <branch-name>`.
- Do **not** open a pull request unless explicitly asked.
