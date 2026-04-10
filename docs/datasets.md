# Datasets

This document provides a complete catalog of all input datasets used in the Global Soil Salinity Screener, including access links, licenses, and usage notes.

## 1. SoilGrids 2.0

**Purpose:** Soil texture (sand, clay, silt) and soil organic carbon (SOC) at standard depths
**Source:** ISRIC — World Soil Information
**Version:** SoilGrids 2.0 (released 2021)
**Resolution:** 250 m
**Geographic coverage:** Global (land only)
**In Earth Engine catalog:** `projects/soilgrids-isric/*`

**Properties used:**
- `sand_0-5cm`, `sand_5-15cm`, `sand_15-30cm`, `sand_30-60cm`, `sand_60-100cm`, `sand_100-200cm`
- `clay_0-5cm`, `clay_5-15cm`, `clay_15-30cm`, `clay_30-60cm`, `clay_60-100cm`, `clay_100-200cm`
- `silt_0-5cm`, `silt_5-15cm`, `silt_15-30cm`, `silt_30-60cm`, `silt_60-100cm`, `silt_100-200cm`
- `organic_carbon_0-5cm`, `organic_carbon_5-15cm`, `organic_carbon_15-30cm`, `organic_carbon_30-60cm`, `organic_carbon_60-100cm`, `organic_carbon_100-200cm`

**Why 100–200 cm?** Salinity persistence and drainage behaviour in the subsoil (saprolite or C horizon) are often more important than topsoil properties.

**License:** [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/)

**Citation:**
```
Poggio, L., A. De Sousa, M. J. I. Batjes, G. Heuvelink, R. Kempen, B. Lifrüz, and F. Saby. 
(2021). SoilGrids 2.0: producing soil information for the globe. SOIL, 7, 291–314.
https://doi.org/10.5194/soil-7-291-2021
```

**Documentation:** https://docs.isric.org/globaldata/soilgrids/

---

## 2. SRTM GL1 v003

**Purpose:** Digital elevation model (DEM) for terrain and slope analysis
**Source:** USGS (United States Geological Survey) / NASA
**Version:** v003
**Acquisition date:** February 2000 (mostly)
**Resolution:** ~30 m
**In Earth Engine catalog:** `USGS/SRTMGL1_003`

**Band used:**
- `elevation` — elevation in meters above sea level

**Post-processing in app:**
- Slope is computed via local gradient operators
- Slope is then inverted for the Flatness component

**License:** Public domain (USGS)

**Citation:**
```
NASA/METI/AIST/Japan Spacesystems, and U.S./Japan ASTER Science Team (2019).
ASTER Global Digital Elevation Model V003. NASA EOSDIS Land Processes DAAC.
Accessed YYYY-MM-DD from https://doi.org/10.5067/ASTER_GDEM.003
```

**Documentation:** https://developers.google.com/earth-engine/datasets/catalog/USGS_SRTMGL1_003

---

## 3. TerraClimate

**Purpose:** Long-term climate water-balance data (precipitation, evapotranspiration)
**Source:** University of California, Merced (Climatology Lab)
**Version:** Monthly data (1958–present, updated annually)
**Resolution:** ~4 km (4638 m)
**In Earth Engine catalog:** `IDAHO_EPSCOR/TERRACLIMATE`

**Bands used:**
- `pr` — monthly precipitation (mm)
- `pet` — monthly potential evapotranspiration (mm)

**Computation in app:**
- Moisture ratio = annual P / annual PET (averaged over full dataset period)
- Ratio is then normalized and inverted (lower ratio = higher score = drier)

**License:** [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/)

**Citation:**
```
Abatzoglou, J. T., S. Z. Dobrowski, S. A. Parks, and K. C. Hegewisch, 2018.
TerraClimate, a high-resolution global dataset of monthly climate and climatic water balance from 1958–2015.
Scientific Data 5, 170191. https://doi.org/10.1038/sdata.2017.191
```

**Documentation:** https://www.climatologylab.org/terraclimate.html

---

## 4. JRC Global Surface Water v1.4

**Purpose:** Persistent surface water occurrence (lakes, rivers, reservoirs)
**Source:** Joint Research Centre of the European Commission
**Version:** v1.4 (based on Landsat observations, 1984–2021)
**Resolution:** 30 m
**In Earth Engine catalog:** `JRC_GSW1_4_GlobalSurfaceWater`

**Bands used:**
- `occurrence` — annual occurrence of surface water (0–100 %, pixel-wise)

**Computation in app:**
- Occurrence layer thresholded to identify persistent water pixels (typically ≥ 50% occurrence)
- Distance-to-water transform (Euclidean distance) identifies nearness to persistent features
- Distance normalized and inverted (closer = higher score)

**License:** [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/)

**Citation:**
```
Pekel, J.-F., A. Cottam, N. Gorelick, and A. S. Belward, 2016.
High-resolution mapping of global surface water and its long-term changes.
Nature 540, 418–422. https://doi.org/10.1038/nature20584
```

**Documentation:** https://developers.google.com/earth-engine/datasets/catalog/JRC_GSW1_4_GlobalSurfaceWater

---

## 5. FAO GAUL 2015 Level 0

