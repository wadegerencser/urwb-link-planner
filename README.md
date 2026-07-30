# URWB Backhaul Link Planner

Interactive 3D point-to-point wireless backhaul simulator for Cisco industrial
access points (IW9165D / IW9165E / IW9167E, URWB-style deployments).

## How to run

No install, no server, no build step — the browser is the runtime.

**1. Zero-install (recommended):** open the live version:
https://wadegerencser.github.io/urwb-link-planner/

**2. One-shot download:**

```bash
curl -LO https://raw.githubusercontent.com/wadegerencser/urwb-link-planner/main/index.html
open index.html        # macOS (Windows: start index.html, Linux: xdg-open index.html)
```

**3. Clone (for hacking on it):**

```bash
git clone https://github.com/wadegerencser/urwb-link-planner
```

Then open `index.html`. Note: p5.js loads from a CDN, so the first load
needs internet access.

## What it does

- Two poles with APs, rendered in 3D with their antenna radiation lobes
  (reconstructed from azimuth x elevation beamwidths) aimed at each other
- First Fresnel zone (F1) ellipsoid drawn along the line of sight, with the
  60% clearance core shown as rings
- Draggable geometry: raise/lower each pole (drag the blue top handles),
  pull the sites apart (drag the orange base handles), or use the sliders
- Obstacle types (tree, small building, generic block) you can slide along
  the path, grow in height, or drag directly in the scene; the Fresnel zone
  highlights red where it is being violated
- Auto-scaled distance markers and labeled meter ticks along the ground so
  dragging the sites apart reads as real distance
- Live link budget: EIRP, free-space path loss, RSSI, fade margin, F1 radius,
  and worst-case Fresnel clearance, with an OK / DEGRADED / BLOCKED verdict
- AP picker: IW9165D internal 15 dBi directional, or IW9165E / IW9167E with
  a selectable external antenna (omni, patch, sector, dish)
- 5 GHz / 6 GHz band selection (affects both FSPL and Fresnel radius)

## Controls

- Drag empty scene: orbit camera
- Scroll: zoom
- Drag blue sphere at pole top: change that pole's height
- Drag orange sphere at pole base: change link distance
- Drag green sphere on the obstacle: move it along the path / change its height
- S: save a PNG snapshot

## Model notes and limitations

- Vertical scale is exaggerated 4x for visibility; all computed values use
  true geometry
- Propagation is free-space path loss only: no rain fade, multipath,
  diffraction, or ducting. Treat results as first-order planning, not a
  replacement for a real site survey
- Radiation lobes are separable reconstructions (azimuth cut x elevation
  cut) from published beamwidths, not full 3D measured patterns
- Rx sensitivity is fixed at -92 dBm (roughly lowest-MCS). Real thresholds
  vary by rate, width, and chain count
- Regulatory EIRP ceilings (LPI / Standard Power / AFC grants) are NOT
  enforced by the tool; the EIRP readout is Tx + antenna gain

## Rules of thumb encoded in the status logic

- Keep 100% of F1 clear; 60% is the minimum before diffraction loss begins
- Fade margin: 10+ dB for reliable backhaul, 20 dB for critical links

## License

MIT
