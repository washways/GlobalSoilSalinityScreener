# Methodology: Multi-factor Salinity Susceptibility Screening

## Overview

This screening layer is designed to identify places where the **environmental setting is more favourable to salinity accumulation or persistence**, not to measure salinity directly. It is therefore a **susceptibility surface**, not an electrical conductivity map, groundwater chemistry map, or regulatory classification. The design is grounded in the current salinity literature, which consistently shows that salinity patterns are typically driven by a combination of **soil texture, drainage, topographic setting, climatic dryness, hydrological position, and groundwater or surface-water interactions**, rather than by any one variable alone.

Source: https://www.sciencedirect.com/science/article/pii/S1569843224002334

The current application evolved from an earlier Google Earth Engine prototype that used SoilGrids sand, clay, silt, and soil organic carbon at 100 to 200 cm depth to calculate a weighted texture index and a normalized texture contrast index. The present version improves that logic by moving from a largely texture-only heuristic to a broader multi-factor susceptibility design.

---

## Conceptual Basis

The model assumes that salinity is more plausible where the following conditions coincide:

### Fine-textured subsoil
Increases the likelihood of slower percolation, greater water retention, and stronger capillary rise, all of which can favour salt storage or upward salt movement under the right groundwater and climatic conditions.

Source: https://www.mdpi.com/2073-4395/15/9/2239

### Weak drainage
Reduces flushing and promotes salt persistence. In salinity mapping and salinization risk studies, poor drainage is repeatedly identified as a central control.

Source: https://www.mdpi.com/2073-4395/15/9/2239

### Low-relief topography
Generally more favourable to accumulation than steep slopes, because flatter areas are more likely to behave as depositional, poorly drained, or evaporative settings.

Source: https://www.researchgate.net/publication/347511193_Risk_Assessment_of_Irrigation-Related_Soil_Salinization_and_Sodification_in_Mediterranean_Areas

### Climatic dryness
Increases the probability that salts concentrate rather than being leached, especially where evapotranspiration pressure exceeds effective rainfall recharge.

Source: https://www.mdpi.com/2073-4395/15/9/2239

### Nearness to persistent surface water
Can indicate lowland, floodplain, valley-bottom, endorheic, or hydrologically connected settings where salinity processes are more plausible, although this factor is context-dependent and therefore weighted lower than texture and drainage.

Source: https://www.nature.com/articles/nature20584

---

## Data Sources

| Input | Source and vintage | Native resolution | Role in model |
|---|---|---:|---|
| Sand, clay, silt, SOC at 100 to 200 cm | SoilGrids 2.0, documented in 2021; predictions at standard depths | 250 m | Represents subsoil texture and carbon conditions relevant to drainage and salt retention |
| Elevation and slope | SRTM GL1 v003, acquired in February 2000 | ~30 m | Used to derive slope and terrain suppression |
| Persistent surface water occurrence | JRC Global Surface Water v1.4, based on Landsat imagery from 1984 to 2021 | 30 m | Used to estimate distance to persistent water and lowland hydrological influence |
| Climate moisture ratio | TerraClimate, published in 2018 and updated annually; monthly water balance variables from 1958 to present | ~4 km, monthly | Used as a long-term dryness signal via precipitation to potential evapotranspiration ratio |
| Country boundaries | FAO GAUL 2015 administrative boundaries | vector | Used only to clip computation to the selected country |

Source links:

- SoilGrids FAQ: https://docs.isric.org/globaldata/soilgrids/SoilGrids_faqs.html
- SRTM GL1 v003 in Earth Engine: https://developers.google.com/earth-engine/datasets/catalog/USGS_SRTMGL1_003
- JRC Global Surface Water v1.4 in Earth Engine: https://developers.google.com/earth-engine/datasets/catalog/JRC_GSW1_4_GlobalSurfaceWater
- TerraClimate: https://www.climatologylab.org/terraclimate.html
- FAO GAUL 2015 level 0 in Earth Engine: https://developers.google.com/earth-engine/datasets/catalog/FAO_GAUL_2015_level0

---

## Variable Construction

The layer is built from five component surfaces, each normalized to a 0 to 1 range.

### 1. Fine-textured Subsoil Component

This component is constructed from SoilGrids sand, clay, silt, and SOC at 100 to 200 cm depth, using a fines-dominance logic. The purpose is to highlight areas where the subsoil is more likely to slow water movement and support salt retention or capillary redistribution. Using the 100 to 200 cm interval is defensible because salinity persistence and drainage behaviour are often controlled by subsurface material, not just the uppermost topsoil.

Source: https://docs.isric.org/globaldata/soilgrids/SoilGrids_faqs.html

### 2. Weak Drainage Component

This is derived from the inverse of a texture-drainage proxy. Coarser subsoils are treated as more freely draining and finer subsoils as less freely draining. This follows the general hydropedological literature, while acknowledging that actual drainage also depends on structure, groundwater depth, restrictive layers, and land use.

Source: https://www.mdpi.com/2073-4395/15/9/2239

### 3. Flatness Component

Slope from SRTM is inverted so that flatter terrain receives higher flatness favourability. The purpose is not to claim that all flat terrain is saline, but to represent the empirical tendency for accumulation processes to be more likely in low-relief settings than on steep hillslopes.

Source: https://developers.google.com/earth-engine/datasets/catalog/USGS_SRTMGL1_003

### 4. Climatic Dryness Component

A moisture ratio based on long-term TerraClimate precipitation and potential evapotranspiration is inverted so that drier settings score higher. This is a practical approximation of salinity-favouring climatic balance, because salts accumulate more readily where evaporative concentration exceeds effective leaching.

Source: https://www.nature.com/articles/sdata2017191

### 5. Near Persistent Water Component

