# System Architecture

This document describes the technical architecture and data flow of the Global Soil Salinity Screener.

---

## High-Level Overview

```
┌─────────────────────────────────────────────────────────────────────┐
│                      User Browser (Frontend)                         │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌──────────────────┐  ┌──────────────────┐  ┌──────────────────┐  │
│  │ GitHub Pages     │  │  GEE App Iframe  │  │  Landing Page    │  │
│  │ (Static HTML)    │  │  (Interactive    │  │  Documentation   │  │
│  │                  │  │   Map + Controls)│  │  & Navigation    │  │
│  └──────────────────┘  └──────────────────┘  └──────────────────┘  │
│                               │                                      │
│                               └────────────────────┐                 │
└──────────────────────────────────────────────────────────────────────┘
                                                      │
                         ┌────────────────────────────┘
                         │
        ┌────────────────▼─────────────────────────┐
        │  Google Earth Engine (Backend)            │
        ├───────────────────────────────────────────┤
        │                                           │
        │  ┌──────────────────────────────────────┐ │
        │  │  GEE Code Editor Script              │ │
        │  │  - Read datasets from catalog        │ │
        │  │  - Normalize components (0-1)        │ │
        │  │  - Combine with weights              │ │
        │  │  - Apply terrain suppression         │ │
        │  │  - Generate tiles for map            │ │
        │  └──────────────────────────────────────┘ │
        │                                           │
        │  ┌──────────────────────────────────────┐ │
        │  │  Data Sources                        │ │
        │  │  - SoilGrids 2.0 (250m)             │ │
        │  │  - SRTM GL1 (30m)                   │ │
        │  │  - TerraClimate (~4km)              │ │
        │  │  - JRC GSW (30m)                    │ │
        │  │  - FAO GAUL 2015                    │ │
        │  └──────────────────────────────────────┘ │
        │                                           │
        └───────────────────────────────────────────┘
```

---

## Frontend Architecture

### GitHub Pages Landing Page (`site/index.html`)

**Purpose:** Static HTML landing page hosting the documentation, methodology, and user interface.

**Key Components:**

1. **Header Navigation**
   - Links to methodology, components, data sources, limitations
   - "Open App" button linking to live GEE app

2. **Hero Section**
   - Embedded iframe displaying the live GEE map
   - Fallback message if iframe fails to load

3. **Content Sections**
   - Methodology overview
   - Component descriptions with cards
   - Score interpretation table
   - Data sources table
   - How-to-use guide
   - Limitations & caveats

4. **Footer**
   - Links to GitHub repository
   - Data attribution
   - License and citation information

**Styling:** Custom CSS for responsive design, dark blue / green color scheme, mobile-friendly layout.

**Interactivity:** Minimal client-side JavaScript (only smooth scrolling, navigation highlighting if needed).

### GEE App Iframe

The landing page embeds the GEE app via iframe:

```html
<iframe 
    src="https://washways.projects.earthengine.app/view/globalsoilsalinityscreener"
    style="width: 100%; height: 600px; border: none;">
</iframe>
```

**Why iframe?** Allows users to interact with the full GEE app without leaving the landing page, while keeping the UI clean and documented.

**Considerations:**
- Iframe height is fixed (600px desktop, 400px mobile) — adjust if needed
- GEE app must be publicly shared (no authentication)
- CORS headers are handled automatically by Google's servers

---

## Backend Architecture (Google Earth Engine)

### GEE Script Structure

The main GEE script (`gee/global_soil_salinity_screener.js`) is organized into sections:

#### Section A: Global Setup
- Import country boundaries (FAO GAUL 2015 level 0)
- Define default center, zoom, and parameters
- Initialize UI state variables (current country, weights, layers)

#### Section B: Data Accessors
Functions wrapping each dataset:
- `getSand()` — Load SoilGrids sand at 100-200 cm
- `getClay()` — Load SoilGrids clay at 100-200 cm
- `getSilt()` — Load SoilGrids silt at 100-200 cm
- `getSOC()` — Load SoilGrids SOC at 100-200 cm
- `getSlope()` — Load and compute slope from SRTM
- `getClimateMoistureRatio()` — Load TerraClimate and compute P/PET
- `getPersistentWater()` — Load JRC GSW and filter for persistent occurrence
- `getCountryBoundary()` — Load FAO GAUL for selected country

**Ordering:** Each accessor clips to current country AOI and renames bands for clarity.

#### Section C: Normalization Functions
General-purpose normalizers:
- `normalize(image, min, max)` — Linear normalization to 0-1
- `normalizeLog(image, min, max)` — Log-space normalization (for long-tailed distributions)
- `invertNormalize(image, min, max)` — Invert then normalize (for inverse relationships)

#### Section D: Component Builders
Functions computing each model component:
- `buildFineTexturedSubsoil()` — Combines sand, clay, silt into fines dominance
- `buildWeakDrainage()` — Inverse texture-drainage proxy
- `buildFlatness()` — Inverted slope
- `buildClimaticDryness()` — Inverted moisture ratio
- `buildNearPersistentWater()` — Distance-to-water transform

Each returns a normalized 0-1 image.

