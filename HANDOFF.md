# Sketch Model Viewer: handoff to Claude Code

From this point the code is edited **only in Claude Code**. The earlier version in the Claude chat is frozen and should not be edited further. `index.html` in this folder is the single source of truth.

## What it is

A single-file web page (`index.html`, no build step) that turns the owner's paper sketches into interactive 3D models. It started as one sketched character and now holds two models and room for a third.

- **Models:** a figure (from the first sketch) and a timber table (from the second sketch), plus any models the owner makes from their own sketch photos (see "Sketch upload" below). A card row at the top switches between them. The last card, **New from sketch**, adds one.
- **Light tab:** four setups (Studio, Sunset, Moody, Flat), main light direction, height, strength and colour, soft fill, back light, shadows, and a light-position marker.
- **Colour tab:** pick a part (chips, or tap it on the model), set its colour with the picker or a swatch, and adjust finish (glossy to matte). The metal rod on the table keeps its own finish.
- **Dimensions toggle:** draws length, depth and height lines in mm with labels. Labels are also drawn into saved JPEGs.
- **Export tab:** JPEG (1080 or 2048 px wide) and OBJ. The OBJ comes as a `.zip` containing `<name>.obj` and `<name>.mtl` (the colours).
- **Controls:** drag to orbit, wheel or pinch to zoom, tap a part to select its colour, turntable switch, reset view.
- **Font:** Helvetica (`"Helvetica Neue", Helvetica, Arial, sans-serif`), as requested by the owner. No web fonts are loaded.

## Run and deploy

- **Local:** open `index.html` in a browser, or run `python3 -m http.server` in this folder and visit `http://localhost:8000`. There are no fetches, so both work.
- **Deploy:** the owner already hosts Claude Code projects on GitHub Pages. Put `index.html` at the repo root (or in `/docs`) and push.
- **Only external dependency:** three.js r128, pinned, from `https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js`. Keep the version pinned. Some APIs differ in newer three.js releases (colour management, `outputEncoding`, `Geometry` removal).

## Structure of `index.html`

1. `<style>`: all colours are CSS tokens on `:root`, with light and dark themes (`prefers-color-scheme` and `data-theme`). Layout is a card row, then a stage plus a 340 px control panel (stacked on phones).
2. Markup: `nav.models` cards, `header`, `#stage` (canvas, dimension labels, overlay buttons), tabbed panel.
3. **Core block** between `/*CORE-START*/` and `/*CORE-END*/`. Plain JS that depends only on a global `THREE`, no DOM. It holds the colour groups, `makeMats`, `buildFigure`, `buildTable`, `finishModel`, `buildOBJ`, `buildMTL`. Anything that creates geometry or writes OBJ belongs here, so it can be unit-tested in Node with the real `three@0.128.0` package.
4. App block (an IIFE below the core): renderer, lights, `entries` (one per model), camera orbit, dimensions, colour UI, thumbnails, export.

### How a model is defined

Each model is added with `addEntry({ id, label, file, groups, build, names, note?, user?, record? })`.

- `groups`: list of `{ id, label, color, rough?, metal?, fixed? }`. `fixed: true` means the finish slider does not touch it.
- `build(mats)`: returns `finishModel(model, meshes)`. Every mesh needs `name`, `userData.group` (a group id), and `castShadow` / `receiveShadow`.
- `names`: the three dimension labels for x, z, y (the figure uses Width, Depth, Height; the table uses Length, Depth, Height).
- Everything else is derived from the model's bounding box: camera distance, shadow bounds, floor size, light distance, marker size, dimension lines, thumbnail.

### To add another built-in model

1. Add a groups array and a `buildX(mats)` in the core block.
2. Add an `addEntry({...})` call next to the figure and table ones in the app block. Cards are generated, so no markup is needed.

## Conventions

- Units: **1 unit = 1 mm. Y is up.** Each model is lowered so its lowest point sits at y = 0.
- OBJ export: each part is its own named object (`o Name`, `usemtl <group id>`). Vertices are welded by position per part so seams are closed. Normals and UVs are included.
- The MTL has one material per colour group (Kd from the current colour, Ks and Ns from finish and metalness).
- Export file names come from the model's `file` field (`sketch-figure`, `timber-table`).

## Sketch interpretation and assumptions to confirm

**Figure** (traced from the first sketch, proportions measured from the drawing): egg-shaped body, round head with two eyes (white with dark pupils), a nose to the right, a dark mouth, small arm and hand ellipsoids, two leg ellipsoids with feet. One foot is raised (walking pose). About 98 mm tall, 70 mm wide.

**Timber table** (second sketch): 2 m long and 900 mm deep were written on the sketch. These were **not** on the sketch, so they were assumed:

| Item | Assumed value |
| --- | --- |
| Overall height | about 750 mm (derived from the values below) |
| Slab top thickness | 60 mm |
| Legs | four planks, 110 × 70 mm section, splayed 7° outward across the width |
| Leg positions | 780 mm from the centre along the length, 290 mm from the centre across the width at the top |
| Joints | four through-tenon blocks on the top, 100 × 50 mm, standing 3 mm proud, in a darker timber colour |
| Metal rod | 22 mm diameter, 860 mm long, one through each pair of legs at 260 mm height, poking out about 50 mm each side |

