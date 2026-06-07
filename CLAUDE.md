# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

Vanilla single-file landing page for a Kyiv landscaping company. No build step, no framework, no package manager. The entire site lives in `index.html` (~2700 lines of inline HTML + CSS + JS).

Deploy: GitHub Pages from `main` → https://yaladnich.github.io/geometriya-sadu/

## Development

Serve locally with Python (no install needed):
```bash
python3 -m http.server 8900 --directory /home/user/geometriya-sadu
# open http://localhost:8900
```

Run automated checks with Playwright (headless Chromium):
```bash
node /tmp/your-check.js
# node is at /opt/node22/bin/node
# playwright at /opt/node22/lib/node_modules/playwright
```

## Architecture of index.html

The file is divided into four logical blocks:

1. **`<head>`** (lines 1–50) — meta, preload hints, font preloads. Hero image preload is a synchronous inline `<script>` using `document.write` — this is intentional and must not get `defer` or be moved.

2. **`<style>`** (lines 51–900) — all CSS inline. BEM-like class prefix `gs-`. Custom properties defined on `:root`. Mobile breakpoint at `768px`.

3. **`<body>` / HTML** (lines 900–1445) — sections in order: Hero → Services (`#services`) → Why (`#why`) → Portfolio (`#portfolio`) → Pricing (`#pricing`) → FAQ (`#faq`) → CTA → Footer → Modal overlay.

4. **`<script>`** (lines 1445–2729) — all JS inline, no modules. Execution order matters:
   - `vendor/gsap.min.js` and `vendor/ScrollTrigger.min.js` loaded **without `defer`** (defer breaks hero animations because inline scripts run before deferred ones resolve)
   - `vendor/lenis.min.js` and `vendor/email.min.js` are lazy-loaded via `loadScriptOnce()` on demand
   - `DOMContentLoaded` initialises: portfolio, custom selects, phone masks, counters, canvas dots (Why section), GSAP scroll animations, Lenis smooth scroll

## Key JS globals and functions

| Symbol | Purpose |
|---|---|
| `portfolioProjects[]` | Array of project objects with `images[]` and optional `imagesMobile[]` |
| `WEB3FORMS_KEY` | Public API key for Web3Forms form submission |
| `submitForm(event, prefix)` | Handles both forms — `prefix` is `'cta'` or `'modal'` |
| `openModal(service?)` | Opens contact modal, optionally pre-selects a service |
| `closeModal()` | Closes modal **and resets the form** (form.reset + clearFormErrors) |
| `initPortfolio()` | Sets up background crossfade slider with Ken Burns |
| `pfGo(direction)` | Advances portfolio image; guarded by `portfolioTransitioning` flag |
| `loadScriptOnce(src, globalName)` | Promise-based lazy script loader, deduplicates by `globalName` |
| `setText(id, value)` | Safe `textContent` setter by element id |

## Images — critical rules

**Every new image needs a mobile-compressed version.** See `BRIEF.md` for the full workflow. Short version:
- Desktop: `name.webp`, Mobile: `name-m.webp` (underscores not spaces — spaces break `srcset`)
- Compress with Pillow: max width 768px, quality 72 (58 for full-bleed dark backgrounds)
- Wire up via `<picture>` for `<img>` tags, CSS media query for backgrounds, `imagesMobile[]` array for portfolio JS

The hero background preload is conditional via inline script (line ~11). Portfolio backgrounds are set by JS only — no hardcoded `background-image` in HTML.

## Adding a portfolio project

1. Add an entry to `portfolioProjects[]` in `index.html` with `name`, `tag`, `area`, `images[]`, and optionally `imagesMobile[]`
2. Add a `<button>` in the portfolio nav HTML calling `pfSelectProject(index, this)`
3. Add compressed mobile images following the naming convention above

## Vendors (all local, no CDN)

| File | Purpose | Load |
|---|---|---|
| `vendor/gsap.min.js` | Animation engine | Synchronous, no defer |
| `vendor/ScrollTrigger.min.js` | Scroll-driven animations | Synchronous, no defer |
| `vendor/lenis.min.js` | Smooth scroll | Lazy on DOMContentLoaded |
| `vendor/email.min.js` | Legacy EmailJS (unused now) | Lazy, kept for reference |

## Git

- Branch: `claude/busy-cerf-uH3lP` → merge to `main`
- Commit author must be `Claude <noreply@anthropic.com>`
- `main` auto-deploys to GitHub Pages (~1–2 min after push)
