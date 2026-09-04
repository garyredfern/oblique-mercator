# Oblique Mercator — reference guide

Living document. Updated at the end of each tranche.

## Purpose
Single-file web page that re-projects the whole world as an **oblique Mercator** centred on a chosen country, so that country is drawn at its true shape and relative size, and overlays its familiar **standard-Mercator** shape for comparison (e.g. Greenland: ~2.2 M km² true vs. the Africa-sized blob on a normal map).

## Decisions (locked)
| # | Question | Decision |
|---|----------|----------|
| 1 | Projection | Oblique Mercator — the *map* re-projects, not just an overlay |
| 2 | Centre | Spherical centroid of the selected country (`d3.geoCentroid`) |
| 3 | Input | Country/territory only, searchable list. No coordinates, no city search |
| 4 | Country list | Natural Earth admin-0 *countries* via `world-atlas@2` — **241 entries**, dependencies listed separately (Cayman Is., Greenland, Puerto Rico…) |
| 5 | Overlay | Option b: ghost of the standard-Mercator shape drawn over the true shape, same centroid |
| — | Cities/labels | Dropped. Borders must be realistic |
| — | Hosting | Single HTML file, CDN deps, Cloudflare Pages OK, not offline |

## Plan
| Tranche | Scope | Status |
|---|---|---|
| 1 | Load data, searchable dropdown, static rotated map, graticule, oblique-equator line, selection shading | **Done** |
| 2 | Zoom/pan (`d3.zoom`), reset view, zoom readout | **Done** — 50 m vs 10 m judgement deferred to user review |
| 3 | Standard-Mercator ghost overlay + area / stretch-factor readout | **Done** |
| 5 | Dark/light themes, canvas colours driven by CSS tokens | **Done** |
| 4 | Zoom-centring fix, 10 m toggle, zoom-dependent simplification, keyboard shortcuts, mobile layout, reduced motion | **Done** |

## Files
- `oblique-mercator.html` — the app (the only deliverable)
- `REFERENCE.md` — this guide

## Dependencies (CDN, jsdelivr)
- `d3@7` — projection, geoPath, zoom
- `topojson-client@3` — unpack world-atlas TopoJSON
- `topojson-simplify@3` — zoom-dependent vertex reduction
- `world-atlas@2/countries-50m.json` (~740 KB). `countries-10m.json` is 3.5 MB, option for tranche 4
- Google Fonts: Instrument Serif (display), IBM Plex Mono (data/UI)

## How the projection works
1. `c = d3.geoCentroid(country)` → `[lon, lat]`.
2. `d3.geoMercator().rotate([-lon, -lat, 0])` — rotate the sphere so `c` lands at (0°, 0°), then apply ordinary Mercator. The projection's equator (line of zero scale distortion) is therefore the great circle through `c`.
3. Graticule drawn is the **rotated frame's** graticule; the dashed amber line is that frame's equator mapped back to geographic coords via `d3.geoRotation(rotate).invert`.
4. Mercator's own ±85° latitude clip applies in the rotated frame, so the region around the rotated poles (near the antipode) is cut off rather than exploding. Tranche 2 will check this is visually acceptable at all zooms.

Verified headlessly (tranche 1): Greenland centroid 73.08° N, 41.81° W projects exactly to screen centre; true area 2.15 M km²; standard-Mercator areal stretch at that latitude ≈ ×11.8.

