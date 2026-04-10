# Component Weights: Scientific Rationale

This document explains the default weighting of the five model components and provides guidance for adjusting weights based on regional knowledge.

## Default Weights

| Component | Default Weight | Minimum | Maximum | Rationale |
|---|---:|---:|---:|---|
| Fine-textured subsoil | 32% | 20% | 40% | Strongest pedological control on drainage and salt retention |
| Weak drainage | 23% | 15% | 30% | Essential for persistence; reduces flushing |
| Flatness | 20% | 10% | 30% | Accumulation favoured in flat depositional settings |
| Climatic dryness | 17% | 10% | 25% | Evaporative concentration requires dry climate |
| Near persistent water | 8% | 0% | 15% | Hydrological indicator; can generate false positives if over-weighted |

**Total: 100%** (always normalised after user adjustment)

---

## Detailed Rationale

### 1. Fine-textured Subsoil (32%)

Fine-textured materials (clay, silt) are the primary control on water movement and capillary rise in many salinity contexts.

**Scientific basis:**

- Fine soils have lower saturated hydraulic conductivity, slowing vertical water movement
- Capillary rise is stronger in fine-textured soil (up to 2–3 m in clays vs. < 0.3 m in sands)
- Capillary rise brings dissolved salts toward the surface, where evaporation concentrates them
- SoilGrids texture at 100–200 cm is more representative of long-term drainage behaviour than topsoil

**Literature:**

- https://www.mdpi.com/2073-4395/15/9/2239 — Salinity control by soil properties
- Hillel & Vanden Eijnden (1975) — Capillary rise and water retention curves

**Why 32% and not higher?**

- While soil texture is a first-order control, it cannot operate alone
- A fine-textured, dry soil in a humid climate may remain fresh because effective leaching exceeds capillary rise
- Therefore, texture must be combined with drainage, climate, and topographic context

### 2. Weak Drainage (23%)

Poor drainage is central to salinity persistence across the literature.

**Scientific basis:**

- Saline groundwater occurs where salts accumulate without flushing
- Drainage capacity depends on soil texture, structure, groundwater depth, and restrictive layers
- In poorly drained settings, upward capillary salt flux is not offset by lateral or downward leaching
- Over seasonal timescales, weak drainage favours soil salinization in irrigated areas

**Literature:**

- https://www.mdpi.com/2073-4395/15/9/2239 — Drainage explicitly linked to salt persistence
- Risk Assessment of Irrigation-Related Soil Salinization (Mediterranean, 2021)
- WHO/FAO reports on irrigation salinity

**Why 23% and not higher?**

- Drainage is a strong factor, but it is also correlated with texture (coarse = free-draining; fine = poor)
- If texture is already at 32%, doubling drainage would double-count the same pedological control
- The 23% allocation respects the independence of other factors (climate, terrain, hydrology)

### 3. Flatness (20%)

Relief is a fundamental control on whether salts accumulate or are flushed.

**Scientific basis:**

- Flat terrain = depositional, poorly drained, lowland settings (favourable to accumulation)
- Steep terrain = runoff-dominated, well-drained, ridges (less favourable for salt storage)
- Regional salinity surveys consistently show higher concentrations in valley bottoms and low-lying areas
- Terrain suppression is essential to avoid the "additive fallacy" of earlier models

**Literature:**

- https://www.researchgate.net/publication/347511193_Risk_Assessment_of_Irrigation-Related_Soil_Salinization_and_Sodification_in_Mediterranean_Areas
- Terrain analysis and salinity mapping (FAO)

**Why 20% and not higher?**

- Flat terrain alone does not cause salinity if climate is humid or soils are very coarse
- The weight of 20% balances terrain with pedological factors (texture + drainage = 55%)
- A separate terrain *suppression* multiplier further penalizes steep terrain, so flatness doesn't have to carry the full load

### 4. Climatic Dryness (17%)

Evaporative concentration is a hallmark of salinity formation.

**Scientific basis:**

- Where P < PET (precipitation less than evapotranspiration), water is in deficit and salts concentrate
- Where P > PET, leaching dominates and salts are flushed downward
- Long-term aridity index (P / PET) is a practical proxy for this balance
- Arid and semi-arid zones are predictably prone to salinity issues

**Literature:**

- https://www.nature.com/articles/sdata2017191 — TerraClimate validation
- Zomer et al. (2022) — Global Aridity Index
- Salinity surveys in drylands consistently show correlation with local aridity

**Why 17% and not higher?**

- Climate alone cannot explain where salinity occurs within a landscape
- Two adjacent pixels with identical climate but different texture/drainage will have very different salinity risk
- The 17% allocation reflects that climate is important but not deterministic
- Other factors (soil, drainage, terrain) create the *structure* within which climate operates

### 5. Near Persistent Water (8%)

Surface water proximity indicates hydrological connectivity and lowland position.

**Scientific basis:**

- Persistent water bodies (lakes, rivers, wetlands) mark lowland, often poorly drained, hydrologically connected zones
- These settings can accumulate salts from upslope groundwater flow
- Endorheic (closed-basin) settings send all drainage to low points where salt concentrates
- Floodplain and alluvial settings are often saline due to multiple cycles of flood-deposited sediment and evaporative concentration

