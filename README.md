# Sketch to Asset

Draw an idea on paper, take a photo, and get a 3D model you can turn, light, recolour and export as OBJ or JPEG.

It is a single web page (`index.html`): no build step and no server code.

## Use it

1. Open the page and click **New from sketch** (or drop a photo onto the page). On a phone, this opens the camera.
2. Choose how to build it:
   - **Trace outline** works offline. It finds the closed outline of your drawing and puffs it into a solid. Use the sliders for ink sensitivity, closing gaps in lines, trimming the photo edges, size in mm, thickness and roundness, plus engraved inner lines. You can also cut out enclosed areas as holes. The preview shows in blue what will become solid.
   - **Claude AI** sends the photo to Claude, which reads the drawing (and any dimensions written on it) and builds the model from simple parts: boxes, spheres, cylinders, cones and rings. You need your own Anthropic API key from [console.anthropic.com](https://console.anthropic.com). The key goes straight from your browser to `api.anthropic.com`, and is saved only if you tick "Remember".
3. View it: drag to orbit, scroll or pinch to zoom, use the turntable, and show dimensions in mm.
4. **Light** tab: preset setups, plus light direction, height, strength and colour, fill, back light and shadows.
5. **Colour** tab: tap a part, then pick a colour and a finish.
6. **Export** tab: JPEG (1080 or 2048 px), OBJ + MTL in a .zip (with colours), or a plain OBJ. Units are millimetres, Y up.

Your models are saved in this browser (`localStorage`). Traced models have an **Adjust** button for changing their settings later.

## Run locally

Open `index.html` in a browser, or serve the folder:

```bash
python3 -m http.server 8000
```

Then visit http://localhost:8000.

## Deploy on GitHub Pages

Push this folder to a GitHub repository. Then go to Settings → Pages, choose "Deploy from a branch" and pick `main` / root.

## Notes

- Only external dependency: three.js r128, pinned, from cdnjs.
- See `HANDOFF.md` for how the code is organised and how to add models.
