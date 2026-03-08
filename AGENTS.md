# Agent Guidelines — wyoming-public-demo

## Deployment workflow

The app runs on Kubernetes (NRP Nautilus, namespace `biodiversity`). The deployment's init container clones the GitHub repo at pod startup:

```
git clone --depth 1 https://github.com/boettiger-lab/wyoming-public-demo.git
```

**This means any file change must be committed and pushed to GitHub BEFORE restarting the deployment.** A rollout restart that precedes the push will pick up stale code.

### Required order of operations

1. Edit files locally
2. `git commit` the changes
3. `git push` to GitHub
4. `kubectl -n biodiversity rollout restart deployment/wyoming-public-demo`
5. Confirm with `kubectl -n biodiversity rollout status deployment/wyoming-public-demo`

## Files served by the app

The init container copies these three files from the repo root into nginx:

- `index.html` — page structure and MapLibre bootstrap
- `layers-input.json` — layer definitions, color schemes, STAC collection IDs
- `system-prompt.md` — LLM system prompt for the data assistant

Changes to any other file in the repo have no effect on the running app.

## STAC collections and S3

Layer metadata (titles, descriptions, class colors, citations) lives in STAC collection JSON files on S3:

- Bucket: `public-wyoming` at `s3-west.nrp-nautilus.io`
- Example: `s3-west.nrp-nautilus.io/public-wyoming/sagebrush-design/stac-collection.json`

Upload updates with rclone (remote named `nrp`):

```bash
rclone copyto <local-file>.json nrp:public-wyoming/<path>/stac-collection.json
```

STAC changes take effect immediately (no deployment restart needed) — but the app fetches STAC at page load, so users must hard-refresh.

## Categorical raster layers

To display a raster COG with a discrete color legend (like NLCD or sagebrush), use this pattern:

**`layers-input.json`** — set `legend_type: "categorical"` (no `colormap` or `rescale`):

```json
{
  "id": "my-cog",
  "display_name": "My Layer",
  "group": "Land Cover",
  "legend_type": "categorical"
}
```

**STAC collection** — add the classification extension and `classification:classes` with `color_hint` (6-char hex, no `#`):

```json
{
  "stac_extensions": [
    "https://stac-extensions.github.io/raster/v1.1.0/schema.json",
    "https://stac-extensions.github.io/classification/v2.0.0/schema.json"
  ],
  "assets": {
    "cog": {
      "raster:bands": [{
        "classification:classes": [
          { "value": 1, "name": "Class A", "description": "...", "color_hint": "1A7837" },
          { "value": 2, "name": "Class B", "description": "...", "color_hint": "F5B842" }
        ]
      }]
    }
  }
}
```

The app reads `classification:classes` from the STAC to build the legend and titiler colormap request.
