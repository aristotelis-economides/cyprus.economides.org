# Interactive field-guide map — design

**Date:** 2026-04-22
**Status:** Approved, pending implementation plan

## Goal

Add an interactive map of Cyprus to `cyprus.economides.org` so visitors get concrete, curated recommendations (places to actually go) instead of only prose. The map should feel hand-made — parchment background, ink-stroked coastline, sepia hillshading, calligraphic labels — in the visual spirit of Tolkien's Middle-earth maps. Underneath the styling, it is a real, geographically accurate map (Mapbox GL JS), not an illustration.

Data lives in a single JSON file the site owner edits directly: each entry has a name, coordinates, a Google Maps link, an optional image, and an optional personal note.

## Non-goals (YAGNI)

- Categories, filters, or a legend
- Search / autocomplete
- Google Places API integration or remotely fetched imagery
- Permalinks to individual markers
- Route drawing or trip planning
- Multiple map layers or toggles

## Architecture

- **Map library:** Mapbox GL JS v3 loaded from Mapbox's CDN. No bundler — matches the site's existing zero-build static-HTML posture.
- **Map style:** Authored in Mapbox Studio, published there, referenced by style URL (e.g. `mapbox://styles/<user>/<style-id>`). Keeps styling iterable in the visual editor without touching the repo.
- **Mapbox token:** Public token restricted by URL referrer to `cyprus.economides.org` and `localhost`. Embedded inline in `index.html`.
- **Data:** `assets/recommendations.json` fetched at runtime, converted to a GeoJSON `FeatureCollection` in JS, added as a Mapbox source with a symbol/circle layer OR as DOM-based `Marker` instances (see §Markers).
- **Module boundary:** A single `<section id="field-guide">` block that is self-contained — its CSS is prefixed `.field-guide` and its JS is scoped to an IIFE. The section can be moved anywhere on the page without side-effects.

## Data schema (`assets/recommendations.json`)

```json
{
  "recommendations": [
    {
      "name": "Blue Lagoon",
      "coords": [32.3217, 35.0739],
      "maps_url": "https://maps.app.goo.gl/xxx",
      "image": "assets/blue-lagoon.webp",
      "note": "Worth the boat or the 4x4 track. Go early."
    }
  ]
}
```

| Field | Required | Notes |
|---|---|---|
| `name` | yes | Displayed as popup title |
| `coords` | yes | `[longitude, latitude]` — GeoJSON order, not `[lat, lng]` |
| `maps_url` | yes | Rendered as "Open in Google Maps →" link |
| `image` | no | Relative path to a WebP/JPG in `assets/`. Omit to skip image. |
| `note` | no | Plain string. Use `\n\n` to separate paragraphs. |

Validation: at runtime, the loader skips any entry missing required fields and logs a console warning. No build-time validation (keeping the no-build posture).

## Visual design

### Base map style (Mapbox Studio)

- **Background:** cream `#f4e8d0`, with a faint paper-noise `background-pattern` if Studio permits a raster pattern upload
- **Water:** pale ink wash `#dcc9a0`, hatched fill-pattern
- **Coastline:** sepia ink `#5a3820`, stroke-width 1.5–2px
- **Terrain:** hillshade layer in sepia tones (no greens/blues) to render the Troodos range visibly
- **Labels:** only country, region, and village names. All POI labels, highway shields, and transit labels hidden.
  - Font: Crimson Pro Italic (uploaded to Mapbox Studio as a custom font)
  - Color: sepia `#3a2410`, light parchment halo for legibility
- **Roads:** hidden. Tolkien didn't draw roads.
- **Administrative boundaries:** hidden.

### Decorative overlays (HTML/CSS, layered above the map canvas)

- **Compass rose** in the top-right corner (inline SVG, sepia strokes, ~60×60px)
- **Thin decorative frame** around the map canvas (1px `#8a7455` border with small corner flourishes)
- **Section title** above the map: "A field guide" in Crimson Pro, matching the existing `h2` treatment

### Markers

- **Visual:** small red-brown ink dot, `#8b1a1a` with `#3a2410` 1.5px ring, 10px diameter; grows to 14px on hover; cursor pointer
- **Implementation:** Mapbox `Marker` instances (DOM-based) for easier CSS hover/transition and popup handling. For ~25 markers performance is fine; if we ever exceed ~100, switch to a `symbol` layer.

