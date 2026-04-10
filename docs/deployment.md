# Deployment Guide

This document provides step-by-step instructions for deploying the Global Soil Salinity Screener application and landing page.

---

## Part 1: Deploying the GEE App

### Prerequisites

- A [Google Earth Engine account](https://earthengine.google.com/) with access to the GEE Code Editor
- A Google Cloud Project with Earth Engine API enabled
- The Google Earth Engine CLI (optional, for batch ops)

### Step 1: Create a GEE Code Editor Project

1. Log into [Google Earth Engine Code Editor](https://code.earthengine.google.com/)
2. Create a new script or open an existing one
3. Copy the complete GEE script from `gee/global_soil_salinity_screener.js` into the Code Editor
4. Verify the script runs without errors by clicking **Run** and testing on a small country (e.g., Malawi)

### Step 2: Copy and Customize (Optional)

Before publishing, you may want to customize:

- **Default country:** Edit the `DEFAULT_CENTER` and `DEFAULT_ZOOM` constants
- **Component weights:** Adjust `DEFAULT_WEIGHTS` if desired
- **Output resolution:** Change `DEFAULT_RESOLUTION` (caution: affects computation time)
- **Color palette:** Modify the `vizParams` for the display stretch

### Step 3: Publish as a GEE App

1. In the Code Editor menu, click **Apps** → **New app**
2. Name your app (e.g., "Global Soil Salinity Screener")
3. Copy the script into the app editor
4. Click **Publish** (Earth Engine will generate a unique URL)
5. Share the app URL via GitHub README or website

**Published app URL format:**
```
https://washways.projects.earthengine.app/view/globalsoilsalinityscreener
```

### Step 4: Monitor and Update

- Check app performance via the [GEE Apps dashboard](https://earthengine.google.com/apps/)
- Monitor for runtime errors or performance issues
- Update the script if datasets become unavailable
- Re-publish whenever you make code changes

---

## Part 2: Deploying the GitHub Pages Landing Page

### Prerequisites

- A [GitHub account](https://github.com/)
- A GitHub repository for your project (e.g., `GlobalSoilSalinityScreener`)
- Git installed locally or GitHub Desktop

### Step 1: Create or Clone the Repository

**If starting fresh:**

```bash
mkdir GlobalSoilSalinityScreener
cd GlobalSoilSalinityScreener
git init
```

**If cloning from existing:**

```bash
git clone https://github.com/washways/GlobalSoilSalinityScreener.git
cd GlobalSoilSalinityScreener
```

### Step 2: Organize Files

Ensure your repository has this structure:

```
GlobalSoilSalinityScreener/
├── README.md
├── METHODOLOGY.md
├── CONTRIBUTING.md
├── CHANGELOG.md
├── LICENSE
├── .gitignore
├── site/
│   └── index.html
├── docs/
│   ├── datasets.md
│   ├── deployment.md
│   ├── architecture.md
│   └── component-weights.md
└── gee/
    └── global_soil_salinity_screener.js
```

### Step 3: Configure GitHub Pages

1. **Push to GitHub:**
   ```bash
   git add .
   git commit -m "Initial commit: Global Soil Salinity Screener"
   git branch -M main
   git remote add origin https://github.com/YOUR_USERNAME/GlobalSoilSalinityScreener.git
   git push -u origin main
   ```

2. **Enable GitHub Pages:**
   - Go to your repository on GitHub
   - Click **Settings** (repository settings, not user settings)
   - Scroll to **Pages** (left sidebar)
   - Under "Source", select **Deploy from a branch**
   - Select `main` branch and `/root` folder
   - Click **Save**

3. **Wait for deployment:**
   - GitHub will build and deploy automatically
   - Check **Actions** tab to monitor the deployment workflow
   - You should see a green checkmark ✅ when complete
   - Your site will be available at:
     ```
     https://your-username.github.io/GlobalSoilSalinityScreener/
     ```

### Step 4: Verify the Site

1. Visit your GitHub Pages URL
2. Confirm that:
   - The header and footer display correctly
   - The embedded GEE app iframe loads (may take 10–30 seconds)
   - All navigation links work
   - Tables and components render properly
3. Test on mobile devices (check viewport sizing)

### Step 5: Add Custom Domain (Optional)

To use a custom domain (e.g., `salinity.washways.org`):

1. **Update DNS:**
   - Add a `CNAME` record pointing to `your-username.github.io`
   - Or add `A` records pointing to GitHub's IP addresses (see [docs](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/managing-a-custom-domain-for-your-github-pages-site))

2. **Configure GitHub Pages:**
   - Go to repository **Settings** → **Pages**
   - Under "Custom domain", enter your domain (e.g., `salinity.washways.org`)
   - GitHub will verify DNS and enable HTTPS

3. **Commit the CNAME file:**
   ```bash
   echo "salinity.washways.org" > CNAME
   git add CNAME
   git commit -m "Add custom domain"
   git push
   ```

---

## Part 3: Continuous Deployment

### Automated GitHub Actions Workflow

To automate GitHub Pages deployment (optional):

Create `.github/workflows/deploy.yml`:

```yaml
name: Deploy GitHub Pages

on:
  push:
    branches: ["main"]
  pull_request:
    branches: ["main"]

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: "pages"
  cancel-in-progress: false

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v3

      - name: Setup Pages
        uses: actions/configure-pages@v3

      - name: Build
        run: |
          # Optional: add build steps (e.g., markdown to HTML conversion)
          echo "Site is ready to deploy"

      - name: Upload artifact
        uses: actions/upload-pages-artifact@v2
        with:
          path: 'site/'

  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v2
```

This workflow will automatically rebuild and deploy the site whenever you push to the `main` branch.

---

## Part 4: Updating Content

### Updating the Landing Page

To update the landing page (HTML):

1. Edit `site/index.html` locally
2. Test in a browser
3. Commit and push:
   ```bash
   git add site/index.html
   git commit -m "Update landing page: [what changed]"
   git push
   ```
4. GitHub Pages will redeploy automatically (usually within 1 minute)

### Updating Documentation

To update markdown documentation (README, METHODOLOGY, etc.):

1. Edit the `.md` file
2. Commit and push:
   ```bash
   git add README.md
   git commit -m "Update README: [what changed]"
   git push
   ```
3. Changes are reflected immediately on GitHub (and pulled by the landing page via GitHub API if implemented)

### Updating the GEE Script

To release a new version of the GEE app:

1. Update `gee/global_soil_salinity_screener.js`
2. Test in the Code Editor
3. Publish a new version via **Apps** → **Publish**
4. Update the version number in `CHANGELOG.md`
5. Commit and push:
   ```bash
   git add gee/global_soil_salinity_screener.js CHANGELOG.md
   git commit -m "Release v1.1: [description of changes]"
   git push
   git tag v1.1
   git push --tags
   ```

---

## Troubleshooting

### GEE App Won't Load

1. Check that the Earth Engine account is active and authorized
2. Verify the script has no syntax errors (run in Code Editor first)
3. Ensure datasets (SoilGrids, SRTM, TerraClimate, JRC GSW, FAO GAUL) are still available in the catalog
4. Check for timeout errors in large countries (try a smaller AOI first)

### GitHub Pages Not Deploying

1. Verify that GitHub Pages is enabled in **Settings** → **Pages**
2. Check the **Actions** tab for workflow errors
3. Ensure `site/index.html` exists and is valid
4. If using a custom domain, verify DNS records are correct

### Map Iframe Not Loading

1. Check that the GEE app URL is correct and publicly accessible
2. Verify the iframe URL matches the published GEE app URL
3. Ensure the GEE app doesn't require authentication (must be publicly shared)
4. Check browser console for CORS or security warnings

### Performance Issues

1. **Large countries slow to load:** Reduce output resolution or split computation
2. **GEE app timing out:** Limit AOI size or use a smaller country
3. **GitHub Pages slow:** Ensure HTML/CSS is optimized; avoid large embedded iframes

---

## Monitoring and Maintenance

### Regular Checks

- **Monthly:** Test the app on a new country; verify no runtime errors
- **Quarterly:** Review CHANGELOG for upstream dataset updates
- **Annually:** Audit dependencies and update documentation

### Reporting Issues

If you encounter problems:

1. Open an issue on [GitHub](https://github.com/washways/GlobalSoilSalinityScreener) with:
   - Error message / screenshot
   - Country / region affected
   - Browser / OS
   - Steps to reproduce

2. Check existing issues first to avoid duplicates

---

## Related Documentation

- [Google Earth Engine Apps Documentation](https://developers.google.com/earth-engine/guides/apps)
- [GitHub Pages Documentation](https://docs.github.com/pages/)
- [GitHub Pages custom domains](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/)

---

**Questions on deployment?** See [CONTRIBUTING.md](../CONTRIBUTING.md) or open an issue on GitHub.