#### Section E: Main Model Function
- `buildSalinitySusceptibility(weight1, weight2, ..., weight5)` — Combines all components
  1. Calls each component builder
  2. Multiplies by corresponding weight
  3. Sums weighted components
  4. Applies terrain suppression (slope-based multiplier)
  5. Rescales output to 0-1
  6. Returns final susceptibility image

#### Section F: Visualization Parameters
- Defines color palette (reversed viridis or custom red-blue ramp)
- Sets min/max stretch parameters
- Defines layer names and opacity
- Applies power transform for display (optional)

#### Section G: Layer Management
- Removes old layers from map
- Builds and adds new layers in correct z-order
- Updates layers panel with toggleable components

#### Section H: User Interface
- Country dropdown selector
- Weight sliders (bounded, auto-rebalancing)
- "Reload map" button to trigger recomputation
- "Reset" button to restore defaults
- Status message area
- Legend / color bar

### Data Flow

**Step 1: User Selects Country**
```
User clicks country dropdown → 
  App reads country name → 
  FAO GAUL boundary is fetched → 
  AOI set to country extent
```

**Step 2: User Adjusts Weights (Optional)**
```
User moves weight sliders → 
  Slider values updated locally → 
  Weights rebalanced to sum 100 → 
  Message: "Weights accepted. Click Reload..."
```

**Step 3: User Clicks "Reload Map"**
```
Reload button click → 
  All layers removed from map → 
  buildSalinitySusceptibility() called with current country & weights → 
  GEE computes:
    ├─ getSand(), getClay(), getSilt(), getSOC() → normalized subsoil texture
    ├─ getSlope() → inverted to flatness
    ├─ getClimateMoistureRatio() → inverted to dryness
    ├─ getPersistentWater() → distance transform
    ├─ Weighted combination: 0.32*texture + 0.23*drainage + ... → intermediate 0-1
    ├─ Slope-based terrain suppression: multiply by exp(-0.15*slope)
    ├─ Rescale to 0-1
    └─ Return susceptibility image
  
  Layer added to map with visualization parameters → 
  Map refreshes with new tiles → 
  Status: "Map loaded" (green checkmark)
```

**Step 4: User Clicks on Map Point**
```
Point click → 
  GEE samples susceptibility + all component values at that pixel → 
  Popup generated with:
    ├─ Final susceptibility score (0-1)
    ├─ Component breakdown (texture %, drainage %, flatness %, ...)
    ├─ Raw input values (sand %, clay %, slope°, P/PET, distance-to-water m, ...)
    ├─ Interpretation guidance ("Medium susceptibility → verify locally")
  
  Popup displayed on map → 
  Auto-dismiss after 12 seconds → 
  Cyan crosshair marks sampled location
```

**Step 5: User Toggles Component in Layers Panel**
```
User checks/unchecks layer in top-right Layers panel → 
  Component layer visibility toggled (no recomputation needed) → 
  User can explore model structure without triggering expensive recalculation
```

---

## Computational Model

### Normalization Strategy

All inputs are normalized to 0-1 because they have different numeric ranges:

| Input | Native range | Normalized range | Method |
|---|---|---|---|
| Sand % | 0–100 | 0–1 | Linear: x / 100 |
| Slope ° | 0–90 | 0–1 | Log + linear: log(1+x) / log(1+90) |
| TerraClimate P/PET | 0–10 | 0–1 | Clamp + linear: clamp(x, 0.05, 1.5) then unitScale |
| Distance to water m | 0–100,000+ | 0–1 | Exponential decay or log + linear |

**Why normalize?** Without normalization, a single high-magnitude input (e.g., very deep DTB) would dominate the weighted sum regardless of its true importance.

### Weighting Scheme

```
Susceptibility = 
  0.32 × FineTexture_normalized +
  0.23 × WeakDrainage_normalized +
  0.20 × Flatness_normalized +
  0.17 × ClimaticDryness_normalized +
  0.08 × NearPersistentWater_normalized
```

Default weights sum to 100. If user adjusts weights, they are rebalanced:

```
Adjusted_Weight_i = UserWeight_i / SUM(UserWeights) × 100
```

This ensures total always = 100 and maintains relative emphasis.

### Terrain Suppression

After the weighted sum, a multiplicative penalty is applied based on slope:

```
Penalty = exp(-0.15 × slope_degrees) × exp(-0.10 × HAND_meters)
Final_Score = Weighted_Sum × Penalty
```

**Effect:**
- Flat valley floor (2°, 3m HAND) → Penalty ≈ 0.95 (minimal reduction)
- Gentle slope (8°, 15m) → Penalty ≈ 0.45 (50% reduction)
- Steep slope (25°, 80m) → Penalty ≈ 0.001 (score → ~zero)

**Why multiplicative, not additive?** Because a steep mountain with high texture & drainage scores must still collapse to near-zero (realistic: no groundwater on ridge). An additive penalty would leave a residual score of 2–3 (dangerously misleading).

### Display Stretch

The final 0-1 susceptibility is often displayed with a power transform to improve visualization:

```
Display_Value = Susceptibility ^ 0.4 (example)
```

