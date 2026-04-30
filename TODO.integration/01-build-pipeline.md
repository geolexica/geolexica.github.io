# 01 — Build Pipeline Using npm Package CLI

## Goal

The geolexica.org repository builds and deploys www.geolexica.org using the published `glossarist-vocabulary-browser` npm package. No checkout of vocabulary-browser source code.

## Current State

- `.github/workflows/build_deploy.yml` checks out `glossarist/vocabulary-browser` at `main`
- Runs `npm ci`, `npm run fetch-datasets`, `npm run build-gcr:all`, `npm run generate-data`, `npm run build`
- Deploys `dist/` to GitHub Pages
- `repository_dispatch` trigger from vocabulary-browser

## Tasks

### 1. Move `datasets.yml` into this repo

The dataset registry is deployment configuration — it belongs here, not in the vocabulary-browser package.

```bash
# Create datasets.yml in repo root
cp /path/to/vocabulary-browser/datasets.yml ./datasets.yml
```

The `datasets.yml` is the source of truth for which datasets to include.

### 2. Rewrite `build_deploy.yml`

```yaml
name: build_deploy

on:
  push:
    branches: [main]
  repository_dispatch:
    types: [deploy]
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

jobs:
  build:
    runs-on: ubuntu-latest
    environment:
      name: production
      url: https://www.geolexica.org
    steps:
      - uses: actions/checkout@v4  # checkout THIS repo (for datasets.yml)

      - uses: actions/setup-node@v4
        with:
          node-version: 20

      - name: Install glossarist-browser
        run: npm install glossarist-vocabulary-browser

      - name: Fetch datasets (download GCR packages)
        run: npx glossarist-browser fetch --config datasets.yml

      - name: Generate data (GCR → JSON-LD)
        run: npx glossarist-browser generate --config datasets.yml

      - name: Build edges
        run: npx glossarist-browser build-edges

      - name: Build SPA
        run: npx glossarist-browser build

      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: dist

  deploy:
    needs: build
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

Key changes:
- No `actions/checkout` of vocabulary-browser — install from npm
- `datasets.yml` lives in this repo
- No `npm ci` on vocabulary-browser source
- No `build-gcr:all` step

### 3. Remove vocabulary-browser checkout

Delete the step that checks out `glossarist/vocabulary-browser`. This repo no longer needs access to vocabulary-browser source code.

### 4. Add `package.json` for dependency tracking

```json
{
  "name": "geolexica.org",
  "private": true,
  "dependencies": {
    "glossarist-vocabulary-browser": "^0.1.0"
  }
}
```

Or use `npx` without installing (simpler for CI).

### 5. Keep `repository_dispatch` trigger

The vocabulary-browser repo (on push to main) should still trigger a rebuild here so the latest SPA code is picked up. But now it triggers an `npx` install of the latest npm version instead of a source checkout.

## Acceptance Criteria

- [ ] No checkout of vocabulary-browser source
- [ ] `datasets.yml` lives in geolexica.org
- [ ] Full pipeline: fetch → generate → edges → build → deploy
- [ ] All 4 datasets appear on www.geolexica.org
- [ ] `repository_dispatch` from vocabulary-browser triggers rebuild