**Literature:**

- https://www.nature.com/articles/nature20584 — JRC GSW validation; lowland water and hydrology
- Alluvial landform and salinity studies (various)

**Why 8% and not higher?**

- The relationship is context-dependent: not all areas near water are saline (coastal sabkhas vs. humid river valleys)
- The weight of 8% helps identify plausible hydrological zones without over-emphasizing a secondary indicator
- Using distance-to-water (not just occurrence) makes the signal more nuanced

**Why not 0%?**

- Lowland zones and floodplains are genuinely important in salinity contexts
- Malawi Lower Shire example: salinity correlates with alluvial valley position near the Shire River
- A non-zero weight captures this real but context-dependent phenomenon without letting it dominate

---

## Adjusting Weights for Your Region

### When to Adjust

Consider adjusting default weights if you have:

1. **Local field data** (EC measurements, borehole logs) showing different patterns
2. **Published studies** for your region quantifying relative importance of factors
3. **Expert knowledge** of local hydrogeology or agriculture
4. **Disagreement** between model output and known salinity zones

### Example Adjustments

#### Arid Region with Shallow Water Table

If your region has persistent shallow groundwater and strong evaporative concentration:

- **Climatic dryness ↑ 20%** (from 17%) — evaporation is the dominant process
- **Weak drainage ↓ 18%** (from 23%) — water table proximity matters more than soil texture
- **Fine-textured subsoil ↓ 28%** (from 32%) — texture is less important than water table
- **Flatness, Near water** — keep near default

**Justification:** In shallow-WT settings, upward capillary flow dominates. Texture matters less than proximity to water.

#### Irrigated Semi-arid Region

If your region has extensive canal or groundwater irrigation feeding surface salinity:

- **Weak drainage ↑ 28%** (from 23%) — irrigation water ponding and secondary salinization are key
- **Flatness ↑ 23%** (from 20%) — flat command areas concentrate irrigation water
- **Climatic dryness ↓ 14%** (from 17%) — climate matters less than irrigation loading
- **Fine-textured subsoil ↓ 28%** (from 32%) — irrigation salinity affects all textures

**Justification:** Irrigation changes the hydrological regime fundamentally; climate becomes secondary to water management.

#### Humid Region with Geological Salinity

If your region has evaporite (halite, gypsum) or other mineralogical salinity sources:

- **Near persistent water ↑ 12%** (from 8%) — salinity from weathering; discharge to lowlands
- **Flatness ↑ 24%** (from 20%) — accumulation in low-lying zones
- **Climatic dryness ↓ 12%** (from 17%) — climate is less important (salinity is geological, not evaporative)
- **Weak drainage, Fine-textured subsoil** — keep default

**Justification:** Geological sources dominate; drainage and tone affect WHERE salts go, not WHETHER they exist.

---

## How to Test Your Adjusted Weights

1. **Use the app's weight sliders** to set your new values
2. **Click "Reload map"** to recompute with your adjusted weights
3. **Compare to field data** or published salinity maps:
   - Do high-susceptibility zones now align better with known saline areas?
   - Do low-susceptibility zones avoid known freshwater zones?
   - Is the geographic pattern more realistic?
4. **Iterate** if needed; there is no single "correct" set of weights without field validation

---

## Future Refinements

### Machine Learning Calibration

A future version can fit weights using field EC data:

1. Collect 50+ field EC measurements across a region
2. Extract component values at those locations
3. Use regression or classification to find optimal weights
4. Test on independent holdout data
5. Report regional weight presets

### Bayesian Prior

Weights could be treated as Bayesian priors, allowing users to specify confidence intervals (e.g., "I'm 80% confident texture is the strongest factor, with ± 5% uncertainty").

### Adaptive Weighting

Different weights could be applied to different regions (e.g., arid vs. humid, alluvial vs. bedrock) based on aquifer type or lithology.

---

## Summary Table: Weight Choices and Their Implications

| Scenario | Fine-texture ↑ | Drainage ↑ | Flatness ↑ | Dryness ↑ | Water ↑ | Implication |
|---|---|---|---|---|---|---|
| Default | 32% | 23% | 20% | 17% | 8% | Balanced, literature-driven |
| Texture dominates | 40% | 20% | 18% | 15% | 7% | Fine soils = primary control |
| Drainage crisis | 20% | 30% | 20% | 20% | 10% | Poor flushing is key |
| Terrain matters | 25% | 25% | 30% | 15% | 5% | Topography drives accumulation |
| Climatic aridity | 20% | 15% | 18% | 35% | 12% | Extreme dryness determines salinity |
| Hydrological | 25% | 22% | 20% | 15% | 18% | Water bodies / lowlands key |

---

**Questions on weights?** Open an issue on [GitHub](https://github.com/washways/GlobalSoilSalinityScreener) or see [CONTRIBUTING.md](../CONTRIBUTING.md) for how to share calibration data.
