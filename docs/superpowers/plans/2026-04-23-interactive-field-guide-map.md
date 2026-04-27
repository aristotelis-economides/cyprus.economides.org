# Interactive Field-Guide Map Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add an interactive, parchment-styled Mapbox map of Cyprus to `index.html` that renders curated recommendations from a JSON file.

**Architecture:** A self-contained `<section id="field-guide">` block in `index.html` that loads Mapbox GL JS from CDN, fetches `assets/recommendations.json`, and renders DOM-based markers with custom popups. All CSS and JS is inline in `index.html` to match the existing zero-build static-site pattern. Map style is authored in Mapbox Studio and referenced by style URL.

**Tech Stack:**
- Mapbox GL JS v3.8.0 (via Mapbox CDN)
- Static JSON data file (`assets/recommendations.json`)
- Inline HTML / CSS / vanilla JS in `index.html`
- Verification via Playwright MCP browser tools (no test framework added; the repo has none)

**Verification approach:** This repo has no test infrastructure and this feature is primarily visual. Rather than add a test framework for one feature, each task ends with an explicit in-browser verification step (the engineer opens the page locally and confirms specific behavior). A final Playwright-driven E2E check at the end confirms the interactive behavior programmatically.

---

## Prerequisites (site owner)

Before Task 4, the site owner must provide two values:

1. **Mapbox public access token** — create at <https://account.mapbox.com/access-tokens/>, restrict allowed URLs to `https://cyprus.economides.org/*` and `http://localhost:*`
2. **Mapbox Studio style URL** — create a style in <https://studio.mapbox.com/>, tune it toward the parchment/ink palette per the design spec, publish it, copy the style URL (format: `mapbox://styles/<user>/<id>`)

Throughout the plan, `<MAPBOX_TOKEN>` and `<STYLE_URL>` are the exact strings to replace with these values. They are not placeholders in the "TBD" sense — they are configuration inputs the owner must supply.

## File plan

- **Create** `assets/recommendations.json` — data file, owner-editable
- **Modify** `index.html` — add CDN links to `<head>`, new `<section id="field-guide">`, CSS in existing `<style>`, init code in existing `<script>`, TOC entries

No other files are created or modified.

---

## Task 1: Create sample recommendations.json

**Files:**
- Create: `assets/recommendations.json`

- [ ] **Step 1: Write the data file with three sample entries**

Write `assets/recommendations.json`:

```json
{
  "recommendations": [
    {
      "name": "Blue Lagoon",
      "coords": [32.3217, 35.0739],
      "maps_url": "https://www.google.com/maps?q=35.0739,32.3217",
      "image": "assets/blue-lagoon-top-view-optimised.webp",
      "note": "Worth the boat trip from Latchi or the 4x4 dirt track through Akamas. Go early morning before the day-boats arrive."
    },
    {
      "name": "Khirokitia",
      "coords": [33.3417, 34.7986],
      "maps_url": "https://www.google.com/maps?q=34.7986,33.3417",
      "image": "assets/khirokitia.webp",
      "note": "A Neolithic settlement from around 7,000 BC. The reconstructed dwellings let you walk inside and see how people actually lived nine millennia ago."
    },
    {
      "name": "Tombs of the Kings",
      "coords": [32.4058, 34.7766],
      "maps_url": "https://www.google.com/maps?q=34.7766,32.4058",
      "note": "An underground necropolis carved into the rock. You can climb down into burial chambers designed like houses, with columns and frescoed interiors."
    }
  ]
}
```

- [ ] **Step 2: Verify it parses as valid JSON**

Run: `python3 -c "import json; json.load(open('assets/recommendations.json'))" && echo OK`
Expected: `OK`

- [ ] **Step 3: Commit**

```bash
git add assets/recommendations.json
git commit -m "add recommendations.json with three sample entries"
```

---

## Task 2: Add Mapbox CDN and empty field-guide section

**Files:**
- Modify: `index.html` (add to `<head>` near existing `<link>` tags; add new `<section>` after `<section id="food">`)

- [ ] **Step 1: Add Mapbox GL JS CSS and JS to `<head>`**

Find the existing Google Fonts `<link rel="stylesheet">` in `<head>` (around line 13) and add these two lines directly after it:

```html
<link href="https://api.mapbox.com/mapbox-gl-js/v3.8.0/mapbox-gl.css" rel="stylesheet">
<script src="https://api.mapbox.com/mapbox-gl-js/v3.8.0/mapbox-gl.js"></script>
```

- [ ] **Step 2: Add `<section id="field-guide">` after the food section**

