# URWB Backhaul Link Planner

Interactive 3D point-to-point wireless backhaul simulator for Cisco industrial
access points (IW9165D / IW9165E / IW9167E, URWB-style deployments).

Open `index.html` in any modern browser. No build step, no server, no
dependencies beyond the p5.js CDN.

## What it does

- Two poles with APs, rendered in 3D with their antenna radiation lobes
  (reconstructed from azimuth x elevation beamwidths) aimed at each other
- First Fresnel zone (F1) ellipsoid drawn along the line of sight, with the
  60% clearance core shown as rings
- Draggable geometry: raise/lower each pole (drag the blue top handles),
  pull the sites apart (drag the orange base handles), or use the sliders
- Obstacle you can slide along the path and grow in height; the Fresnel
  zone highlights red where it is being violated
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