This compresses mid-range values and reserves the reddest colors for truly high-risk areas.

---

## Scalability & Performance

### Why Country-Scoped Computation?

The app clips all rasters to national boundaries to:

1. **Reduce memory:** Processing 250 million pixels (global) vs. 10 million (country) saves resources
2. **Improve responsiveness:** Smaller AOI = faster tile generation
3. **Allow interactivity:** Manual reload button gives user control over when recomputation happens
4. **Avoid timeouts:** Large processing jobs can timeout; country-scale is manageable

### Typical Compute Times

| Country Size | Area (km²) | Typical Runtime | Notes |
|---|---:|---:|---|
| Luxembourg | ~2,600 | 5–10 sec | Very fast |
| Malawi | ~118,500 | 20–40 sec | Recommended |
| South Africa | ~1.2 million | 60–120 sec | Larger; plan for patience |
| USA | ~10 million | Timeout risk | Try smaller state instead |

**Scaling factor:** Roughly linear with pixel count.

### Optimization Strategies

1. **Aggregation:** All inputs coarsened to 250 m before combination
2. **Lazy evaluation:** GEE defers computation until tiles are needed
3. **Country clipping:** FAO GAUL mask reduces pixel count
4. **Manual reload:** Avoids recomputation on every slider move

---

## Data Flow Diagram

```
┌──────────────────┐
│ User Actions     │
│ (country select, │
│  weight adjust,  │
│  map click)      │
└────────┬─────────┘
         │
         ▼
┌──────────────────┐     ┌──────────────────────┐
│ GEE App          │────▶│ FAO GAUL 2015        │ (country boundary)
│ UI State         │     └──────────────────────┘
└────────┬─────────┘
         │
         ▼
┌──────────────────────────────────────────────────┐
│ buildSalinitySusceptibility()                    │
└────────┬─────────────────────────────────────────┘
         │
         ├─▶ getSand/Clay/Silt/SOC ──▶ SoilGrids 2.0
         │   ├─ Fine-textured subsoil (32%)
         │   └─ Weak drainage (23%)
         │
         ├─▶ getSlope ──────────────▶ SRTM GL1
         │   └─ Flatness (20%)
         │
         ├─▶ getClimateMoistureRatio ──▶ TerraClimate
         │   └─ Climatic dryness (17%)
         │
         ├─▶ getPersistentWater ────▶ JRC GSW v1.4
         │   └─ Near persistent water (8%)
         │
         ├─ Normalize each 0–1
         ├─ Weighted sum (0.32 + 0.23 + 0.20 + 0.17 + 0.08 = 1.0)
         ├─ Terrain suppression (multiply by slope penalty)
         ├─ Rescale 0–1
         │
         ▼
┌──────────────────────────────────────────────────┐
│ Susceptibility Image (0–1)                       │
└────────┬─────────────────────────────────────────┘
         │
         ├─ Display with power transform & color palette
         ├─ Add component layers (toggleable)
         ├─ Add country boundary layer
         │
         ▼
┌──────────────────────────────────────────────────┐
│ GEE Map Tiles                                    │
└──────────────────────────────────────────────────┘
```

---

## Error Handling

### Dataset Unavailability

If a dataset is removed from GEE catalog or becomes inaccessible:

1. Script will fail with error message: `"ImageCollection not found"`
2. Recommend checking:
   - Earth Engine dataset documentation
   - GEE community forum for migration announcements
   - Dataset license or data provider for discontinuation

**Mitigation:** Maintain a backup list of alternative datasets (e.g., alternative soil texture sources, alternative elevation models).

### Country Not Available

If a user selects a country not in FAO GAUL:

1. App should display message: "Country not in database"
2. Fall back to global view

**Mitigation:** Ensure FAO GAUL layer is correctly loaded; test on a known country first.

### Large Country Timeout

If a large country (USA, Russia, Canada) times out:

1. Message: "Computation exceeded time limit. Try a smaller country or zoom to a specific region."
2. User can reduce resolution setting (trade detail for speed)

**Mitigation:** Add warnings for countries > 5 deg² and suggest zooming to state/province.

---

## Future Architecture Improvements

1. **Pre-computed assets:** Export national-level salinity maps at 90 m resolution as GEE assets (avoids interactive compute)
2. **Caching:** Cache component layers for frequently accessed countries
3. **Regional weight presets:** Automatically apply region-specific weights based on user location
4. **Machine learning calibration:** Ingest field EC data and refit weights
5. **Mobile app:** React Native / Flutter wrapper for smartphone deployment

---

## Technical Stack Summary

| Layer | Technology |
|---|---|
| User frontend | HTML5, CSS3, JavaScript (vanilla) |
| Landing page hosting | GitHub Pages (static) |
| Interactive app | Google Earth Engine Apps platform |
| Backend computation | Google Earth Engine API (server-side) |
| Data sources | Public GEE catalog (SoilGrids, SRTM, TerraClimate, JRC, FAO) |
| Version control | Git / GitHub |
| Documentation | Markdown |

---

**Questions on architecture?** See [CONTRIBUTING.md](../CONTRIBUTING.md) or open an issue on GitHub.