Find the closing `</section>` of `<section id="food">` (around line 643) and insert this immediately after it, before `</article>`:

```html
            <!-- Section 11: Field guide -->
            <section id="field-guide">
                <h2>A field guide</h2>
                <p>A small collection of specific places, with notes. Click a marker for details.</p>
                <div class="field-guide-wrap">
                    <div id="field-guide-map" class="field-guide-map" aria-label="Map of recommended places in Cyprus"></div>
                </div>
            </section>
```

- [ ] **Step 3: Add TOC entries to both the sidebar and mobile TOC**

In the `<nav class="toc-mobile">` block, after the `<li><a href="#food">The food</a></li>` entry, add:

```html
                <li><a href="#field-guide">A field guide</a></li>
```

Do the same in the `<nav class="toc-sidebar">` block.

- [ ] **Step 4: Verify the page loads without console errors**

Start a local server: `python3 -m http.server 8000` (run in a separate terminal; leave it running for the rest of the plan).
Open `http://localhost:8000` in a browser.
Expected: page loads, scrolling to the bottom shows "A field guide" heading and intro paragraph, the TOC includes a new "A field guide" link, no red errors in the browser console. The map div is empty — that's fine, we haven't initialized it yet.

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "add field-guide section scaffold and mapbox cdn"
```

---

## Task 3: Add field-guide CSS

**Files:**
- Modify: `index.html` (inside the existing `<style>` block)

- [ ] **Step 1: Add field-guide styles inside the existing `<style>` block**

Find the `/* ── Footer ── */` comment inside the `<style>` block (around line 303). Insert the following CSS block immediately *before* it:

```css
        /* ── Field guide map ── */
        .field-guide-wrap {
            position: relative;
            margin: 0 0 1.5rem 0;
            border-radius: 3px;
            overflow: hidden;
            border: 1px solid var(--color-border);
        }

        .field-guide-map {
            width: 100%;
            height: 70vh;
            max-height: 600px;
            min-height: 420px;
            background: #f4e8d0;
        }

        @media (max-width: 680px) {
            .field-guide-wrap {
                margin-left: -1.5rem;
                margin-right: -1.5rem;
                border-radius: 0;
                border-left: none;
                border-right: none;
            }
        }

        #field-guide .mapboxgl-marker {
            cursor: pointer;
        }

        .fg-marker {
            width: 10px;
            height: 10px;
            border-radius: 50%;
            background: #8b1a1a;
            border: 1.5px solid #3a2410;
            box-shadow: 0 0 0 2px rgba(244, 232, 208, 0.9);
            transition: transform 0.15s ease;
        }

        .fg-marker:hover {
            transform: scale(1.4);
        }
```

- [ ] **Step 2: Verify the map container has the expected dimensions**

Reload `http://localhost:8000` in the browser.
Expected: scrolling to the field-guide section shows an empty cream-colored rectangle (no map yet) with the correct proportions — about 70% of the viewport height, capped at 600px, with a thin border. On a mobile width (<680px), the container stretches edge-to-edge.

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "style field-guide section container and marker"
```

---

## Task 4: Initialize the Mapbox map

**Files:**
- Modify: `index.html` (inside the existing `<script>` block at the bottom)

Prerequisite: the site owner has provided `<MAPBOX_TOKEN>` and `<STYLE_URL>` per the Prerequisites section.

- [ ] **Step 1: Add map initialization code inside the existing `<script>` block**

Find the closing `</script>` tag near the end of `index.html` (around line 691). Insert the following code *before* `</script>`, after the existing image fade-in code:

```javascript
        // ── Field guide map ──
        (function initFieldGuideMap() {
            const MAPBOX_TOKEN = '<MAPBOX_TOKEN>';
            const STYLE_URL = '<STYLE_URL>';
            const CYPRUS_BOUNDS = [[32.2, 34.5], [34.6, 35.75]];

            if (!window.mapboxgl) {
                console.warn('Mapbox GL JS failed to load');
                return;
            }

            mapboxgl.accessToken = MAPBOX_TOKEN;

            const map = new mapboxgl.Map({
                container: 'field-guide-map',
                style: STYLE_URL,
                bounds: CYPRUS_BOUNDS,
                fitBoundsOptions: { padding: 40 },
                maxBounds: [[31.8, 34.3], [35.0, 35.95]],
                minZoom: 7,
                maxZoom: 14,
                cooperativeGestures: true
            });

            map.addControl(new mapboxgl.NavigationControl({ showCompass: false }), 'top-right');

            // expose for later tasks
            window.__fieldGuideMap = map;
        })();
