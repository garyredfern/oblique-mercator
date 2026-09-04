# Oblique Mercator — the world, re-centred on one country

**Live:** https://YOUR-PROJECT.pages.dev *(replace with your link)*

Pick any of 241 countries and territories and the whole world re-projects as an
**oblique Mercator** whose equator runs through that country's centre. Because
Mercator has zero distortion along its equator, the chosen country is drawn at
its **true shape and relative size** — and a dotted "ghost" shows the inflated
shape a standard north-up Mercator map (the Google-Maps / wall-map view) would
draw at the same nominal scale.

The classic demo: select **Greenland**. Its familiar map-shape turns out to be
about **16× the area** of the real thing — more than the ~11.8× you'd guess
from squaring the stretch factor at its centre, because Mercator's distortion
compounds rapidly toward 83° N.

## Features

- Searchable list of 241 Natural Earth admin-0 countries and territories
  (dependencies listed separately)
- Full-screen canvas map: scroll/pinch zoom (×1–×200), drag pan, double-click zoom
- The projection's equator drawn as a dashed line — the locus of zero distortion
- Ghost overlay toggle, true-area and stretch-factor readout
- Borders at Natural Earth 50 m, optional 10 m on demand, with zoom-dependent
  simplification (`topojson-simplify`) so high zoom stays smooth
- Dark and light (Google-Maps-ish) themes; follows your OS preference,
  remembers your choice
- Keyboard: `/` search · `0` reset view · `g` ghost · `t` theme
- Single HTML file. No build step, no server, no tracking.

## How it works

1. `d3.geoCentroid` gives the country's spherical centroid *(λc, φc)*.
2. `d3.geoMercator().rotate([-λc, -φc, 0])` rotates the sphere so that point
   sits at (0°, 0°), then applies ordinary Mercator — an oblique Mercator whose
   equator is the great circle through the centroid.
3. The ghost is a second, un-tilted Mercator (`rotate([-λc, 0, 0])`) at the
   same nominal scale, translated so both centroids share a screen pixel.
4. Panning moves the viewport, never the rotation — the distortion-free line
   stays locked through the chosen country.

Build notes, decisions, and the maths in more detail: [REFERENCE.md](REFERENCE.md).

## Running locally

Open `index.html` in a browser. That's it. (Data and libraries load from CDNs,
so you need to be online.)

## Credits

- Borders: [Natural Earth](https://www.naturalearthdata.com/) (public domain),
  packaged by [world-atlas](https://github.com/topojson/world-atlas)
- [D3](https://d3js.org/), [topojson-client](https://github.com/topojson/topojson-client),
  [topojson-simplify](https://github.com/topojson/topojson-simplify)
- Fonts: Instrument Serif, IBM Plex Mono (Google Fonts)

Built collaboratively with Claude (Anthropic).

## License

[MIT](LICENSE)
