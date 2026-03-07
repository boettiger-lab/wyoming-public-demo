# NLCD Categorical Legend Implementation Plan

**Date:** 2026-03-07
**Status:** Implemented
**Branch:** fix/menu-header-title → merged to main
**Repos involved:** `wyoming-public-demo`, `geo-agent`

---

## Problem

The NLCD 2024 Land Cover layer is a **discrete categorical** raster (20 integer class codes,
values 11–95) but was configured and rendered as a **continuous** raster:

- `colormap: "rdylgn"` — a continuous diverging colormap stretched from 11→95 (meaningless)
- `rescale: "11,95"` — stretches the colormap across class code numbers, not data ranges
- Legend showed a gradient bar labeled "11 NLCD class … 95 NLCD class" — no class names

Sagebrush cover and annual grass (0–100% cover) work well with the continuous legend.
NLCD does not.

---

## Solution Overview

1. **STAC collection** — Add proper raster band documentation per STAC raster + classification
   extensions: spatial resolution, sampling, scale/offset, and canonical NLCD colors via
   `classification:classes` (with `color_hint`).
2. **`layers-input.json`** — Switch NLCD to `colormap: "nlcd"` (rio-tiler built-in),
   remove `rescale`, add `legend_type: "categorical"`.
3. **`dataset-catalog.js`** — Extract `classification:classes` from STAC raster bands and
   pass through `legendType` + `legendClasses` to the layer config pipeline.
4. **`map-manager.js`** — Branch on `legendType === "categorical"` to render a discrete
   color-swatch list instead of a gradient bar.
5. **`style.css`** — Reuse existing `.legend-item` / `.legend-item span` styles (already
   present); no new CSS classes needed.

---

## 1. STAC Collection Update (`nrp:public-wyoming/nlcd-2024/stac-collection.json`)

### 1a. Add extensions

Add the STAC classification extension to `stac_extensions`:

```json
"stac_extensions": [
    "https://stac-extensions.github.io/raster/v1.1.0/schema.json",
    "https://stac-extensions.github.io/eo/v1.1.0/schema.json",
    "https://stac-extensions.github.io/classification/v2.0.0/schema.json"
]
```

### 1b. Updated `raster:bands` entry

Replace the existing `raster:bands[0]` object with a fully-documented entry:

