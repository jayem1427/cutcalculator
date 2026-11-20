# djr_cutcalculator

A single-page milling speeds, feeds and cutting-load calculator for side milling, slotting and high-speed (dynamic) toolpaths. It uses Sandvik Coromant's specific-cutting-force model, with wrought aluminium calibrated against Sandvik CoroPlus ToolGuide.

**Live:** https://jayem1427.github.io/djr_cutcalculator/

## Features

- Spindle speed, feed rate, material removal rate, net and motor power, spindle torque and cutting force
- Exact average chip thickness for any radial engagement, plus a chip-thinning factor
- To-scale top view and cross-section showing stepover (aₑ) and depth of cut (aₚ)
- Spindle load gauge with warnings for rpm limits, power limits, tool deflection and rubbing chips
- Exact instantaneous cutting force over one revolution (closed-form integral along the helix), with peak force, force slope (N/° and N/ms) and ripple
- **Optimize:** picks stepover, depth, feed per tooth and spindle speed for the highest removal rate inside your limits (peak force, force slope, feed, power, chip thickness, cutting speed, depth). Lock stepover or depth to keep it fixed. Limits save as named profiles.
- Stepover × depth map of removal rate, with the failing limit labelled and smooth-stepdown depths drawn. Click a cell to use it.
- Imperial and metric units
- Saved setups (stored in your browser), plus built-in example cuts
- Works offline as one HTML file, with no build step

## The math

| Quantity | Formula |
|---|---|
| Spindle speed | n = 1000·v<sub>c</sub> / (π·D) |
| Table feed | v<sub>f</sub> = f<sub>z</sub>·n·z |
| Removal rate | Q = a<sub>e</sub>·a<sub>p</sub>·v<sub>f</sub> / 1000 |
| Engagement angle | φ = arccos(1 − 2a<sub>e</sub>/D) |
| Average chip thickness | h<sub>m</sub> = f<sub>z</sub>·sin κ<sub>r</sub>·(2a<sub>e</sub>/D) / φ |
| Specific cutting force | k<sub>c</sub> = k<sub>c1</sub>·h<sub>m</sub><sup>−m<sub>c</sub></sup>·(1 − γ<sub>0</sub>/100) |
| Net power | P<sub>c</sub> = a<sub>e</sub>·a<sub>p</sub>·v<sub>f</sub>·k<sub>c</sub> / (60·10⁶) |
| Torque | M<sub>c</sub> = P<sub>c</sub>·30·10³ / (π·n) |

| Max chip | h<sub>ex</sub> = f<sub>z</sub>·sin φ for a<sub>e</sub> < D/2, else f<sub>z</sub> |
| Force slope (time) | dF/dt = dF/dθ · 6·n / 1000 (N/ms) |
| Flute pitch | πD / (z·tan β) |

Tool deflection (a cantilever with an equivalent diameter of 0.8·D) is an estimate. The force curve is the exact integral of k<sub>c</sub>·f<sub>z</sub>·sin θ along each helical flute.

### How the optimizer works

Force and force slope both scale exactly with f<sub>z</sub><sup>1−m<sub>c</sub></sup>, and per degree of rotation neither depends on rpm. So the optimizer runs the spindle as fast as the rpm and cutting-speed limits allow. Then, on a grid of stepover × depth (log-spaced stepover), it computes one force curve per point and solves for the largest f<sub>z</sub> every limit allows. It refines around the best point and re-checks the final cut on a 0.05° force curve. All of it lives in `optimizer.js`, which the page, the tests and the Fusion add-in share.

**Smooth stepdowns.** Force is nearly constant at depths *just under* a whole multiple of the flute pitch. Go even slightly deeper and the full force slope comes back. So near a multiple, slope is taken as the worst within ±0.05 mm of depth, and the optimizer snaps depth to just under a multiple when that gives at least 99% of the best removal rate.

### Reference cases (checked by `tests/optimizer.test.js`)

Both use 6061 (k<sub>c1</sub> 295, m<sub>c</sub> 0.25), rake 0°, κ<sub>r</sub> 90°.

| | Sandvik check | 3/8" HSM |
|---|---|---|
| D, z, helix | 6 mm, 4, 35° | 9.525 mm, 3, 45° |
| a<sub>e</sub>, a<sub>p</sub> (mm) | 1.67, 15 | 0.5, 10 |
| f<sub>z</sub>, v<sub>c</sub> | 0.0654, 452.39 | 0.06, 612 |
| P<sub>c</sub> | 1.82 kW | 265 W |
| Peak force | 271.9 N | 26.5 N |
| Max force slope | 3.04 N/° | 1.92 N/° |

Run the tests with `node --test tests/optimizer.test.js`.

### Known limitations

- Adaptive toolpaths go above the nominal engagement in corners. The optimizer uses nominal engagement, so leave margin on the force limit.
- Forces are tangential only. The resultant force is typically 1.1–1.3× higher.
- Thicker chips lower k<sub>c</sub>, so with no max chip thickness set, the optimizer tends toward thick chips. Set a max chip thickness for your tool.
- Only wrought aluminium (and cast, by scaling) is calibrated to Sandvik.

### Calibration

Wrought aluminium k<sub>c1</sub> = 295 N/mm² reproduces Sandvik CoroPlus ToolGuide for 6061 (88 HB) with a 6 mm 4-flute tool at aₑ 1.67 mm, aₚ 15 mm, 24,000 rpm and 0.0654 mm/tooth. Sandvik gives 1.82 kW and 0.722 N·m; this calculator gives 1.82 kW and 0.723 N·m. This cut is included as an example setup.

Cast aluminium is scaled by the same factor. The other materials use reference values that have not been checked against Sandvik, so if you have CoroPlus results for them, please open an issue or PR.

## Usage

Open `index.html` in a browser, or use the live page. Select **Custom** to enter your own k<sub>c1</sub> and m<sub>c</sub>.

Results are planning values only. Verify them against your tooling, machine and setup.

## Credits

Forked from [joevenzon/cutcalculator](https://github.com/joevenzon/cutcalculator) (MIT). The tool-deflection and force-curve approach come from that project, and the original calculator is kept as `milling.html`. The cutting formulas follow Sandvik Coromant's published milling formulas.

## License

MIT. See [LICENSE](LICENSE).
