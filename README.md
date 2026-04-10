# 🌍 Global Soil Salinity Screener

A satellite-powered screening tool for mapping groundwater salinity susceptibility in data-scarce regions.

| [Live App](https://washways.projects.earthengine.app/view/globalsoilsalinityscreener) | [Methodology](./METHODOLOGY.md) | [GitHub](https://github.com/washways/GlobalSoilSalinityScreener) | [License: MIT](./LICENSE) |
|---|---|---|---|

The Global Soil Salinity Screener identifies places where the **environmental setting is more favourable to salinity accumulation or persistence**, not to measure salinity directly. It is therefore a **susceptibility surface**, not an electrical conductivity map or regulatory classification. The tool runs entirely on [Google Earth Engine](https://earthengine.google.com/).

## 📑 Table of Contents

- [Quick Start](#-quick-start)
- [Why This Tool Exists](#-why-this-tool-exists)
- [How It Works](#-how-it-works)
- [The Five Components](#-the-five-components)
- [Score Interpretation](#-score-interpretation)
- [Data Sources](#-data-sources)
- [How to Use the App](#-how-to-use-the-app)
- [Repository Structure](#-repository-structure)
- [Deployment](#-deployment)
- [Limitations & Caveats](#-limitations--caveats)
- [Contributing](#-contributing)
- [Citation](#-citation)
- [License](#-license)

---

## 🚀 Quick Start

1. **Open the app** → https://washways.projects.earthengine.app/view/globalsoilsalinityscreener
2. **Select a country** from the dropdown or click on the map
3. **Adjust weights** (optional) using the sliders — each represents a component of the model
4. **Click "Reload map"** to apply changes
5. **Explore layers** using the Layers panel (top right) to understand what drives the score
6. **Click any point** for detailed diagnostics at that location

---

## 🎯 Why This Tool Exists

Groundwater salinity is an emerging challenge in arid and semi-arid regions worldwide, threatening drinking water access and agricultural productivity. Traditional salinity mapping relies on expensive field campaigns or chemical surveys. This tool provides a **first-pass screening layer** using freely available satellite data to identify where salinity is more likely to occur.

The design is grounded in current salinity literature, which consistently shows that salinity patterns are typically driven by a **combination of soil texture, drainage, topographic setting, climatic dryness, hydrological position, and groundwater or surface-water interactions**, rather than by any one variable alone.

### Key Design Principles

1. **Multi-factor susceptibility**: Based on peer-reviewed hydropedological and salinity literature
2. **Transparent**: Every component is visible as a toggleable map layer
3. **Open data**: Uses only public satellite datasets (SoilGrids, SRTM, TerraClimate, JRC)
4. **Adjustable**: Users can tune component weights to match local knowledge
5. **Heuristic, not calibrated**: Honest about its limitations; designed for screening, not prediction

---

## 🧠 How It Works

The model assumes that salinity is more plausible where several broad environmental conditions coincide:

```
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│  Fine-textured subsoil (32%)  ────┐                         │
│  "Fine soils resist percolation"   │                        │
│                                    │   Weighted   Terrain  │
│  Weak drainage (23%)  ─────────────┤   Sum  ──→  Suppress ─┼→ Score
│  "Poor flushing = salt persistence"│   (0–1)     (0–1)      │
│                                    │                        │
│  Flatness (20%)  ─────────────────┤                         │
│  "Flat terrain ↑ salt accumulation"│                        │
│                                    │                        │
│  Climatic dryness (17%)  ──────────┤                         │
│  "Dry climate ↑ evaporative salts" │                        │
│                                    │                        │
│  Near persistent water (8%)  ──────┘                        │
│  "Lowlands & wetlands ↑ salt zones"                         │
└─────────────────────────────────────────────────────────────┘
```

Each component is independently normalized to 0–1, weighted according to literature-informed defaults (summing to 100%), combined, and then terrain-suppressed using slope to reduce false positives in steep terrain.

---

## 🔬 The Five Components

### 1. Fine-textured Subsoil (32%)

**Question**: Does the subsoil contain fine materials that resist water movement?

- **Data**: SoilGrids 2.0 sand, clay, silt, SOC at 100–200 cm depth (250 m)
- **Rationale**: Fine-textured materials slow percolation, increase water retention, and strengthen capillary rise—all of which favour salt storage or upward salt movement under dry conditions.
- **Formula**: Fines-dominance logic on subsoil texture

**Why 100–200 cm?** Because salinity persistence and drainage behaviour are often controlled by subsurface material properties, not just the uppermost topsoil.

### 2. Weak Drainage (23%)

**Question**: Is the subsoil poorly drained?

- **Data**: Inverse of texture-drainage proxy (SoilGrids)
- **Rationale**: Poor drainage reduces flushing and promotes salt persistence. In salinity mapping and salinization risk studies, drainage is consistently identified as a central control.
- **Formula**: Coarser subsoils treated as freely draining; finer subsoils as poorly drained

**Limitation**: Actual drainage also depends on structure, groundwater depth, restrictive layers, and land use. This is a proxy, not a complete assessment.

### 3. Flatness (20%)

**Question**: Is the terrain flat (low-relief)?

- **Data**: SRTM GL1 slope (~30 m)
- **Rationale**: Flat terrain is generally more favourable to salt accumulation than steep slopes because flatter areas are more likely to behave as depositional, poorly drained, or evaporative settings.
- **Formula**: Slope inverted (steeper slope → lower flatness score)

### 4. Climatic Dryness (17%)

**Question**: Is the climate dry enough to concentrate salts rather than leach them?

- **Data**: TerraClimate moisture ratio (precipitation / PET) (~4 km, monthly, 1958–present)
- **Rationale**: Salts accumulate more readily where evaporative concentration exceeds effective leaching. Long-term aridity is a practical approximation of this balance.
- **Formula**: Moisture ratio inverted (lower precipitation/PET → higher dryness score)

### 5. Near Persistent Water (8%)

**Question**: Is the area near persistent surface water (indicating lowland/hydrological connection)?

- **Data**: JRC Global Surface Water v1.4 (Landsat-based, 1984–2021, 30 m)
- **Rationale**: Nearness to persistent water can indicate lowland, floodplain, valley-bottom, endorheic, or hydrologically connected settings where salinity processes are plausible.
- **Formula**: Distance-to-water transform; closer → higher score

**Caveat**: This relationship is context-dependent and therefore weighted lowest (8%). Over-emphasis can generate false positives.

---

## 📊 Score Interpretation

The output is normalized to 0–1 (before visualization). On the map, higher values appear progressively redder, with red reserved for the highest-susceptibility areas.

**What does the score mean?** It indicates areas where several broad environmental conditions *favour* salt accumulation—not places where salinity *definitely* exists. High susceptibility is a hypothesis, not a guarantee.

| Score | Interpretation | Rationale |
|---|---|---|
| 0.0–0.2 | Very Low | Low combined evidence. De-prioritise unless strong local knowledge suggests otherwise. |
| 0.2–0.4 | Low | Some signal, but weak. Cross-reference with field EC or borehole data. |
| 0.4–0.6 | Medium | Promising. Identify which component(s) drive the score and verify locally. |
| 0.6–0.8 | High | Strong screening signal. Good candidate for targeted survey or testing. |
| 0.8–1.0 | Exceptional | Top-tier signal. Merge with field salinity observations for validation. |

---

## 📡 Data Sources

| Input | Source | Native Resolution | Role in Model |
|---|---|---:|---|
| Sand, clay, silt, SOC at 100–200 cm | SoilGrids 2.0 (ISRIC, 2021) | 250 m | Represents subsoil texture and carbon conditions |
| Elevation and slope | SRTM GL1 v003 (USGS, 2000) | ~30 m | Used to derive slope and terrain suppression |
| Persistent surface water occurrence | JRC Global Surface Water v1.4 (1984–2021) | 30 m | Used to estimate distance to persistent water |
| Climate moisture ratio | TerraClimate (1958–present, updated annually) | ~4 km, monthly | Long-term dryness signal via P/PET |
| Country boundaries | FAO GAUL 2015 level 0 | Vector | Used only to clip computation to selected country |

For detailed dataset documentation, see [docs/datasets.md](./docs/datasets.md).

---

## 🗺️ How to Use the App

1. **Zoom in** to your area of interest (zoom level ≥ 9 recommended for district/catchment scale)
2. **Select a country** from the dropdown on the left, or click on the map
3. **(Optional) Adjust weights** using the slider controls:
   - Minimum and maximum bounds are set to keep weights plausible
   - Total always sums to 100 by automatic rebalancing
4. **(Optional) Review component layers** by toggling them in the Layers panel (top right)
5. **Click "Reload map"** to apply weight changes and recompute
6. **Click any location** for a detailed diagnostic popup showing:
   - The susceptibility score at that point
   - Component contributions
   - Raw input values (texture, slope, moisture, etc.)
7. **(Optional) Enable Auto-stretch** for better colour contrast at local zoom levels

**Why the manual "Reload" button?** In Google Earth Engine, re-running a country-scale raster model on every slider movement creates excessive recomputation and poor user experience. A manual reload step improves responsiveness.

---

## 📁 Repository Structure

```
GlobalSoilSalinityScreener/
├── README.md                         ← You are here
├── METHODOLOGY.md                    ← Full scientific methodology
├── CONTRIBUTING.md                   ← How to contribute
├── CHANGELOG.md                      ← Version history
├── LICENSE                           ← MIT License
├── .gitignore
│
├── gee/
│   └── global_soil_salinity_screener.js    ← The complete GEE script
│
├── docs/
│   ├── architecture.md               ← System architecture & data flow
│   ├── datasets.md                   ← Complete dataset catalog
│   ├── deployment.md                 ← GEE App + GitHub Pages deployment
│   └── component-weights.md          ← Rationale for default weighting
│
├── .github/
│   └── workflows/
│       └── deploy.yml                ← GitHub Pages deployment workflow
│
└── site/
    ├── index.html                    ← Landing page (embeds GEE app iframe)
    ├── style.css                     ← Custom styling
    └── script.js                     ← Client-side interactivity (optional)
```

---

## 🚀 Deployment

### Google Earth Engine App

The app is published via the GEE Apps platform at:
```
https://washways.projects.earthengine.app/view/globalsoilsalinityscreener
```

See [docs/deployment.md](./docs/deployment.md) for step-by-step instructions on publishing a GEE app.

### GitHub Pages (Landing Page)

The `site/index.html` landing page is hosted via GitHub Pages and embeds the GEE app in an iframe. Users can interact with the full tool directly from the GitHub Pages site.

**Setup:**
1. Enable GitHub Pages in repository settings
2. Set "Source" to `Deploy from a branch`
3. Select `main` branch and `/root` directory
4. Push changes to trigger automatic deployment

---

##  ⚠️ Limitations & Caveats

| Limitation | Implication |
|---|---|
| **Screening tool only** | Not a substitute for field EC survey, hydrogeological investigation, or regulatory compliance mapping |
| **Resolution ceiling** | Limited by coarsest input (SoilGrids & DTM at 250 m / 30 m). Cannot resolve meter-scale soil variation |
| **Texture-only drainage proxy** | Actual drainage also depends on structure, groundwater depth, restrictive layers, and land use |
| **Static climate proxy** | TerraClimate moisture ratio is a 60+ year average, not year-specific recharge or current drought |
| **Heuristic weighting** | Default weights (32 / 23 / 20 / 17 / 8) are literature-informed but not statistically calibrated against field EC data |
| **No field calibration** | Not validated against measured groundwater salinity, electrical conductivity, or sodium adsorption ratio |
| **Context-dependence** | Relationships between predictors and salinity vary by aquifer type, irrigation intensity, and climatic regime. Results should always be interpreted locally. |
| **No water quality component** | High susceptibility ≠ hazardous salinity. Field salinity observations and water chemistry are essential. |

**Bottom line:** This is a *hypothesis-generating tool*. Use it to identify zones worthy of further investigation, but always ground decisions in field data and local expertise.

---

## 🤝 Contributing

Contributions are welcome! Priority areas:

- **Field calibration data**: Measured groundwater EC, SAR, or TDS to validate the susceptibility model
- **Regional adaptation**: Ground-truthing in new geographic areas; optimised weight presets for specific regions
- **New data sources**: Finer-resolution soil maps, aquifer lithology, borehole density, or irrigation extent
- **Documentation**: Case studies, regional case studies, or translations
- **UX improvements**: Better visualisations, accessibility enhancements, or mobile responsiveness

See [CONTRIBUTING.md](./CONTRIBUTING.md) for guidelines.

---

## 📖 Citation

If you use this tool in your work, please cite:

```
WASHways (2026). Global Soil Salinity Screener: A satellite-derived
susceptibility screening tool for groundwater salinity mapping.
https://github.com/washways/GlobalSoilSalinityScreener
```

---

## 📄 License

This project is licensed under the [MIT License](./LICENSE).

The input datasets have their own licenses — see [docs/datasets.md](./docs/datasets.md) for details.

---

**Built by [WASHways](https://washways.org/)** — Improving water access through open science and technology.

This is a screening tool — always verify results with field investigation.