Distance to persistent surface water is derived from JRC Global Surface Water occurrence. Nearness is treated as a secondary indicator of hydrological position, especially floodplains, valley bottoms, and other wet-to-dry transition environments where salinity may emerge or persist. Because this relationship is less universal than the texture-drainage-dryness relationship, it is deliberately given the lowest default weight.

Source: https://developers.google.com/earth-engine/datasets/catalog/JRC_GSW1_4_GlobalSurfaceWater

---

## Default Weighting and Rationale

The default weights sum to 100 and are intentionally interpretable rather than purely data-driven:

### Fine-textured subsoil: 32%
This is the strongest term because finer subsoil commonly exerts first-order control on permeability, storage, capillary rise, and salt retention.

### Weak drainage: 23%
This is the second strongest term because poor flushing is central to salinity persistence.

### Flatness: 20%
Flatness is weighted strongly because relief affects whether salts accumulate or are flushed downslope.

### Climatic dryness: 17%
Dryness matters materially, but the model keeps it below the pedological terms because climate alone does not explain where salts concentrate within a landscape.

### Near persistent water: 8%
This remains a secondary factor because it helps identify hydrologically plausible accumulation zones, but can generate false positives if over-emphasized.

Supporting sources:

- https://www.mdpi.com/2073-4395/15/9/2239
- https://www.sciencedirect.com/science/article/pii/S1569843224002334
- https://www.researchgate.net/publication/347511193_Risk_Assessment_of_Irrigation-Related_Soil_Salinization_and_Sodification_in_Mediterranean_Areas
- https://www.nature.com/articles/sdata2017191
- https://www.nature.com/articles/nature20584

This weighting structure is therefore literature-informed and mechanistically plausible, but it should still be described honestly as **heuristic rather than statistically calibrated**. In a future version, the strongest upgrade would be to calibrate these terms against field EC, groundwater chemistry, groundwater depth, and valley-bottom metrics such as HAND.

Source: https://www.sciencedirect.com/science/article/pii/S1470160X25006181

---

## Terrain Suppression

The model includes a separate terrain suppression step based on slope, so that increasingly steep terrain progressively reduces final susceptibility. This was added because otherwise dry, fine-textured hills can be overstated, whereas salinity is more commonly associated with flatter lowlands, valley bottoms, floodplains, endorheic settings, and poorly drained plains.

Source: https://www.researchgate.net/publication/347511193_Risk_Assessment_of_Irrigation-Related_Soil_Salinization_and_Sodification_in_Mediterranean_Areas

---

## Malawi-specific Rationale

Although the application is global, the Malawi starting view is scientifically sensible because groundwater salinity is a known issue in parts of Malawi, especially the Lower Shire and related alluvial settings. Published work and regional atlases link salinity there to hydrogeological context, evaporative concentration, and alluvial or lowland conditions rather than to a single soil property alone.

Source: https://strathprints.strath.ac.uk/87278/7/Kalin-etal-MSW-2022-Hydrogeology-and-Groundwater-Quality-Atlas-of-Malawi-Water-Resource-Area-1-The-Lower-Shire.pdf

---

## Limits of Interpretation

This layer should **not** be described as a groundwater salinity map, drinking-water quality map, or agricultural suitability map on its own. It does not yet include groundwater depth, aquifer connectivity, irrigation salinity loading, detailed lithology, valley-bottom metrics, or field salinity calibration. The literature is clear that the highest-performing salinity models usually integrate field EC or ECe with multiple environmental covariates and, increasingly, machine learning.

Source: https://www.sciencedirect.com/science/article/pii/S1569843224002334

---

## Code Design Principles

The application is structured around five engineering principles:

### 1. Country-scoped Computation

The app clips every raster calculation to the currently selected country boundary from FAO GAUL. That design keeps the workflow tractable for interactive use and avoids doing unnecessary global raster processing every time the user changes a parameter.

### 2. Layered Model Structure

The code separates:
- Source data accessors
- Derived component layers
- The combined susceptibility model
- Visualization settings
- Map layer management
- User interface controls

That separation makes the logic auditable and allows the component layers to be inspected independently in the standard Earth Engine Layers panel.

### 3. Deferred, On-Demand Execution

Earth Engine uses deferred execution, so the code constructs computation graphs and only materializes them when the map needs tiles or the user reloads the model. The app design leans into that behaviour by rebuilding layers only when country or parameter state changes.

### 4. Reproducible but Adjustable Weighting

The model exposes bounded weights for the five main components. Users can change the emphasis, but only within plausible ranges, and the total is kept at 100 by automatic rebalancing. This preserves a controlled design space while still allowing scenario testing.

### 5. Display Separated from Model Logic

The visualization layer is intentionally separated from the underlying model. In the current implementation, the displayed salinity surface uses a power transform to compress mid-range values so that red is reserved for the highest-risk areas. This means the color map is easier to interpret without changing the underlying normalized model values.

---

## Suggested Summary for Users

The application builds a country-specific salinity susceptibility surface from globally available environmental covariates. SoilGrids provides subsoil texture and SOC at 250 m; SRTM provides 30 m terrain; JRC Global Surface Water provides 30 m persistent surface-water occurrence; TerraClimate provides monthly climate water-balance information at approximately 4 km. Each component is normalized, weighted, combined, and then terrain-suppressed to reduce false positives in steep terrain. The final display stretch is intentionally conservative, so only the highest-risk areas plot in red.

---

## Recommended Caveat

This application is a literature-informed heuristic screening tool. It is not calibrated to field EC or groundwater chemistry and should not be used as a stand-alone decision tool for drilling, irrigation, or drinking-water planning. The most defensible next step is local calibration using field salinity observations, groundwater depth, lithology, and valley-bottom metrics.

Source: https://www.sciencedirect.com/science/article/pii/S1569843224002334