## Zoom / pan model
- `d3.zoom`, scale extent ×1–×200. Transform `k,x,y` is mapped onto `projection.scale(base·k)` and `translate(t.apply([W/2,H/2]))`.
- **Bug fixed in tranche 4:** originally `translate(W/2 + x, H/2 + y)`. d3.zoom assumes screen = k·p + t for base-view pixel p, so the centre point must also be multiplied by k; without that, every zoom step drifted the country toward the top-left. Verified: at k=8 about the centre the centroid stays at exactly (W/2, H/2).
- **Panning moves the viewport, not the rotation.** The oblique equator stays fixed through the chosen country; dragging never changes which point is distortion-free. Selecting a new country resets to ×1 centred.
- Line widths are in screen pixels, so borders don't fatten on zoom.
- Redraw is a full canvas repaint per zoom event.
- **Simplification by zoom** (`topojson-simplify`): `presimplify` once per dataset, then `simplify(threshold)` where threshold ≈ (degrees per screen pixel)², quantised to ×4 bands (−8…2) and cached. Measured: 50 m ranges 22 k verts at ×1 → 80 k fully zoomed; 10 m 31 k → 452 k. Rebuild cost ≤120 ms on a band change, never per frame. The selected feature is re-pointed to the new level's copy by `id` so shading stays correct.
- **10 m borders** (3.5 MB) load on demand via the "Borders" button; the search list is built once from whichever dataset loads first (same 241 ids in both).

## Ghost overlay (standard-Mercator comparison)
- Second projection: `d3.geoMercator().rotate([-λc, 0, 0])` — ordinary north-up Mercator — given the **same nominal scale** as the oblique map, then translated so the country's centroid lands on the same screen pixel as in the oblique view.
- Because the oblique map has scale factor 1 at the centroid, "same nominal scale" means the ghost is exactly the shape and size a wall map / Google Maps would draw at that scale. Orientation matches: at the centroid both projections have north straight up.
- Readout gives two numbers, deliberately different:
  - **linear stretch at centre** = 1/cos φc (a point value, e.g. Greenland ×3.44);
  - **ghost ÷ true area** = integrated over the whole shape via `geoPath.area` on unclipped projections (Greenland ×16.2, not 11.8 = 3.44², because stretch climbs steeply toward 83° N).
- True area = `d3.geoArea × R²`, R = 6371.0088 km (spherical; ~0.3 % from ellipsoidal). Greenland → 2.15 M km² (reference value 2.17 M).
- Toggle button hides/shows the ghost. Ghost colour token `--ghost #6FB3D2`.

## Interaction
- Scroll/pinch zooms about the cursor; drag pans; double-click zooms in.
- Keys (when the search box isn't focused): `/` focus search · `0` reset view · `g` toggle ghost · `Esc` leave search.
- `prefers-reduced-motion` → reset has no animation.
- ≤480 px wide: panel docks to the bottom edge, status/zoom readouts move to the top.

## Names
world-atlas abbreviates (`Cayman Is.`, `Dem. Rep. Congo`). `LONG_NAMES` in the HTML expands ~36 of them for the search box; both long and short forms are accepted as input.

## Known limitations / to watch
- Centroid of multi-part countries can fall in water (Indonesia, Philippines, USA incl. Alaska/Hawaii). Harmless for projection; worth noting in readout later.
- `datalist` search is browser-native: prefix/substring behaviour varies by browser (Chrome substring, Safari prefix). Revisit if it annoys.
- Antarctica as a selection is legitimate but the rotated frame puts the Arctic at the cut-off.
- No offline mode.

## Themes
`data-theme="dark" | "light"` on `<html>`; canvas reads every colour from the CSS variables at draw time, so a theme is purely a token block. Default follows `prefers-color-scheme`, choice remembered in `localStorage` (`om-theme`). Toggle via button or `t`.

| token | dark | light (Google-Maps-like) |
|---|---|---|
| ocean | `#0F1E2D` | `#A9D3F2` |
| land | `#2A3A46` | `#EDE6D3` |
| border | `#7F97A6` | `#B3A78F` |
| graticule | `#30455A` | `#8FC0E4` |
| select / equator | `#E8B04B` | `#C46A00` |
| ghost | `#6FB3D2` | `#1D5FA8` |
| ink / muted | `#E9EEF2` / `#8FA3B3` | `#1F2A33` / `#5E6C78` |

## Design tokens
Ghost `#6FB3D2` · Ocean `#0F1E2D` · land `#2A3A46` · borders `#7F97A6` · graticule `#30455A` · selection/equator `#E8B04B` · ink `#E9EEF2` · muted `#8FA3B3`.
Signature element: the projection's equator drawn as a dashed amber great circle through the chosen country — the line of zero distortion is the whole thesis of the tool.
