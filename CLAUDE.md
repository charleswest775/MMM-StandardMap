# MMM-StandardMap — context for Claude sessions

Charles's MagicMirror² module: Chirikov's standard map, orbit after orbit plotted as dots, the KAM islands and the chaotic sea, at four kick strengths in turn, for his hallway mirror, as one page in a
rotation of pages. Made on 2026-09-29, one of five new pages (with MMM-ChaoticWaterwheel, MMM-DoubleSlit, MMM-Sandpile, MMM-Harmonograph),
with the same shell as its siblings. On the mirror since 2026-09-30.

## Files

- `MMM-StandardMap.js` — module shell: one canvas plus an HTML caption (equations + live readout,
  updated 2×/s). A new start every `cycleSeconds` and on each `resume()`. Loop: `setTimeout`
  until a frame is due, then one `requestAnimationFrame`. `suspend()` stops it; a sim with
  `resting = true` is polled only every 500 ms; while MagicMirror fades the module out (`hidden`
  is set at the start, `suspend()` comes after), frames draw nothing. `turns: { of, at }`: only
  every nth showing; otherwise the wrapper gets `display: none` and nothing starts.
- `simulations/standard-map.js` — the `standardMap` sim on `window.StandardMapSimulations`.
  `orbit(K, θ, p, n)` returns the points (θ, p mod 2π), the Lyapunov exponent (tangent map,
  after 500 kicks' warm-up) and the rotation number (p kept unwrapped). One orbit of 2,000 kicks
  per changed frame, 3 a second, 126 then rest (42 s: fits a 45 s page). Scenes K = 0.5,
  0.971635, 1.3, 2.4 in turn by a static counter (`StandardMap.shown`); `standardMapK` pins one.
  Chaotic orbits (λ > 0.02) draw only 500 dots, grey: the sea fills soon anyway.
- UMD-style, so it runs in Node: `tests/standard-map.test.js` (`node --test`, no dependencies): area kept, the fixed point's stability at K = 4, Chirikov's ln(K/2), Greene's K (no crossing at 0.9, crossing at 1.2), rotation numbers, and the page's timing.
- `node_helper.js` — the stats panel (`statsPanel: true`): CPU of Electron and cage, per core,
  temperature, from `/proc`, only while shown.
- `dev/preview.html` — runs the module in a desktop browser (`python3 -m http.server` in the repo,
  then `/dev/preview.html`).

MMM-ChaosTheory has the same simulation (`standardMap`, among its eleven): a fix to the physics or
the drawing belongs in both.

The shell (`MMM-StandardMap.js`, `node_helper.js`'s stats panel, `dev/preview.html`) is shared in
spirit with the sibling modules (MMM-ChaosTheory, MMM-LorenzAttractor, MMM-DoublePendulum, MMM-FractalBasins, MMM-LogisticMap, MMM-SymmetricIcons, MMM-ThreeBody, MMM-ChaoticBilliards, MMM-Rule30, MMM-ChaoticWaterwheel, MMM-Sandpile, MMM-Atom, MMM-DoubleSlit, MMM-FractalZoom, MMM-Chladni, MMM-SacredGeometry, MMM-Tilings, MMM-PlanetsDance, MMM-Harmonograph, MMM-SnowCrystal, MMM-NightSky, MMM-PhotoDeck): a fix there probably belongs in the siblings too.

## Cost on the Pi

Measured on the Pi, 2026-09-30 (900², 20 fps, Electron + cage over a 45 s page): 31% of a core; hidden 0.2%.

## Performance findings on the Pi (measured)

- A frame that changes the canvas costs ~2%/fps fixed; beyond that, cost scales with the
  **bounding box of everything changed in the frame**. Full redraws of a 900² canvas at 20 fps
  saturate the pipeline (~150%). JS is rarely the bottleneck (<3 ms/frame for most pages).
- So: draw incrementally, keep each frame's changes spatially compact, and rest when the picture
  is static. Line width, opacity, `rAF` vs timer made no difference.
- MagicMirror applies `electronSwitches` after app ready, so `remote-debugging-port` can't be set
  that way; use `debugStats: true` and a `grim` screenshot to see fps on the Pi.

## Hard constraints: the target device

- **Raspberry Pi 3 B+, 905 MB RAM, 64-bit Debian 13.** Mirror runs Electron 42 in a cage
  Wayland kiosk.
- **No GPU acceleration, and it can't be enabled**: the Pi 3's VideoCore IV only does GLES 2.0,
  Chromium needs ES 3.0 (tested). All canvas drawing is CPU. **No WebGL / three.js.**
- Screen will be **portrait 1200×1920** once mounted (Dell U2413, rotated). Design for portrait.
- Electron baseline is ~0.5% of one core. **Measure, don't guess**: on the Pi,
  `~/.cache/mm-sample.sh 60` prints Electron CPU% and RSS over 60 s. Record before/after numbers
  in the README.
- The mirror rotates pages every 15-45 s (MMM-pages, which hides/shows modules). `suspend()` and
  `resume()` must fire on page changes, or the loop burns CPU 24/7.

## Deploying and testing

- This repo is public so the Pi can `git clone`/`git pull` without credentials. It is meant for
  modules.magicmirror.builders: keep `screenshot.png` and the README's Installation / Update /
  Configuration sections, which the list's checks look for.
- Pi access: `ssh fatherson@raspberrypi.local` (key auth). Module path:
  `~/MagicMirror/modules/MMM-StandardMap`. Restart: `pm2 restart MagicMirror`
  (pm2 is in `~/.npm-global/bin`). Logs: `pm2 logs MagicMirror`.
- The mirror's **config.js lives in a separate private repo**, `charleswest775/magicmirror-setup`
  (cloned at `~/dev/magicmirror-setup`). Add the module's config block there, then
  `./deploy.sh diff` and `./deploy.sh push` (push validates config before restarting).
  Don't hand-edit config.js on the Pi without `./deploy.sh pull` afterwards.
- Faster iteration: run it in a desktop browser (`dev/preview.html`), then confirm performance
  on the Pi.
- Commit as Charles's GitHub noreply address (set in this repo's git config).
