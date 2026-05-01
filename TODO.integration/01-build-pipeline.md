# 01 — Build Pipeline: Use datasets.yml from this repo

## Goal

The geolexica.org repository owns `datasets.yml` (the dataset registry) and uses the vocabulary-browser to build and deploy www.geolexica.org.

## What Changed

### Moved datasets.yml here
- `datasets.yml` is deployment configuration — it belongs in the deployment repo
- The vocabulary-browser copies it at build time

### Updated build_deploy.yml
- Check out THIS repo first (for datasets.yml)
- Then check out vocabulary-browser (still source checkout for now)
- Copy datasets.yml into vocabulary-browser before running
- Removed `build-gcr:all` step (GCR packages are pre-built by glossary repos)
- Fetch step downloads pre-built GCR packages from GitHub releases

### Remaining (future work)
- Switch from vocabulary-browser source checkout to `npx glossarist-vocabulary-browser`
- Add `package.json` to this repo for dependency tracking
- Remove vocabulary-browser checkout entirely

## Acceptance Criteria

- [x] `datasets.yml` lives in geolexica.org
- [x] No `build-gcr:all` step (GCR packages are pre-built)
- [ ] Full pipeline: fetch → generate → edges → build → deploy
- [ ] All 4 datasets appear on www.geolexica.org
- [ ] `repository_dispatch` from vocabulary-browser triggers rebuild
