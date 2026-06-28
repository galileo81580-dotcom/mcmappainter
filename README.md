# MC Map Painter

A browser tool that turns any image into a build guide for **Minecraft map art** — the
"staircase" style where each block's shade depends on its height relative to its northern
neighbour (▲ raised / ■ flat / ▼ lowered).

Drop in an image and the app dithers it to the real Minecraft map palette, then gives you a
zoomable, pannable block grid with per-block material info, a chunk-by-chunk column build
list, a materials shopping list, and PNG export.

**Live app:** https://galileo81580-dotcom.github.io/mcmappainter/

## Features

- **Load any image** — drag & drop, file picker, or the built-in sample. Automatic
  Floyd–Steinberg / ordered dithering to the Minecraft palette (no external pre-processing).
- **Adjustable map size** — set how many maps wide × tall (1 map = 128×128 blocks) with
  contain / cover / stretch fit.
- **Material control** — enable/disable any of the 53 block materials; the matcher only uses
  what you have, and the shopping list updates to match.
- **Fast viewport rendering** — renders only what's visible, so it stays smooth at any zoom
  even on multi-map builds (the original redrew every pixel every frame).
- **Block inspector** — hover any block for its material, placement, alternatives, and
  in-game / chunk coordinates (set your upper-left world X,Z).
- **Column build list** — click a block to see the 16-block chunk column; step through
  neighbouring columns with the on-screen buttons or the arrow keys (← → ↑ ↓).
- **Painting area** — click maps on the minimap to mark them *skipped* (dimmed, and excluded
  from the shopping list).
- **Materials shopping list** — total blocks per material, stack count, across painted maps.
- **Export** — the dithered result at 1× (1 px per block) or the current view as PNG.

## Usage

It's a single self-contained `index.html` — open it directly, or serve the folder:

```bash
python3 -m http.server 8000
# then open http://localhost:8000
```

## How it works

`palette.json` maps every Minecraft map colour (hex → R/G/B, placement symbol, material,
alternative blocks). The palette is embedded directly into `index.html`, so the app needs no
network and no build step. Images are matched to the nearest palette colour in CIELAB space
with optional error-diffusion dithering.

## Repo layout

| Path | Purpose |
|------|---------|
| `index.html` | The entire app (palette embedded). |
| `palette.json` | Source palette, for editing / regenerating. |
| `examples/` | Sample dithered image and the original prototype for reference. |

To regenerate the embedded palette after editing `palette.json`, replace the array following
the `/*PALETTE_PLACEHOLDER*/` marker in `index.html`.

## License

MIT — see [LICENSE](LICENSE).