```json
{
    "name": "land_cover_class",
    "description": "NLCD land cover class code. Integer values correspond to the 20-class Anderson Level II classification scheme. See classification:classes for class definitions and display colors.",
    "data_type": "uint8",
    "nodata": 0,
    "sampling": "area",
    "spatial_resolution": 30,
    "scale": 1.0,
    "offset": 0.0,
    "statistics": {
        "minimum": 11,
        "maximum": 95
    },
    "classification:classes": [
        { "value": 11, "name": "Open Water",                      "description": "Areas of open water, generally with less than 25% cover of vegetation or soil.",                                                          "color_hint": "466B9F" },
        { "value": 12, "name": "Perennial Ice/Snow",               "description": "Areas characterized by a perennial cover of ice and/or snow, generally visible for two or more months per year.",                          "color_hint": "D1DEF8" },
        { "value": 21, "name": "Developed, Open Space",            "description": "Mix of constructed materials and vegetation, less than 20% impervious surface. Includes large-lot single-family housing, parks, golf courses.", "color_hint": "DEC5C5" },
        { "value": 22, "name": "Developed, Low Intensity",         "description": "Predominantly single-family housing units. Impervious surface accounts for 20–49% of total cover.",                                          "color_hint": "D99282" },
        { "value": 23, "name": "Developed, Medium Intensity",      "description": "Predominantly single-family housing units, row houses, and manufactured housing. Impervious surface 50–79%.",                              "color_hint": "EB0000" },
        { "value": 24, "name": "Developed, High Intensity",        "description": "Highly developed areas where people reside or work in high numbers (apartments, commercial, industrial). Impervious surface 80–100%.",        "color_hint": "AB0000" },
        { "value": 31, "name": "Barren Land",                      "description": "Areas of bedrock, desert pavement, scarps, talus, slides, volcanic material, glacial debris, sand dunes, strip mines, gravel pits. Vegetation generally less than 15%.", "color_hint": "B3AC9F" },
        { "value": 41, "name": "Deciduous Forest",                 "description": "Areas dominated by trees generally greater than 5 meters tall. Greater than 75% of tree species shed foliage simultaneously in response to seasonal change.", "color_hint": "68AB5F" },
        { "value": 42, "name": "Evergreen Forest",                 "description": "Areas dominated by trees generally greater than 5 meters tall. Greater than 75% of tree species maintain their leaves year-round.",            "color_hint": "1C5F2C" },
        { "value": 43, "name": "Mixed Forest",                     "description": "Areas dominated by trees generally greater than 5 meters tall. Neither deciduous nor evergreen species exceed 75% of total tree cover.",       "color_hint": "B5C58F" },
        { "value": 51, "name": "Dwarf Scrub",                      "description": "Alaska only. Low-growing upright or matted shrubs less than 20 centimeters tall with shrub canopy typically greater than 20%.",               "color_hint": "CCB879" },
        { "value": 52, "name": "Shrub/Scrub",                      "description": "Areas dominated by shrubs less than 5 meters tall with shrub canopy typically greater than 20%; true shrubs, young trees in early successional stages, or species stunted from environmental conditions.", "color_hint": "CCBA7C" },
        { "value": 71, "name": "Grassland/Herbaceous",             "description": "Areas dominated by upland grasses and forbs. Less than 10% of total vegetation is trees or shrubs. Greater than 80% of total vegetation is herbaceous.", "color_hint": "FFD3AB" },
        { "value": 72, "name": "Sedge/Herbaceous",                 "description": "Alaska only. Dominated by sedges and forbs, with greater than 80% of total vegetation cover.",                                               "color_hint": "FDE9AA" },
        { "value": 73, "name": "Lichens",                          "description": "Alaska only. Dominated by fruticose or foliose lichens generally greater than 80% of total vegetation.",                                       "color_hint": "D1D182" },
        { "value": 74, "name": "Moss",                             "description": "Alaska only. Dominated by mosses generally greater than 80% of total vegetation.",                                                             "color_hint": "A3CC51" },
        { "value": 81, "name": "Pasture/Hay",                      "description": "Areas of grasses, legumes, or grass-legume mixtures planted for livestock grazing or the production of seed or hay crops.",                   "color_hint": "DCD93D" },
        { "value": 82, "name": "Cultivated Crops",                 "description": "Areas used for the production of annual crops such as corn, soybeans, vegetables, tobacco, and cotton, as well as perennial woody crops.",     "color_hint": "AB6C28" },
        { "value": 90, "name": "Woody Wetlands",                   "description": "Areas where forest or shrubland vegetation accounts for greater than 20% of vegetative cover and the soil or substrate is periodically saturated with or covered with water.", "color_hint": "B8D9EB" },
        { "value": 95, "name": "Emergent Herbaceous Wetlands",     "description": "Areas dominated by persistent emergent herbaceous vegetation with greater than 80% of total vegetation cover. Substrate periodically saturated or covered with water.", "color_hint": "6C9FB8" }
    ]
}
```

**Field notes:**
- `sampling: "area"` — each pixel represents an area (standard for classified rasters)
- `spatial_resolution: 30` — native NLCD resolution in meters (USGS source is 30m Albers;
  this COG is reprojected to WGS84 but spatial_resolution records the meaningful native res)
- `scale: 1.0` / `offset: 0.0` — pixel values are raw integer class codes, no scaling applied
- `classification:classes` follows the STAC classification extension v2.0.0 spec;
  `color_hint` is hex color without `#` prefix (per spec)
- Old `class_values` array and `unit: "categorical"` are removed; classification extension
  supersedes these

