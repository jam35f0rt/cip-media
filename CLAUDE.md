# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A single-page marketing website for **Clip Media**, a creative studio in Port-au-Prince, Haiti
(cinematic production, branding, storytelling, marketing). The entire site is **one self-contained
file**: `index.html` (~2800 lines) with inline CSS and JavaScript. There is no build step, no
package manager, no framework, and no dependencies installed locally — everything ships as static
HTML.

## Running and previewing

There is nothing to build or compile. Serve the file over HTTP rather than opening it with `file://`,
because the projects feed uses `fetch()`:

```bash
python3 -m http.server 8000   # then open http://localhost:8000
```

There are no tests, linters, or CI configured. Validation is manual: open the page and check the
console for warnings (the Sanity fetch logs `[Clip Media] Sanity fetch error:` on failure).

## File layout inside `index.html`

The single file is organized in three blocks; know which one you're editing:

- **`<style>` (≈ lines 21–1763)** — all CSS. Design tokens live in the `:root` block at the top
  (color, type scale, spacing `--s1..--s10`, radius, elevation). Reuse these variables instead of
  hardcoding values; the visual system depends on them.
- **`<body>` (≈ 1765–2014)** — markup. Main sections are `#accueil` (hero), `#services`,
  `#realisations` (projects), `#rdv` (CTA band), `#contact` (footer). Header nav, language dropdown
  (`#lang`), and mobile menu (`#mobileMenu`) live above `<main>`.
- **`<script>` (≈ 2016–end)** — all behavior: i18n dictionary + engine, the Sanity projects feed
  (IIFE marked `SANITY — Projects feed`), and the service-modal / projects-overlay engine (IIFE
  marked `SERVICE MODAL + PROJECTS OVERLAY ENGINE`).

## Internationalization (the central convention)

The site is fully translated into **four languages: `fr` (French, default), `ht` (Haitian Creole),
`en`, `es`**. This is the most important thing to get right when changing copy.

- All user-facing text is keyed in the `I18N` object (one sub-object per language). **Every key must
  exist in all four languages** — never add a string to one language only.
- In markup, attach `data-i18n="some.key"` to an element; `setLang()` replaces its `textContent`.
  The default text written inline in the HTML is just a fallback — the real source of truth is `I18N`.
- Multi-item lists (service modal bullet points) are stored as a single string split on `||`
  (e.g. `svc1.modal.items`).
- Language choice persists in `localStorage` under the key `cip_lang`; default is French.
- When adding visible copy: add the key to all four language dicts, then reference it with
  `data-i18n` (static text) or via the `t()` / `tr()` helpers (JS-generated text).

## Projects feed (Sanity CMS)

The "Réalisations" grid is populated at runtime from **Sanity**, not from anything in this repo:

- Config is hardcoded in the projects IIFE: `PROJECT_ID = '9451pfzt'`, `DATASET = 'production'`,
  fetched from `https://<id>.apicdn.sanity.io/...` with a GROQ query over `_type == "project"`
  (fields: `title`, `category`, `description`, `tags`, `projectUrl`, and `imageUrl` from
  `thumbnail.asset->url`).
- Images are requested through the Sanity image CDN with transform params
  (`?w=700&h=525&fit=crop&auto=format`).
- The grid shows 4 cards on small screens (≤560px) and 5 otherwise; "Voir toutes nos réalisations"
  reveals the rest inline. The fetched list is cached on `window.__cipProjects`.
- The fetch has an 8s `AbortController` timeout and degrades to a localized error message
  (`proj.error`) or empty state (`proj.empty`). To change the data, edit content in Sanity, not this
  repo.

## Other behaviors to know

- **Contact / booking is WhatsApp-based**, not a form or calendar widget. `WA_NUMBER = "50937007592"`;
  `updateWA()` rewrites every `a[href*="wa.me"]` with a language-specific prefilled message (`wa.msg`).
  Changing the number means editing that one constant.
- **Service "En savoir plus" links** carry `data-service="card1..card4"` and open a modal whose
  content is pulled from `svcN.modal.title|desc|items` i18n keys.
- **Motion**: scroll reveals (`.reveal` / `.stagger`), the hero parallax, and card animations all
  check `prefers-reduced-motion` and skip animation when reduced. Preserve that guard for any new
  animated element.
- The only external runtime dependency besides Sanity is **Google Fonts** (Plus Jakarta Sans).

## Conventions

- Keep everything inline in `index.html` — do not introduce a build system, split into modules, or
  add npm dependencies unless explicitly asked.
- The JS is written in defensive, framework-free ES5/ES6 style (feature-detects `AbortController`,
  guards `typeof ... function`). Match that style; assume no transpilation.
- Default/source language is French; write fallback inline text and the first-authored copy in `fr`.
