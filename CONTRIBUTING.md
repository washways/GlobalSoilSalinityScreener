# Contributing to Global Soil Salinity Screener

We welcome contributions! This document outlines how to get involved.

## Types of Contributions

### 1. Field Calibration Data 🌍

Do you have measured groundwater EC, electrical conductivity, salinity measurements, or SAR data? Consider sharing it:

- **Ground-truth observations**: Point EC measurements, borehole hydrochemistry, TDS, or sodicity indices
- **Regional validation datasets**: Existing EC maps, soil salinity surveys, or published borehole logs
- **Feedback on screening accuracy**: Does the susceptibility layer match known salinity zones in your region?

These datasets are valuable for future model calibration and validation.

### 2. Regional Adaptation 🗺️

Help adapt the model to new regions:

- **Regional weight presets**: Suggest optimal weights for specific countries or climatic zones based on local expertise
- **Dry-season month refinement**: Identify the appropriate peak dry season for NDVI or aridity calculations
- **Ground-truthing studies**: Compare screening scores to field observations in new regions
- **Case studies & documentation**: Document how the tool performs in local contexts

### 3. New Data Sources 📊

Propose improvements to model inputs:

- **Higher-resolution soil maps**: Regionally available soil texture or properties datasets
- **Aquifer lithology**: Geological formation maps, well logs, or aquifer productivity data
- **Irrigation extent**: Maps of irrigated areas (helps identify secondary salinization risk)
- **Groundwater depth**: HAND alternatives or measured water-table depth maps
- **Borehole density mapping**: Data on borehole distribution as a proxy for yield accessibility

### 4. Code & Documentation 💻

Contribute to the codebase or documentation:

- **Bug fixes**: Report or fix errors in the GEE script or landing page
- **UX improvements**: Better visualisations, accessibility, mobile responsiveness, or new control widgets
- **Documentation**: Write tutorials, case studies, regional guides, or translate documentation
- **Testing**: Systematic testing on large countries or edge cases

---

## How to Contribute

### For Data or Observations

1. Open a GitHub issue with the label `data` describing what you have
2. Include:
   - Geographic location and spatial extent
   - Type of data (EC measurements, borehole logs, published maps, etc.)
   - Data format and availability (public? can it be published?)
   - Contact information
3. We'll discuss integration options

### For Code Changes

1. **Fork** the repository
2. **Create a feature branch**: `git checkout -b feature/your-feature-name`
3. **Make your changes** (see Code Style below)
4. **Test thoroughly** — especially for GEE scripts, test on multiple countries
5. **Commit with clear messages**: `git commit -m "Add [feature]: description"`
6. **Push to your fork**: `git push origin feature/your-feature-name`
7. **Open a Pull Request** describing:
   - What the change does
   - Why it's needed
   - Any testing you've done
   - Related issues (if any)

### For Documentation

1. Fork and edit the relevant `.md` file
2. Follow the structure of existing documentation
3. Add sources and citations where appropriate
4. Submit a pull request

---

## Code Style

### GEE Scripts (JavaScript)

- Use meaningful variable names: `fineTextureSusceptibility` not `x`
- Comment complex logic: multi-component derivations, non-obvious normalizations
- Keep source data accessors in a dedicated section
- Normalize all inputs to 0–1 before weighting
- Test on multiple countries and zoom levels before submitting

### Landing Page (HTML/CSS/JavaScript)

- Use semantic HTML5
- Mobile-responsive design (test on tablets and phones)
- Keep client-side logic minimal; defer computation to GEE
- Comment non-obvious CSS or JavaScript
- Validate HTML: [W3C Validator](https://validator.w3.org/)

### Markdown

- Use clear headings and subheadings
- Include links and citations to sources
- Use tables for structured information
- Keep line lengths ≤ 100 characters for readability in text editors

---

## Reporting Issues

Found a bug? Have a question?

1. **Check existing issues** to avoid duplicates
2. **Open a new issue** with:
   - Clear title describing the problem
   - Steps to reproduce (if applicable)
   - Expected vs. actual behaviour
   - Screenshots or diagnostic data (for map issues)
   - System info (browser, OS, GEE account type)

---

## Review Process

- Maintainers will review PRs within 1–2 weeks
- Large changes or new features may require discussion before merging
- We'll request changes if needed (don't take it personally!)
- Once approved, your contribution will be merged and credited

---

## Licensing

By contributing, you agree that your contributions will be licensed under the [MIT License](./LICENSE). This includes any data, code, documentation, or ideas you share.

---

## Priority Areas (2026)

We're currently seeking help with:

1. **Malawi ground-truthing**: EC measurements in Lower Shire, Shire Valley
2. **Sub-Saharan Africa regional presets**: Weight calibration for Botswana, Namibia, South Africa, Kenya
3. **High-resolution soil data**: Regional 30 m soil texture maps where available
4. **Mobile app wrapper**: React Native or Flutter app to run locally on phones
5. **Multi-language documentation**: French, Portuguese, and Amharic translations

---

## Questions?

- Open an issue with the label `question`
- Contact the maintainers via [washways.org](https://washways.org/)

Thank you for helping make groundwater salinity science more accessible! 🌍
