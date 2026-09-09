# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

Static, self-contained HTML slide decks for a Gold Loan business presentation (FinnOne Neo). No build system, no package.json, no server — each `.html` file is opened directly in a browser. There is no test suite and no linter configured.

## Files

- `Gold_Loan_Executive_Deck.html` — main executive deck, single self-contained HTML file (~4MB). This is the canonical/tracked file. (Renamed from `Merged_Deck_Claude_Bundled.html`.)
- `index.html` — exact copy of `Gold_Loan_Executive_Deck.html`, kept in sync so GitHub Pages (which serves from the repo root) shows the deck at the site root. Update both files together when editing the main deck.
- `Merged_Deck_Claude_Bundled.html.bak`, `Merged_Deck_Claude_Bundled copy.html`, `Merged_Deck_Claude_Bundled copy 2.html` — snapshots/working copies of the main deck, still under the old filename. Check with the user which copy is the active edit target before making changes; don't assume it's the tracked one.
- `Gold loan Auditing and Vaulting Deck copy.html` — deep-dive deck for Auditing & Vaulting content, opened from the main deck via "Vaulting →" / "Auditing →" buttons.
- `Auction/auction-platform.html` — current Auction portal, linked from the main deck's Auction stage ("📊 Auction Dashboard →" button). `Auction/Auction Mockups.htm` is the prior version, kept as a backup but no longer linked.
- `Audit/Audit Mockups.html`, `vaulting/Vault Mockups.html` — standalone mockup files for audit/vaulting sections.
- `FinnOne Neo - Why Now (refined).html` — standalone slide/section, smaller file.
- `India_Gold_Loan_Market_Opportunity_Board_Deck (1).pptx`, `Gold_Loan_Workflow_and_Integration_Editable (2).pptx` — source PowerPoint decks that content/mockups get ported into the HTML decks; not consumed programmatically.
- `Before Bundled stack mockups.zip` — archived pre-bundling mockup reference, not part of the live decks.

## Architecture (per deck file)

Each deck is one HTML file with three inline blocks in `<head>`/`<body>`, no external build step:

1. **CSS** — multiple `<style>` blocks, including a Tailwind-derived `<style id="tailwind-injected">` block (utility classes, no Tailwind build tooling involved — it's pre-generated/inlined).
2. **Markup** — `<div class="deck" id="deck">` contains one `<section class="slide" data-title="...">` per slide, in document order. The first slide has `class="slide active"`. Within a slide, progressively-revealed content uses `class="fragment"` (optionally `f-up`/`f-right`/`f-scale` for animation variant).
3. **JS** (bottom, inline `<script>` tags) — hand-written navigation controller, no framework:
   - `slides`/`cur`/`TOTAL`, `goSlide(n)` / `enterSlide(n, revealAll)` — slide switching.
   - `nextStep()` / `prevStep()` — Right Arrow/Space and Left Arrow step through fragments first, then slides.
   - `onSlideEnter(n)` — dispatches per-slide setup by index (e.g. `if (n === 2) buildMarketChart();`) — when inserting/reordering slides, these hardcoded indices must be updated.
   - Chart.js (loaded from CDN `cdn.jsdelivr.net/npm/chart.js`) builds charts like `buildMarketChart()`, `buildTimeline()`, `buildRiskChart()`.
   - GSAP (referenced via `typeof gsap !== 'undefined'` guards) drives slide/fragment transition animations — code degrades gracefully if GSAP isn't loaded.
   - `Cmd/Ctrl-K` opens a command-palette-style quick switcher (`openCmdK`/`closeCmdK`).
   - `stampSVG(label)` renders the recurring circular "assay-mark" stamp graphic used across slides.
   - `fitScreen()` scales the whole `#deck` to fit the viewport against a fixed 1920×1080 design canvas.

## Cross-deck deep-linking

The main deck links out to the Auditing & Vaulting deck via `openDeepDive(hash, currentSlideId)`:
```js
function openDeepDive(hash, currentSlideId) {
  var self = 'Gold_Loan_Executive_Deck.html#' + currentSlideId;
  var target = encodeURI('Gold loan Auditing and Vaulting Deck copy.html');
  var url = target + '?return=' + encodeURIComponent(self) + '#' + hash;
  navTo(url);
}
```
**Gotcha:** `self` hardcodes the filename `Gold_Loan_Executive_Deck.html`, not the current document's actual filename. If you're editing one of the copy/snapshot files (e.g. `Merged_Deck_Claude_Bundled copy.html`), the "back" link from the deep-dive deck will return to the renamed tracked file, not the copy — keep this in mind when testing deep-dive navigation from a copy.

The Auditing & Vaulting deck reads the return link back via `URLSearchParams(location.search).get('return')`.

`navTo(url)` (defined near the top of `<head>`) wraps navigation to use the native Cross-Document View Transitions API when available (`@view-transition { navigation: auto; }`), falling back to a manual opacity fade for browsers without it.

## Working with these files

- There's no build/lint/test command — verify changes by opening the HTML file directly in a browser.
- Files are large (2-4MB); prefer targeted `grep`/`Edit` over reading the whole file into context. Slide sections are delimited by large comment banners like `<!-- ═══ SLIDE 1 — COVER ═══ -->`.
- Editing `Gold_Loan_Executive_Deck.html` for real (not a copy/snapshot)? Mirror the change into `index.html` too, or the two drift out of sync.
- When adding/removing/reordering slides, check `onSlideEnter(n)` for index-based `if (n === N)` setup calls that need updating, and check `data-title` attributes used by the slide picker.