Upload command:
```bash
echo '<updated json>' | rclone rcat nrp:public-wyoming/nlcd-2024/stac-collection.json
# or
rclone copyto /tmp/stac-collection.json nrp:public-wyoming/nlcd-2024/stac-collection.json
```

---

## 2. `layers-input.json` (wyoming-public-demo)

Change the `nlcd-2024-cog` asset config from:
```json
{
    "id": "nlcd-2024-cog",
    "display_name": "NLCD 2024 Land Cover",
    "group": "Land Cover",
    "colormap": "rdylgn",
    "rescale": "11,95",
    "legend_label": "NLCD class"
}
```
To:
```json
{
    "id": "nlcd-2024-cog",
    "display_name": "NLCD 2024 Land Cover",
    "group": "Land Cover",
    "colormap": "nlcd",
    "legend_type": "categorical"
}
```

**Notes:**
- `colormap: "nlcd"` — rio-tiler ships this built-in colormap mapping each NLCD integer code
  to its official MRLC color. titiler exposes it as `?colormap_name=nlcd`.
- No `rescale` — categorical colormaps do not use linear rescaling.
- `legend_type: "categorical"` — new field read by `dataset-catalog.js` to signal discrete
  legend rendering.
- `legend_label` removed (not meaningful for categorical).

---

## 3. `geo-agent/app/dataset-catalog.js`

### 3a. `processCollection()` — geotiff branch (~line 195)

After the existing fields, add extraction of legend metadata from STAC:

```js
// Extract categorical legend classes from STAC raster band metadata
const band0 = asset['raster:bands']?.[0];
const legendClasses = band0?.['classification:classes'] || null;
```

Add to the `layers.push({...})` call:
```js
legendType: config.legend_type || null,
legendClasses: legendClasses,
```

### 3b. `getLayerConfigs()` — raster branch (~line 454)

Add to the `configs.push({...})` call:
```js
legendType: ml.legendType,
legendClasses: ml.legendClasses,
```

No change to tile URL construction — the `rescale` append is already conditional
(`if (ml.rescale)`) so omitting `rescale` in the config is sufficient.

---

## 4. `geo-agent/app/map-manager.js`

### 4a. `registerLayer()` — destructuring (~line 136)

Add `legendType` and `legendClasses` to the destructured config:
```js
const { ..., legendLabel, legendType, legendClasses } = config;
```

Add to `this.layers.set(layerId, {...})` (~line 192):
```js
legendType: legendType || null,
legendClasses: legendClasses || null,
```

### 4b. `_showRasterLegend()` — branch on legend type (~line 548)

After the early-return guard and `_ensureLegend()` call, replace the single rendering path
with a branch:

```js
async _showRasterLegend(layerId) {
    const state = this.layers.get(layerId);
    if (!state) return;

    this._ensureLegend();
    this._legendEl.style.display = '';

    if (this._legendItems.has(layerId)) {
        this._legendItems.get(layerId).style.display = '';
        return;
    }

    const item = document.createElement('div');
    item.className = 'legend-section';

    if (state.legendType === 'categorical' && state.legendClasses?.length) {
        // Discrete class list
        const rows = state.legendClasses.map(cls => {
            const color = cls.color_hint ? `#${cls.color_hint}` : '#888';
            const label = cls.name || `Class ${cls.value}`;
            return `<div class="legend-item">
                        <span style="background:${color};"></span>${label}
                    </div>`;
        }).join('');
        item.innerHTML = `<h4>${state.displayName}</h4>${rows}`;
    } else {
        // Continuous gradient (existing logic)
        const gradient = await this._getColormapGradient(state.colormap || 'reds');
        const [minVal, maxVal] = (state.rescale || '0,1').split(',');
        const unit = state.legendLabel ? ` ${state.legendLabel}` : '';
        item.innerHTML = `
            <h4>${state.displayName}</h4>
            <div class="legend-colorbar" style="background: ${gradient};"></div>
            <div class="legend-labels">
                <span>${minVal}${unit}</span>
                <span>${maxVal}${unit}</span>
            </div>
        `;
    }

    this._legendContent.appendChild(item);
    this._legendItems.set(layerId, item);
}
```

**Note:** The existing `.legend-item` and `.legend-item span` CSS in `style.css` (lines 314–335)
already provides the swatch+label layout (flex row, 18×18px colored square, 8px gap).
No new CSS is needed.

---

## 5. `geo-agent/app/style.css`

No changes required. The existing `.legend-item` / `.legend-item span` styles handle the
categorical swatch rendering:

```css
.legend-item {
    display: flex; align-items: center; font-size: 11px; color: #333;
    margin-bottom: 5px; line-height: 1.3;
}
.legend-item span {
    display: inline-block; width: 18px; height: 18px;
    margin-right: 8px; border-radius: 2px;
    border: 1px solid rgba(0,0,0,0.2); flex-shrink: 0;
}
```

With 20 NLCD classes the legend section will be ~200px tall; `#legend-content` already has
`max-height: calc(80vh - 60px)` and `overflow-y: auto` so it scrolls cleanly.

