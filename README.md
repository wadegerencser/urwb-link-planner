# 📡 URWB Backhaul Link Planner

> **CX US Public Sector Automation Hub**
> Repository: `sac-mgerencs-urwb-link-planner`
> Owner: Wade Gerencser (`mgerencs`) · CX US Public Sector
> Template base: [`catalystcenter-as-code-template`](https://wwwin-github.cisco.com/cx-usps-auto/catalystcenter-as-code-template)

---

## Overview

URWB Backhaul Link Planner is a **Services as Code (SaC) wireless planning tool** for Cisco Ultra-Reliable Wireless Backhaul deployments. It gives field engineers and wireless architects an interactive 3D simulator to validate point-to-point link geometry, Fresnel zone clearance, and link budget before deploying Cisco IW9165 or IW9167 radios — and before writing a single line of NaC configuration.

The tool runs entirely in the browser — no Python, no npm, no server. It is a single self-contained HTML file. p5.js loads from a CDN on first use.

**Live:** https://wadegerencser.github.io/urwb-link-planner/ — nothing to install.

---

## How It Fits the NaC Workflow

```
┌─────────────────────────────────────────────────────────────────┐
│  1. PLAN    →  Open URWB Link Planner in your browser            │
│               Drag poles · set heights · confirm Fresnel clear   │
│               Validate link budget before committing to hardware  │
├─────────────────────────────────────────────────────────────────┤
│  2. DEFINE  →  Edit YAML files in data/                          │
│               sites.yaml · wireless_rf_profiles.yaml             │
│               urwb_link_profiles.yaml                            │
├─────────────────────────────────────────────────────────────────┤
│  3. DEPLOY  →  terraform plan && terraform apply                 │
│               Terraform module pushes URWB config to             │
│               Cisco Catalyst Center / IW management plane        │
├─────────────────────────────────────────────────────────────────┤
│  4. VERIFY  →  Re-open the planner post-install with real        │
│               measured pole heights and site distance            │
│               Confirm margin and Fresnel clearance match design  │
└─────────────────────────────────────────────────────────────────┘
```

The planner handles step 1. Steps 2–3 are standard NaC — you define desired state in YAML and Terraform handles all Catalyst Center API calls, ensuring every URWB deployment is consistent and repeatable.

---

## Installation — Every Platform

### Browser — Nothing to Install (Recommended)

Open **https://wadegerencser.github.io/urwb-link-planner/** in any modern browser. No install, no login.

### macOS — One-Liner Download
```bash
curl -LO https://raw.githubusercontent.com/wadegerencser/urwb-link-planner/main/index.html && open index.html
```

### Linux — One-Liner Download
```bash
curl -LO https://raw.githubusercontent.com/wadegerencser/urwb-link-planner/main/index.html && xdg-open index.html
```

### Windows — One-Liner Download (PowerShell)
```powershell
curl -LO https://raw.githubusercontent.com/wadegerencser/urwb-link-planner/main/index.html; start index.html
```

### Clone (To Modify or Host Internally)
```bash
# Public GitHub
git clone https://github.com/wadegerencser/urwb-link-planner.git

# Cisco Internal
git clone https://wwwin-github.cisco.com/cx-usps-auto/sac-mgerencs-urwb-link-planner.git

cd urwb-link-planner
open index.html     # macOS
xdg-open index.html # Linux
start index.html    # Windows
```

No pip, no npm, no build step — the browser is the runtime.

---

## What It Does

- **3D scene** — Two poles with Cisco IW-series APs, rendered with antenna radiation lobes reconstructed from published azimuth × elevation beamwidths, aimed at each other across a procedural urban environment
- **Fresnel zone** — First Fresnel zone (F1) ellipsoid drawn along the line of sight; 60% clearance core shown as rings; zone highlights red when an obstacle violates it
- **Draggable geometry** — Raise/lower each pole (drag the blue top handle), pull the sites apart (drag the orange base handle), or use the panel sliders; obstacle can be dragged along the path and resized
- **Live link budget** — EIRP, free-space path loss, RSSI, fade margin, F1 radius, worst-case Fresnel clearance, and a green/amber/red link verdict updated every frame
- **2D side-view profile** — Compact link profile chart in the panel showing poles, LOS, Fresnel ellipse, and obstacle
- **AP picker** — IW9165D (internal 15 dBi directional), IW9165E, or IW9167E with selectable external antenna (omni, patch, sector, dish)
- **Band selection** — 5 GHz / 6 GHz affects both FSPL and Fresnel radius
- **PNG export** — Press **S** to save a snapshot for documentation

---

## Controls

| Action | Control |
|---|---|
| Orbit camera | Drag empty scene |
| Zoom | Scroll wheel |
| Change pole height | Drag blue sphere at pole top |
| Change link distance | Drag orange sphere at pole base |
| Move / resize obstacle | Drag green sphere on obstacle |
| Save PNG | **S** |

---

## AP and Antenna Catalog

| Model | Antenna | Gain | Beamwidth | Notes |
|---|---|---|---|---|
| IW9165D | Internal | 15 dBi | 30° × 30° | Purpose-built P2P/P2MP backhaul |
| IW9165E | External (N-type) | User-selected | User-selected | 2× N-type ports |
| IW9167E | External (N-type) | User-selected | User-selected | Tri-band, URWB mode |
| — | 8 dBi omni | 8 dBi | 360° × 16° | |
| — | 13 dBi patch | 13 dBi | 35° × 35° | |
| — | 17 dBi sector | 17 dBi | 90° × 8° | |
| — | 19 dBi dish | 19 dBi | 18° × 18° | |

---

## Link Budget Model

```
EIRP     =  Tx power (dBm)  +  Antenna gain (dBi)
FSPL     =  32.44  +  20·log₁₀(d_km)  +  20·log₁₀(f_MHz)   [dB]
RSSI     =  EIRP (A)  +  Antenna gain (B)  −  FSPL           [symmetric link]
Margin   =  RSSI  −  Rx sensitivity (−92 dBm baseline)
```

### Status Thresholds

| Status | Condition |
|---|---|
| **LINK OK** | Margin ≥ 10 dB and 100% F1 clear |
| **USABLE** | Margin ≥ 10 dB, F1 partially obstructed |
| **MARGINAL** | Margin 6–10 dB |
| **DEGRADED** | Fresnel clearance < 60% |
| **BLOCKED** | Obstacle in line of sight |
| **LINK DOWN** | Margin < 6 dB |

### URWB Rules of Thumb

- Keep **100% of F1 clear** — URWB is designed for high-reliability links; do not accept partial clearance as a design target
- **60% F1** is the absolute minimum before measurable diffraction loss begins
- **Fade margin target:** 10+ dB for reliable backhaul; **20+ dB for critical links** (first responder, utility SCADA, transit)
- Vertical scale is **exaggerated 4× for visibility**; all computed values use true geometry

---

## Model Limitations

- Propagation is **free-space path loss only** — no rain fade, multipath, diffraction, or ducting
- Radiation lobes are **separable reconstructions** from published beamwidths, not full 3D measured antenna patterns
- Rx sensitivity is fixed at **−92 dBm** (approximately lowest-MCS). Real thresholds vary by modulation, channel width, and chain count
- Regulatory EIRP ceilings (LPI / Standard Power / AFC grants) are **not enforced** — EIRP readout is Tx + antenna gain; the deployer is responsible for regulatory compliance

Treat results as first-order planning inputs, not a replacement for a full RF site survey or propagation simulation.

---

## Repository Structure

```
sac-mgerencs-urwb-link-planner/
│
├── index.html     # Complete application — browser is the runtime
└── README.md
```

This tool is intentionally zero-dependency. Everything required to run it ships in one file.

---

## Related NaC Resources

| Resource | Link |
|---|---|
| catalystcenter-as-code-template | [cx-usps-auto/catalystcenter-as-code-template](https://wwwin-github.cisco.com/cx-usps-auto/catalystcenter-as-code-template) |
| nac-catalystcenter (source) | [cx-usps-auto/nac-catalystcenter](https://wwwin-github.cisco.com/cx-usps-auto/nac-catalystcenter) |
| Beacon Finder | [cx-usps-auto/sac-mgerencs-beacon-finder](https://wwwin-github.cisco.com/cx-usps-auto/sac-mgerencs-beacon-finder) |
| WiFiSizer | [cx-usps-auto/sac-mgerencs-wifi-density-calculator](https://wwwin-github.cisco.com/cx-usps-auto/sac-mgerencs-wifi-density-calculator) |
| RF Math Toolkit | [wadegerencser/rf-math-toolkit](https://github.com/wadegerencser/rf-math-toolkit) |
| Cisco URWB Design Guide | [cisco.com/c/en/us/support/wireless/ultra-reliable-wireless-backhaul](https://www.cisco.com/c/en/us/support/wireless/ultra-reliable-wireless-backhaul/series.html) |
| NaC Documentation | [cx-usps-auto org](https://wwwin-github.cisco.com/cx-usps-auto) |
| SaC Enablement Launchpad | CX US PS SharePoint |

---

## Security & Compliance

- This repository contains no customer data, credentials, or network configuration
- The tool runs entirely client-side in the browser — no data is transmitted to any server
- Do not commit customer site coordinates, link survey data, or CUI
- For CUI or classified data use the [US PS GitLab](https://sscm.gus.cisco.com/)

---

## License

MIT

---

## Author

**Wade Gerencser** (`mgerencs`)
Network Engineer / Wireless Architect — CX US Public Sector
Cisco CX · [`cx-usps-auto`](https://wwwin-github.cisco.com/cx-usps-auto)