The sketch also hints at a small peg or hole in each leg and angled cuts at the feet. Neither is modelled. Ask the owner for the real dimensions before treating the table as a build reference.

## Behaviour that changes outside Claude

In the Claude chat version, saving goes through Claude's `downloads` capability (a confirmation prompt, and `.obj` is not an allowed file type, hence the zip). In this standalone file, `window.claude` does not exist, so `save()` falls back to a normal browser download via a Blob link (`localDownload`). The zip is kept so the OBJ and MTL travel together.

## Verified and not verified

- **Checked in Node:** the core block's geometry and OBJ output were run with a stand-in for three.js (the sandbox could not install the package). Table bounding box 2000 × 900 × about 750 mm, figure about 70 × 40 × 98 mm, OBJ and MTL generated for both. The inline script passes `node --check`.
- **Not checked in a browser:** the rendered page itself. Look first at:
  - Thumbnails drawn at start-up by `refreshThumb` / `snapshot`.
  - Shadow quality on the table (`key.shadow.normalBias` and `bias` are tuned on the figure scale and scaled by model size).
  - Dimension label placement and the JPEG label overlay (`drawLabels`).
  - Layout on a phone (about 400 px wide) and in dark mode.
  - That a saved zip opens, and that the OBJ imports into the owner's tools with colours intact.

## Owner context

The owner usually models in Shapr3D, exports OBJ, and imports it into PrusaSlicer for printing. They also design timber pieces. Sketch-to-3D is the point of this project. They find it confusing when code is edited in two places, so keep all code changes here.

## Sketch upload (added in Claude Code, Nov 2026)

The empty third slot was replaced by a **New from sketch** card. Upload a photo (file picker, which opens the camera on phones, or drag and drop). The page then builds a model in one of two ways.

**Trace outline (offline).** Core functions: `traceSketch(img, p)` and `buildTraced(mats, tr, p)`.
1. Adaptive threshold (pixel against local mean) on a 2× analysis image finds the ink. This copes with uneven light on a photographed page.
2. Ink is dilated by "close gaps" steps. The background is flood-filled in from the border, and whatever the flood cannot reach is solid. That is then eroded back (a morphological closing).
3. Components touching the photo edge (table, page border) and specks are dropped. An optional "holes" pass cuts out enclosed areas other than each shape's largest one.
4. A chamfer distance field gives a height profile: quarter-circle "roundness" from the rim, flat once at full thickness. Inner ink lines away from the rim are engraved on the front.
5. The mesh is a front and back grid with Laplacian-smoothed outline, plus a thin rim wall (8% of thickness). Front and back therefore never share vertices, and the result is closed and manifold. This was checked in the browser: 0 open edges, 0 duplicated directed edges, positive volume, including with holes.
- Sketch = front view (X right, Y up), thickness along Z. Dimension names: Width, Thickness, Height.

**Claude AI.** `askClaude()` in the app block calls `POST https://api.anthropic.com/v1/messages` directly from the browser with the owner's key. Headers: `anthropic-version: 2026-10-21`, `anthropic-dangerous-direct-browser-access: true`, and `anthropic-beta: server-side-fallback-2026-07-01`. Body: `claude-opus-5-5` by default (Sonnet 5.5 selectable), adaptive thinking, `output_config.effort: "high"`, structured output via `output_config.format` (`SPEC_SCHEMA`), and `fallbacks: "default"`. These are raw `fetch` calls, not the SDK, because the page has no build step. Refusals and `max_tokens` cut-offs are reported to the user. The reply is tidied by `normalizeSpec()` (cleans ids and colours, unknown groups fall back to the first one, duplicate names get numbered) and built by `buildFromSpec()`. Shapes: box, sphere, cylinder, cone and torus, sized by full extents in mm, rotated by Euler XYZ in degrees.

**Persistence.** User models are stored in `localStorage` under `smv-models-v1`. Trace models keep a ≤1568 px JPEG and their settings, and are re-traced on load. AI models keep the parts list. Trace settings are remembered under `smv-trace-params`, and the API key under `smv-anthropic-key` only if "Remember" is ticked. Cards are rendered from `entries` (`renderCards`). User cards have Adjust (trace only) and Remove.

**Export additions.** "Save OBJ only" alongside the zip. Outside Claude, normal browser downloads are used.

### Not verified

- A real Claude API call with a valid key. The request shape was checked against a stubbed `fetch`, and a call with an invalid key reached the API (CORS fine, 401 handled). Body validation (structured output + `fallbacks` on version `2026-10-21`) has not been exercised with a valid key.
- Real phone photos. The tests used synthetic drawings with a lighting gradient and a dark table edge.
- Import of the traced OBJ into Shapr3D or PrusaSlicer.

## Ideas, not started

- STL export for slicers directly.
- Trace from a side view as well, and intersect the two for a truer 3D form.
- Per-model dimension extras on the table (top thickness, leg splay) beyond the three overall dimensions.
- Remember colours and light setup between visits.
