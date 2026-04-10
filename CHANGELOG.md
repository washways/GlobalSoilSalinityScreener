# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.0.0] - 2026-04-09

### Added

- **Initial release** of Global Soil Salinity Screener
- Multi-factor susceptibility model combining:
  - Fine-textured subsoil (32%)
  - Weak drainage (23%)
  - Flatness (20%)
  - Climatic dryness (17%)
  - Near persistent water (8%)
- Country-scoped computation with FAO GAUL boundaries
- Adjustable component weights (bounded, sum-to-100)
- Terrain suppression based on slope (reduces false positives in steep terrain)
- Component layer visibility in Layers panel for model inspection
- Click-to-query diagnostics with popup showing score and component breakdown
- GitHub Pages landing page with embedded GEE app
- Comprehensive methodology documentation grounded in salinity literature
- Support for all countries globally (with Malawi as default starting view)

### Features

- **Data sources**: SoilGrids, SRTM, TerraClimate, JRC Global Surface Water, FAO GAUL
- **Visualization**: Power-transform display stretch; red reserved for highest-risk areas
- **Usability**: Manual "Reload map" button to control computational efficiency
- **Documentation**: Full METHODOLOGY.md with scientific rationale and citations

### Known Limitations

- Not calibrated to field EC or groundwater chemistry measurements
- Screening tool only; not for regulatory or formal prediction use
- Resolution limited by coarsest input (250 m for SoilGrids)
- Weighting is heuristic, not statistically fitted
- No water quality component assessed
- Static climate proxy (long-term average, not year-specific)

---

## Future Roadmap

### Version 1.1 (Planned)

- [ ] Ground-truth validation using field EC from Malawi Lower Shire
- [ ] Sub-Saharan Africa regional weight presets based on calibration data
- [ ] Higher-resolution soil texture layer for Malawi (if available)
- [ ] HAND-based drainage indicator as alternative to texture proxy
- [ ] Improved documentation with regional case studies

### Version 2.0 (Aspirational)

- [ ] Machine learning calibration against field salinity observations
- [ ] Aquifer lithology integration (fissured vs. alluvial)
- [ ] Groundwater depth incorporation
- [ ] Irrigation extent overlay for secondary salinization risk
- [ ] Mobile-friendly app wrapper (React Native / Flutter)
- [ ] Multi-language support (French, Portuguese, Amharic)
- [ ] High-resolution exports for GIS integration
- [ ] National-level pre-computed assets for faster access

---

## Contributing

See [CONTRIBUTING.md](./CONTRIBUTING.md) for guidelines on contributing data, code, or documentation.
Priority areas are ground-truthing data, regional calibration, and improved soil datasets.

---

**Built by [WASHways](https://washways.org/) — Improving water access through open science and technology.**