```

- [ ] **Step 2: Replace the token and style URL placeholders**

In the code you just added, replace the literal string `<MAPBOX_TOKEN>` with the owner's actual public token (e.g. `pk.eyJ1Ijoi...`) and `<STYLE_URL>` with the owner's actual style URL (e.g. `mapbox://styles/aristotelis/clxxxxx`).

- [ ] **Step 3: Verify the map renders with the parchment style**

Reload `http://localhost:8000` and scroll to the field-guide section.
Expected: the Mapbox map loads and displays Cyprus styled per the Mapbox Studio style (parchment/sepia palette). Regular mouse-wheel scrolling over the map scrolls the page; a brief overlay appears telling the user to Ctrl/⌘+scroll to zoom. Holding Ctrl (or ⌘ on Mac) and scrolling zooms the map. Attempting to drag far off Cyprus snaps back. The zoom-in/zoom-out buttons appear top-right. No console errors.

(`cooperativeGestures: true` is Mapbox's built-in option that enables the Ctrl/⌘+scroll behavior and shows the overlay hint; do not also set `scrollZoom: false` — the two conflict.)

If the map is blank or errors: check the browser console for a Mapbox 401/403 (bad token or missing URL referrer restriction), 404 (bad style URL), or CORS error.

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "initialize mapbox map with parchment style and bounds"
```

---

## Task 5: Load recommendations and render markers

**Files:**
- Modify: `index.html` (extend the IIFE added in Task 4)

- [ ] **Step 1: Add data loading and marker creation inside the IIFE**

Inside the `initFieldGuideMap` IIFE, find the `window.__fieldGuideMap = map;` line. Insert the following code *before* it:

```javascript
            map.on('load', async () => {
                let data;
                try {
                    const res = await fetch('assets/recommendations.json');
                    if (!res.ok) throw new Error('HTTP ' + res.status);
                    data = await res.json();
                } catch (err) {
                    console.error('Failed to load recommendations.json:', err);
                    return;
                }

                const entries = Array.isArray(data.recommendations) ? data.recommendations : [];
                for (const entry of entries) {
                    if (!entry.name || !Array.isArray(entry.coords) || entry.coords.length !== 2 || !entry.maps_url) {
                        console.warn('Skipping invalid entry:', entry);
                        continue;
                    }
                    const el = document.createElement('div');
                    el.className = 'fg-marker';
                    el.setAttribute('role', 'button');
                    el.setAttribute('aria-label', entry.name);
                    el.setAttribute('tabindex', '0');
                    el.dataset.name = entry.name;

                    new mapboxgl.Marker({ element: el })
                        .setLngLat(entry.coords)
                        .addTo(map);
                }
            });
```

- [ ] **Step 2: Verify the three sample markers appear**

Reload `http://localhost:8000`, scroll to the field-guide section, wait for the map to finish loading.
Expected: three red-brown ink dot markers appear at Blue Lagoon (western tip of Akamas), Khirokitia (south-central), and Tombs of the Kings (Paphos). Hovering a marker makes it grow slightly. No console errors.

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "load recommendations and render markers on map"
```

---

## Task 6: Show a popup when a marker is clicked

**Files:**
- Modify: `index.html` (extend marker creation in the IIFE)

- [ ] **Step 1: Add popup creation and a helper to render popup content**

Inside the IIFE, *above* the `map.on('load', ...)` handler, add this helper function:

```javascript
            function renderPopupHTML(entry) {
                const imageHTML = entry.image
                    ? `<img class="fg-popup-img" src="${entry.image}" alt="${entry.name}" loading="lazy">`
                    : '';
                const noteHTML = entry.note
                    ? `<p class="fg-popup-note">${entry.note.replace(/\n\n/g, '</p><p class="fg-popup-note">')}</p>`
                    : '';
                return `
                    <h3 class="fg-popup-title">${entry.name}</h3>
                    ${imageHTML}
                    ${noteHTML}
                    <a class="fg-popup-link" href="${entry.maps_url}" target="_blank" rel="noopener noreferrer">Open in Google Maps →</a>
                `;
            }
```

- [ ] **Step 2: Wire the popup into the marker creation loop**

Inside the `map.on('load', ...)` handler, replace the existing marker-creation loop body with one that also creates a popup and attaches a click handler. Replace this block:

```javascript
                    const el = document.createElement('div');
                    el.className = 'fg-marker';
                    el.setAttribute('role', 'button');
                    el.setAttribute('aria-label', entry.name);
                    el.setAttribute('tabindex', '0');
                    el.dataset.name = entry.name;

                    new mapboxgl.Marker({ element: el })
                        .setLngLat(entry.coords)
                        .addTo(map);
```

with this:

```javascript
                    const el = document.createElement('div');
                    el.className = 'fg-marker';
                    el.setAttribute('role', 'button');
                    el.setAttribute('aria-label', entry.name);
                    el.setAttribute('tabindex', '0');
                    el.dataset.name = entry.name;

                    const popup = new mapboxgl.Popup({
                        offset: 14,
                        closeButton: true,
                        closeOnClick: true,
                        maxWidth: '280px',
                        className: 'fg-popup'
                    }).setHTML(renderPopupHTML(entry));

                    new mapboxgl.Marker({ element: el })
                        .setLngLat(entry.coords)
                        .setPopup(popup)
                        .addTo(map);
```

- [ ] **Step 3: Verify clicking a marker opens a popup with the expected content**

Reload `http://localhost:8000`, scroll to the field-guide section, click the Blue Lagoon marker.
Expected: a popup opens anchored above the marker containing the name "Blue Lagoon", the blue-lagoon WebP image, the italic note about the boat trip, and an "Open in Google Maps →" link. Clicking the link opens a new tab to the real Google Maps URL. Clicking another marker closes the current popup and opens the new one. Clicking the map background closes the popup.

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "show popup with details on marker click"
```

---

## Task 7: Style the popup to match the parchment palette

**Files:**
- Modify: `index.html` (inside the existing `<style>` block, extending the field-guide CSS)

- [ ] **Step 1: Add popup styles to the field-guide CSS**

Find the `.fg-marker:hover` rule from Task 3. Insert the following CSS directly after it, still inside the `/* ── Field guide map ── */` block:

```css
        .fg-popup .mapboxgl-popup-content {
            background: #f4e8d0;
            border: 1px solid #5a3820;
            border-radius: 3px;
            padding: 1rem 1.1rem 0.9rem;
            font-family: var(--font-serif);
            color: var(--color-text);
            box-shadow: 0 2px 12px rgba(90, 56, 32, 0.2);
        }

        .fg-popup .mapboxgl-popup-tip {
            border-top-color: #f4e8d0;
            border-bottom-color: #f4e8d0;
        }

        .fg-popup .mapboxgl-popup-close-button {
            color: #5a3820;
            font-size: 1.25rem;
            padding: 0 0.4rem;
        }

        .fg-popup-title {
            font-family: var(--font-serif);
            font-weight: 500;
            font-size: 1.15rem;
            color: var(--color-text);
            margin: 0 0 0.5rem 0;
            line-height: 1.3;
        }

        .fg-popup-img {
            display: block;
            width: 100%;
            height: auto;
            max-height: 140px;
            object-fit: cover;
            border-radius: 2px;
            margin: 0 0 0.6rem 0;
        }

        .fg-popup-note {
            font-style: italic;
            font-size: 0.92rem;
            line-height: 1.55;
            color: var(--color-text);
            margin: 0 0 0.7rem 0;
        }

        .fg-popup-note:last-of-type {
            margin-bottom: 0.7rem;
        }

        .fg-popup-link {
            display: inline-block;
            font-family: var(--font-sans);
            font-size: 0.78rem;
            color: var(--color-accent);
            text-decoration: none;
        }

        .fg-popup-link:hover {
            text-decoration: underline;
        }
```

- [ ] **Step 2: Verify the popup matches the parchment aesthetic**

Reload `http://localhost:8000`, scroll to the field guide, click a marker.
Expected: the popup has a cream/parchment background with a thin sepia border. The title is in Crimson Pro, the note is in italic serif, the image (when present) sits between them, and the "Open in Google Maps →" link appears in the existing teal accent color (Inter font). The popup tip/arrow is the same cream color.

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "style popup with parchment palette"
```

---

## Task 8: Add a decorative compass rose overlay

**Files:**
- Modify: `index.html` (extend `<section id="field-guide">` HTML and the field-guide CSS)

- [ ] **Step 1: Add the compass rose SVG inside the map wrapper**

In the `<section id="field-guide">` block, find the `<div id="field-guide-map" ...></div>` line. Insert this SVG directly after it, still inside `.field-guide-wrap`:

```html
                    <svg class="fg-compass" viewBox="0 0 60 60" aria-hidden="true">
                        <circle cx="30" cy="30" r="22" fill="none" stroke="#5a3820" stroke-width="1"/>
                        <circle cx="30" cy="30" r="14" fill="none" stroke="#5a3820" stroke-width="0.7"/>
                        <path d="M30 8 L33 30 L30 52 L27 30 Z" fill="#5a3820"/>
                        <path d="M8 30 L30 27 L52 30 L30 33 Z" fill="#5a3820" opacity="0.55"/>
                        <text x="30" y="6" text-anchor="middle" fill="#5a3820" font-family="Georgia, serif" font-size="7" font-style="italic">N</text>
                    </svg>
```

- [ ] **Step 2: Add compass positioning CSS**

Inside the `/* ── Field guide map ── */` block, add:

```css
        .fg-compass {
            position: absolute;
            top: 12px;
            left: 12px;
            width: 54px;
            height: 54px;
            pointer-events: none;
            opacity: 0.75;
            z-index: 1;
        }

        @media (max-width: 680px) {
            .fg-compass {
                width: 42px;
                height: 42px;
                top: 10px;
                left: 10px;
            }
        }
```

(Placed top-left so it doesn't collide with the Mapbox navigation control in the top-right.)

- [ ] **Step 3: Verify the compass appears over the map**

Reload `http://localhost:8000`, scroll to the field guide.
Expected: a small sepia compass rose appears in the top-left corner of the map, overlaying the map canvas. It does not intercept clicks (panning and clicking through it works).

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "add compass rose overlay to field-guide map"
```

---

## Task 9: End-to-end verification with Playwright

This task uses the Playwright MCP browser tools to confirm the full interactive behavior. No new commit — it's verification only.

- [ ] **Step 1: Make sure the local server is still running**

Run `curl -sf http://localhost:8000/ > /dev/null && echo UP || echo DOWN`.
Expected: `UP`. If `DOWN`, restart `python3 -m http.server 8000` in a separate terminal.

- [ ] **Step 2: Navigate to the page in Playwright**

Call `mcp__plugin_playwright_playwright__browser_navigate` with `url: "http://localhost:8000/#field-guide"`.
Expected: the page loads and jumps to the field-guide anchor.

- [ ] **Step 3: Wait for the map to initialize, then snapshot**

Call `mcp__plugin_playwright_playwright__browser_wait_for` with `time: 3` (seconds) to let the map finish loading.
Then call `mcp__plugin_playwright_playwright__browser_snapshot`.
Expected: the snapshot includes `.fg-marker` elements (the three markers) and the `#field-guide-map` canvas.

- [ ] **Step 4: Evaluate marker count via Playwright**

Call `mcp__plugin_playwright_playwright__browser_evaluate` with:

```javascript
() => document.querySelectorAll('#field-guide .mapboxgl-marker').length
```

Expected return value: `3`.

- [ ] **Step 5: Click a marker and assert the popup appears**

Call `mcp__plugin_playwright_playwright__browser_evaluate` with:

```javascript
() => { document.querySelector('#field-guide .mapboxgl-marker .fg-marker').click(); return true; }
```

Then call `mcp__plugin_playwright_playwright__browser_evaluate` with:

```javascript
() => {
  const popup = document.querySelector('.mapboxgl-popup.fg-popup');
  if (!popup) return { ok: false, reason: 'no popup' };
  const title = popup.querySelector('.fg-popup-title')?.textContent;
  const link = popup.querySelector('.fg-popup-link')?.getAttribute('href');
  return { ok: true, title, link };
}
```

Expected: `ok: true`, `title` is one of the three recommendation names, `link` starts with `https://maps.app.goo.gl/`.

- [ ] **Step 6: Check the browser console for errors**

Call `mcp__plugin_playwright_playwright__browser_console_messages` with `onlyErrors: true`.
Expected: empty list, or only warnings (no errors).

- [ ] **Step 7: If anything failed in steps 3–6, diagnose and fix before marking complete**

Common causes: bad token → 401/403 on tile requests; bad style URL → 404; typo in CSS class names between popup markup and styles.

No commit — verification only.

---

## Done

When all nine tasks are checked off:
- The site shows a parchment-styled map of Cyprus below the food section
- Three sample markers render at their correct positions
- Clicking a marker opens a popup with name, optional image, note, and working Google Maps link
- Scrolling over the map scrolls the page; Ctrl/⌘+scroll zooms the map
- The map is bounded to the island
- A compass rose decorates the corner
- The site owner can add new places by appending entries to `assets/recommendations.json`

## Follow-on work (explicitly out of scope)

- Iterating the Mapbox Studio style to get closer to hand-drawn Tolkien aesthetic
- Replacing the placeholder `maps.app.goo.gl` URLs with real ones the owner verifies
- Adding more recommendations beyond the initial three
- Decorative border frame around the map (low priority, easy to add later)
- Categories/filters (explicitly YAGNI per the design spec)
