# AGENTS.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

แนะนำอาหารไทย (Thai Food Recommendation) — a static, no-backend, multi-page site recommending Thai dishes, with a random-pick feature and localStorage-backed favorites. Pure HTML/CSS/vanilla JS, no framework, no build step, no package.json.

## Commands

There is no build/lint/test tooling in this repo. To view the site locally, serve the directory statically, e.g.:

```
python3 -m http.server
```

then open `index.html` / `detail.html` in a browser.

To sanity-check JS syntax (no test runner exists):

```
node --check menu-data.js
```

(inline `<script>` blocks in `index.html`/`detail.html` can be checked by extracting them, but there is no automated harness for this — see task.md's "Verify" sections, which note that in-browser testing has not been done because the Claude in Chrome extension was not connected.)

## Architecture

- `index.html` — main listing page. Menu cards are hardcoded `<div class="card" data-dish="..." data-category="...">` elements. All page behavior (random pick, favorites toggle, search/category filter, wiring detail links) lives in a single inline `<script>` IIFE at the bottom of the file — there is no separate JS file for this page.
- `detail.html` — ingredient detail page. Reads the `dish` query param, looks it up in `window.menuData` (from `menu-data.js`), and renders it. Shows a "ไม่พบเมนูนี้" not-found state if the dish isn't found.
- `menu-data.js` — the single source of truth for full per-dish data (name, emoji/image, category, tag, description, ingredients) used only by `detail.html`.
- `styles.css` — shared stylesheet for both pages.

### Important: two partially-duplicated data sources

Each of the 9 dishes exists in **two places** that must stay in sync by dish `name`:
1. The card markup in `index.html` (`data-dish`, emoji/image, category, tag, description) — used for the listing, search/filter, and favorites.
2. The corresponding entry in `menu-data.js` — used for the detail page, and includes the extra `ingredients` array.

When adding, renaming, or removing a dish, update both files, and keep the `name` string byte-for-byte identical (detail page lookup is an exact string match against the decoded `dish` query param). `index.html`'s inline script sets each card's `.detail-link` href to `detail.html?dish=<encoded data-dish>` automatically from `data-dish`, so the dish name only needs to be typed once per file.

### Client-side state

- Favorites are stored in `localStorage` under the key `thaiFoodFavorites` as a JSON array of dish names, read/written entirely from `index.html`'s inline script.
- The random-pick "last shown dish" is kept in an in-memory JS variable only (intentionally not persisted), so it resets on page reload but avoids repeating the same dish twice in a row within a session.

## Terminology (from CONTEXT.md)

Follow these Thai terms exactly when working on related features/copy — the project has explicit synonyms to avoid:

- **รายการโปรด (Favorites)** — the set of dishes a visitor has liked via the heart icon on each card; multi-select, persisted in `localStorage`. Avoid: รายการที่เลือก, สิ่งที่ชอบ, wishlist.
- **เมนูสุ่มวันนี้ (Random Pick)** — the single dish randomly chosen by the "สุ่มเมนูอาหารไทยวันนี้" button. Despite the name including "today", it is **not** locked to the calendar date — each click can produce a new result, and only immediate repeats are avoided. Avoid: เมนูประจำวัน, เมนูของวันนี้ (these imply a date-locked pick, which is incorrect).