### Popup

- **Trigger:** click a marker; only one popup open at a time; clicking the map background closes it
- **Style:** parchment background `#f4e8d0`, sepia border `#5a3820`, the `mapboxgl-popup-tip` restyled to match
- **Content, top to bottom:**
  - `name` as an `h3` in Crimson Pro, sepia
  - optional image (full width of popup, `max-height: 180px`, `object-fit: cover`, rounded 3px)
  - optional `note` in Crimson Pro italic
  - "Open in Google Maps →" link in the existing site accent teal `#1a7a6d`

## Interaction

- **Initial view:** `map.fitBounds(CYPRUS_BOUNDS, { padding: 40 })` where `CYPRUS_BOUNDS = [[32.2, 34.5], [34.6, 35.75]]` (approximate)
- **Bounds lock:** `maxBounds` set slightly outside the island to prevent panning into empty ocean/Turkey
- **Zoom range:** `minZoom: 7`, `maxZoom: 14`
- **Scroll gesture:** `scrollZoom` disabled; re-enabled only when `Ctrl` or `⌘` is held (cooperative gesture pattern). Prevents the map from hijacking page scroll. Pinch-zoom on touch devices works normally.
- **Drag:** enabled within bounds.
- **Popup behavior:** clicking another marker closes the current popup and opens the new one.

## Page integration

- Added as `<section id="field-guide">` directly after the existing `<section id="food">` block in `index.html`.
- TOC entries added to both the sidebar TOC (`.toc-sidebar`) and the mobile TOC (`.toc-mobile`): "A field guide" → `#field-guide`.
- Section anatomy:
  - `h2`: "A field guide"
  - Optional intro sentence (~1 sentence, tone-matched to the essay)
  - `<div class="field-guide-map">` — the map canvas container, height `70vh` capped at `600px`
  - Full-bleed on mobile, matching the `.section-image` responsive pattern (`margin-left/-right: -1.5rem` at `max-width: 680px`)

## File structure

```
index.html                          # adds section, Mapbox CDN <link>/<script>, init code
assets/recommendations.json         # the data
```

All CSS and JS for the field guide lives inline in `index.html` `<style>` and `<script>` blocks to match the existing pattern. If either grows past ~200 lines, extract to `assets/field-guide.css` / `assets/field-guide.js`.

## Setup steps the site owner performs

These are prerequisites before implementation can finish:

1. Create a free Mapbox account; generate a **public** access token
2. In the token settings, restrict allowed URLs to `cyprus.economides.org` and `localhost:*`
3. In Mapbox Studio, start from a blank or Monochrome base style; tune layers toward the parchment palette described in §Visual design; publish the style; copy the style URL (`mapbox://styles/<user>/<style-id>`)
4. Share the token and style URL with the implementer

Alternative if the owner prefers not to use Studio: the implementer hand-authors `assets/map-style.json` from scratch using the Mapbox Style Spec. Slower to iterate visually but keeps the style in git.

## Risks and open questions

- **Style fidelity:** The parchment/ink aesthetic pushes Mapbox's style spec in directions it wasn't designed for (decorative strokes, hand-drawn feel). The result may end up looking "map with a sepia filter" rather than genuinely hand-made. Mitigation: iterate in Studio with a quick feedback loop; if the ceiling is too low, revisit the "single illustrated image" alternative.
- **Custom fonts in Studio:** Uploading Crimson Pro to Studio requires the `.ttf` or `.otf` files and may be limited on the free tier. Fallback: use a built-in serif italic that's closest in feel.
- **Token exposure:** Public tokens are safe to commit provided URL restrictions are set. If the owner skips restrictions, the token is abusable.
- **Mobile cooperative gesture:** Some users find Ctrl/⌘-to-zoom frustrating on laptops. If feedback is negative, switch to normal scroll-zoom and live with occasional scroll hijack.

## Success criteria

- A visitor who opens the page sees a parchment-styled map of Cyprus rendered within 2 seconds on a typical connection
- Clicking a marker opens a popup with name, optional image, optional note, and a working Google Maps link
- The site owner can add a new recommendation by appending one entry to `recommendations.json` — no code changes required
- The map section is visually cohesive with the existing page (serif type, sepia/cream palette, understated tone — no dramatic flourishes per the established writing-style memory)
- Scroll behavior on the surrounding page is not hijacked when the cursor is over the map
