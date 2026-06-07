# Building Coverage Ratio (BCR) Analysis
**Author:** Martins
**Date:** May 11, 2026
**Software:** QGIS (EPSG:2227 — NAD83 / California Zone 3, US Feet)
**Project File:** `BCR_Analysis.qgz`

---

## 1. Project Overview

This project computes the **Building Coverage Ratio (BCR)** for every land parcel
within the study area by intersecting OSM building footprints against the cadastral
parcel layer. BCR is a fundamental urban morphology metric expressing what fraction
of each parcel's land area is physically covered by a building footprint.

BCR is used in planning to:
- Identify vacant and underutilised land within built-up areas
- Assess development intensity relative to zoning designation
- Establish baseline conditions for redevelopment feasibility studies
- Validate or replace zoning-proxy development filters in suitability analyses

---

## 2. Input Data

| Layer | Source | Features | Geometry | CRS |
|-------|--------|----------|----------|-----|
| `parcels` | Original project layer | 1,904 | Polygon | EPSG:2227 |
| `building_footprints` | OpenStreetMap via QuickOSM | 1,974 | Polygon | EPSG:2227 |
| `boundary` | Original project layer | 1 | Polygon | EPSG:2227 |

Building footprints were fetched from the Overpass API using the QuickOSM QGIS
plugin, reprojected from EPSG:4326 to EPSG:2227, and clipped to the study boundary
prior to this analysis.

---

## 3. Methodology

### 3.1 Spatial Intersection

Building footprint polygons are intersected with parcel polygons using
`native:intersection`. This splits any building that straddles multiple parcels
at the parcel boundary, attributing the correct fragment area to each parcel.
The intersection produced **4,187 pieces** from 1,974 input buildings across
1,904 parcels.

### 3.2 Building Area Aggregation

For each parcel (identified by `blklot`), the geometry areas of all intersection
pieces are summed in EPSG:2227 (US feet) to produce `bldg_sqft` — the total
building footprint area within that parcel.

### 3.3 BCR Calculation

```
BCR (%) = (bldg_sqft / parcel_sqft) × 100
```

BCR is capped at 100% to absorb minor geometric slivers from the intersection.
Parcels with no intersecting building receive `bldg_sqft = 0` and `BCR = 0`.

### 3.4 Tier Classification

| Tier | BCR Range | Count | % of Parcels |
|------|-----------|-------|--------------|
| ⬜ Vacant | 0% | 22 | 1.2% |
| 🟩 Underdeveloped | 0% < BCR ≤ 25% | 47 | 2.5% |
| 🟢 Moderate | 25% < BCR ≤ 50% | 196 | 10.3% |
| 🌲 Dense | 50% < BCR ≤ 75% | 868 | 45.6% |
| 🟫 Saturated | BCR > 75% | 771 | 40.5% |

---

## 4. Key Statistics

| Metric | Value |
|--------|-------|
| Total parcels analysed | 1,904 |
| Parcels with ≥ 1 building | 1,882 (98.8%) |
| Vacant parcels (BCR = 0) | 22 (1.2%) |
| BCR range | 0.0% – 100.0% |
| Mean BCR | 68.7% |
| Median BCR | 70.7% |

The high mean and median BCR (≈ 69–71%) confirms the study area is a dense,
mature urban neighbourhood typical of San Francisco's Hayes Valley / Downtown
corridor, with very little undeveloped land remaining.

---

## 5. Output Fields

| Field | Type | Description |
|-------|------|-------------|
| `blklot` | String | Block-lot parcel identifier |
| `street` | String | Street name |
| `zoning_sim` | String | Simplified zoning code |
| `districtna` | String | Planning district name |
| `parcel_sqft` | Double | Parcel area (sq ft, EPSG:2227) |
| `bldg_sqft` | Double | Building footprint area within parcel (sq ft) |
| `bcr_pct` | Double | Building coverage ratio (%) |
| `bcr_tier` | String | Tier: Vacant / Underdeveloped / Moderate / Dense / Saturated |

---

## 6. Output Files

```
Floor Area Ratio (FAR) Estimation/
├── BCR_Analysis.qgz              ← QGIS project (open this)
├── BCR_Analysis.gpkg
│   ├── bcr_parcels               ← BCR-scored parcels (primary output)
│   ├── building_footprints       ← OSM building polygons (clipped, EPSG:2227)
│   ├── boundary                  ← Study area boundary
│   └── parcels                   ← Original cadastral parcels
└── README.md
```

---

## 7. Map Symbology

| Tier | Colour | Hex |
|------|--------|-----|
| Vacant | White | `#f7f7f7` |
| Underdeveloped | Light green | `#c7e9c0` |
| Moderate | Medium green | `#74c476` |
| Dense | Dark green | `#238b45` |
| Saturated | Deep green | `#00441b` |

Building footprints are overlaid in mid-grey (`#bdbdbd`) to show the physical
building stock against the parcel BCR classification.

---

## 8. Limitations

- **OSM completeness** — OSM building coverage in San Francisco is generally
  high but not exhaustive. Some buildings may be missing or have imprecise
  geometries, slightly underestimating BCR for affected parcels.
- **Building:levels not used** — this analysis measures footprint coverage only,
  not volumetric development intensity. A single-storey warehouse and a
  10-storey office block with the same footprint receive identical BCR scores.
- **Parcel geometry slivers** — very small intersection slivers may inflate BCR
  marginally beyond the true coverage; capped at 100%.

---

## 9. Recommended Next Steps

- **Combine BCR with FAR** — multiply `bldg_sqft` by `building:levels` to
  estimate total floor area and compute Floor Area Ratio (FAR = floor area /
  parcel area). This captures volumetric intensity, not just footprint coverage.
- **Overlay with zoning** — parcels with `Vacant` or `Underdeveloped` BCR in
  high-intensity zones (C-3, NCT) are the strongest redevelopment candidates.
- **Replace suitability v2 development filter** — substitute the RH-zoning proxy
  used in the v2 suitability analysis with this geometry-derived BCR flag for
  a more accurate built-out classification.
- **Track change over time** — repeat this analysis with a future building
  dataset to measure densification rates across the study area.

---

*Generated with QGIS MCP Plugin + Claude (Anthropic) — May 11, 2026*