**Purpose:** Global administrative boundaries (country-level)
**Source:** FAO (Food and Agriculture Organization of the United Nations)
**Version:** GAUL 2015 level 0 (country level only)
**Resolution:** Vector (polygon)
**In Earth Engine catalog:** `FAO/GAUL/2015/level0`

**Properties used:**
- `ADM0_NAME` — country name

**Usage in app:**
- Used to clip all raster computations to selected country boundary
- Ensures only pixels within country are computed (reduces unnecessary processing)
- Provides country list for dropdown menu

**License:** [CC BY 3.0](https://creativecommons.org/licenses/by/3.0/)

**Citation:**
```
FAO/CFS (2015). Global Administrative Unit Layers (GAUL) Version 10.
Cartography & GIS Section, Food and Agriculture Organization of the United Nations.
http://www.fao.org/geonetwork/srv/en/main.home
```

**Documentation:** https://developers.google.com/earth-engine/datasets/catalog/FAO_GAUL_2015_level0

---

## Data Integration in Google Earth Engine

All datasets are accessed via the **Google Earth Engine Catalog** and are pre-processed to be on the same coordinate reference system (WGS84 / EPSG:4326).

### Aggregation and Resampling

| Input dataset | Native resolution | Aggregation method | Working resolution |
|---|---|---|---|
| SoilGrids 2.0 | 250 m | None | 250 m |
| SRTM GL1 | 30 m | Focal mean aggregate | 250 m |
| TerraClimate | ~4 km | Bilinear resampling | 250 m (for combination) |
| JRC GSW | 30 m | Distance transform + aggregate | 250 m (for combination) |

**Why this approach?** Normalizing all inputs to a common working resolution (250 m) ensures:
- Computations remain tractable for interactive Earth Engine apps
- Memory usage stays within Earth Engine limits
- Results are interpretable at country/regional scale
- Individual component layers can be toggled at any zoom level

---

## Caveats and Recommendations

### SoilGrids 2.0

- **Uncertainty:** Like all global soil datasets, SoilGrids has quantified uncertainty estimates. Higher uncertainty in regions with sparse training data (e.g., Africa, central Asia).
- **Pedotransfer functions:** Properties are predicted via machine learning (pedotransfer functions), not direct measurement everywhere.
- **Updates:** SoilGrids is updated periodically. Version 2.0 was released in 2021; future versions may improve predictions.

### SRTM GL1

- **Artifacts:** Some DEM artifacts (radar layover, foreshortening) exist in steep terrain.
- **Void fill:** No-data areas were filled using ASTER GDEM satellite stereo data.
- **January 2000:** DEM is now 24+ years old; does not reflect recent human landscape changes (deforestation, urbanization).

### TerraClimate

- **Long-term average:** Moisture ratio represents ~60 years of climate (1958–present), not current year or drought. Does not capture recent climate shifts superimposed on long-term trends.
- **Temporal coarseness:** Monthly temporal resolution may smooth out extreme drought or wet events.
- **Hydrology simplification:** Ratio (P / PET) is a simplified water-balance proxy; actual recharge depends on soil infiltration, groundwater connectivity, and human water use.

### JRC Global Surface Water

- **Seasonal variation:** Persistence thresholds (≥ 50% occurrence) may miss seasonally dry rivers or lakes.
- **Observation gaps:** Landsat data has cloud cover; persistent water is inferred, not directly observed every month.
- **Anthropogenic water:** Does not distinguish permanent natural water from dams, reservoirs, or anthropogenic ponds.

### FAO GAUL

- **Admin boundaries:** Political boundaries do not always align with hydrological or aquifer boundaries. Cross-border aquifers / basins may be split incorrectly.
- **Vintage:** Based on 2015 data; some boundary changes may have occurred since.

---

## Future Data Improvements

### High Priority

1. **Higher-resolution soil texture** (ideally 30–100 m) where available regionally
2. **Measured groundwater depth** (HAND, DTW) to refine drainage proxy
3. **Aquifer lithology** (fractured vs. alluvial vs. consolidated) to inform salinity zones
4. **Field EC or TDS calibration data** to fit model weights

### Medium Priority

5. **Borehole density maps** as a proxy for aquifer accessibility
6. **Irrigation extent** to identify secondary salinization risk zones
7. **Valley-bottom classification** (e.g., terraces, floodplains) as a hydrogeomorphic control on salinity
8. **Year-specific recharge estimates** to update static climate proxy

### Lower Priority

9. **Groundwater quality maps** from national water authorities (if publicly available)
10. **Evapotranspiration and water use data** to infer irrigation-driven salinity
11. **Landcover and land-use maps** to adjust drainage estimates for human modifications

---

## Accessing Datasets Outside Earth Engine

All datasets except FAO GAUL are publicly downloadable if you wish to use them in QGIS, ArcGIS, or other GIS software:

- **SoilGrids 2.0:** https://www.isric.org/projects/soilgrids (download by tile or WCS service)
- **SRTM GL1:** https://lpdaac.usgs.gov/products/ (search "SRTMGL1")
- **TerraClimate:** https://www.climatologylab.org/terraclimate.html (download monthly GeoTIFFs)
- **JRC GSW:** https://www.globalwaterwatch.io/ (download 30 m GeoTIFFs or Landsat-based data)

---

**Questions about a dataset?** Open an issue on [GitHub](https://github.com/washways/GlobalSoilSalinityScreener) with the label `data-inquiry`.
