# www.geolexica.org

Deployment configuration for [www.geolexica.org](https://www.geolexica.org), the multi-dataset hub for Geolexica terminology registers.

Built with [`@glossarist/concept-browser`](https://github.com/glossarist/concept-browser) — a Vue 3 SPA that browses ISO terminology datasets.

## Datasets

The hub hosts four datasets, configured in `datasets.yml`:

| ID | Title | Owner |
|---|---|---|
| `iev` | IEC Electropedia (IEV) | IEC TC 1 |
| `isotc211` | ISO/TC 211 Multi-Lingual Glossary | ISO/TC 211 |
| `isotc204` | ISO/TC 204 ITS Vocabulary | ISO/TC 204 |
| `osgeo` | OSGeo Lexicon | OSGeo |

## Repository Structure

```
datasets.yml        Dataset registry (GCR sources, colors, tags)
site-config.yml     Site branding, routing, and feature flags
index.html          Legacy static landing page (fallback)
.github/workflows/  CI/CD: builds and deploys to GitHub Pages
```

## Deployment

Pushing to `main` triggers the [build_deploy workflow](.github/workflows/build_deploy.yml) which:

1. Checks out this repo and `glossarist/concept-browser`
2. Copies `datasets.yml` into the browser project
3. Fetches GCR packages, generates data, builds edges
4. Builds the SPA and deploys to GitHub Pages

### Adding a Dataset

Add an entry to `datasets.yml` — no code changes required.

## Site Configuration

`site-config.yml` controls branding, fonts, features, and per-dataset overrides. See the [`concept-browser` docs](https://github.com/glossarist/concept-browser) for the full schema.

## License

MIT