---

## Testing Checklist

- [ ] STAC JSON validates against raster extension schema
- [ ] STAC JSON validates against classification extension schema
- [ ] `titiler.nrp-nautilus.io/colorMaps/nlcd` returns a valid colormap response
- [ ] NLCD layer renders on map with correct colors (water=blue, forest=green, etc.)
- [ ] Legend shows 20 colored swatches with class names
- [ ] Legend scrolls within the panel (no overflow)
- [ ] Continuous-gradient layers (RAP, sagebrush) unaffected
- [ ] Works in wyoming-public-demo; also verify no regression in example-ghpages / example-k8s

---

## Files Changed

| File | Repo | Change |
|---|---|---|
| `nlcd-2024/stac-collection.json` | S3 / NRP | Add classification extension + full band metadata |
| `layers-input.json` | wyoming-public-demo | `colormap: "nlcd"`, `legend_type: "categorical"`, remove rescale/legend_label |
| `app/dataset-catalog.js` | geo-agent | Extract + pass `legendType`, `legendClasses` |
| `app/map-manager.js` | geo-agent | Categorical branch in `_showRasterLegend` |
| `app/style.css` | geo-agent | No changes (existing styles sufficient) |

---

## NLCD Color Reference (official MRLC canonical colors)

| Code | Name | Hex |
|------|------|-----|
| 11 | Open Water | `#466B9F` |
| 12 | Perennial Ice/Snow | `#D1DEF8` |
| 21 | Developed, Open Space | `#DEC5C5` |
| 22 | Developed, Low Intensity | `#D99282` |
| 23 | Developed, Medium Intensity | `#EB0000` |
| 24 | Developed, High Intensity | `#AB0000` |
| 31 | Barren Land | `#B3AC9F` |
| 41 | Deciduous Forest | `#68AB5F` |
| 42 | Evergreen Forest | `#1C5F2C` |
| 43 | Mixed Forest | `#B5C58F` |
| 51 | Dwarf Scrub (Alaska only) | `#CCB879` |
| 52 | Shrub/Scrub | `#CCBA7C` |
| 71 | Grassland/Herbaceous | `#FFD3AB` |
| 72 | Sedge/Herbaceous (Alaska only) | `#FDE9AA` |
| 73 | Lichens (Alaska only) | `#D1D182` |
| 74 | Moss (Alaska only) | `#A3CC51` |
| 81 | Pasture/Hay | `#DCD93D` |
| 82 | Cultivated Crops | `#AB6C28` |
| 90 | Woody Wetlands | `#B8D9EB` |
| 95 | Emergent Herbaceous Wetlands | `#6C9FB8` |

Source: https://www.mrlc.gov/data/legends/national-land-cover-database-class-legend-and-description
