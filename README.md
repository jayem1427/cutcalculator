# djr_cutcalculator

A single-page milling speeds, feeds and cutting-load calculator for side milling, slotting and high-speed (dynamic) toolpaths. It uses Sandvik Coromant's specific-cutting-force model, with wrought aluminium calibrated against Sandvik CoroPlus ToolGuide.

**Live:** https://jayem1427.github.io/djr_cutcalculator/

## Features

- Spindle speed, feed rate, material removal rate, net and motor power, spindle torque and cutting force
- Exact average chip thickness for any radial engagement, plus a chip-thinning factor
- To-scale top view and cross-section showing stepover (aₑ) and depth of cut (aₚ)
- Spindle load gauge with warnings for rpm limits, power limits, tool deflection and rubbing chips
- Instantaneous cutting force over one revolution, including helix angle
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

Tool deflection (a cantilever with an equivalent diameter of 0.8·D) and the force-per-revolution curve are estimates.

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
