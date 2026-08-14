---
name: cindra-site-setup
description: Use when configuring or validating a CIndRA sea-level site, selecting UHSLC stations, resolving EEZ or regional spatial settings, preparing station inventories, or creating the site JSON used by downstream CIndRA skills.
---

# CIndRA Site Setup

## Purpose

Create or validate the shared CIndRA site configuration used by downstream sea-level indicator workflows. Site setup resolves active product profile, selected UHSLC station(s), station metadata, spatial scope, EEZ/regional boundary metadata, analysis date bounds, CMEMS extraction settings, and provenance paths.

Site setup may record preliminary coverage and datum availability, but station suitability decisions are controlled by `cindra-quality-control`.

## Source of Truth

- Notebook: `notebooks/historical/0_site_setup.ipynb`
- Core module: `functions/sea_level.py`
- Downloaders: `functions/data_downloaders.py`
- Pilot configuration: Palau, Malakal tide gauge, UHSLC ID `7`

## Site Configuration Fields

Include fields where relevant:

- `site_name`, `site_lon`, `site_lat`
- `product_profile`
- `site_config_path`
- `start_date`, `end_date`
- `selected_uhslc_id`, `selected_station_name`, `station_country_filter`
- `site_eez_shapefile` or `site_boundary_path`
- `site_boundary_name`, `site_boundary_source`, `site_boundary_source_url`, `site_boundary_version_or_date`
- `site_boundary_geometry_type`, `site_boundary_bbox`, `site_boundary_use`
- `site_boundary_fallback_used`, `site_boundary_fallback_rationale`
- `cmems_bbox`
- `regional_mask_id`
- `spatial_scope_description`
- `station_ids`, `included_station_list`, `excluded_station_list`
- `coverage_source_selected`, `coverage_source_status`, `coverage_source_rule`
- `rqds_versions_available`, `rqds_version_used`
- `datum_availability`, `mhhw_availability`, `qc_status_reference`

## Workflow

1. Load requested site metadata and product profile.
2. Use repository site-preparation helpers where available.
3. Verify station identity using authoritative UHSLC metadata. If station ID/name is uncertain, use IDEA station lookup before proceeding.
4. For PICCM/Pacific Islands regional products, use the CIndRA regional station mask skill.
5. Record preliminary coverage, datum, and MHHW availability.
6. Save or update the site configuration.
7. Confirm downstream paths, station metadata, date ranges, boundary metadata, and CMEMS bounds are populated.

## Regional vs. National/EEZ Product Profile Workflows

### Regional Profile (e.g., PICCM Pacific Islands)

**Use when**: Analyzing multiple stations within a defined regional station mask or geographic domain.

**Key workflow steps**:
1. **Regional Mask Definition**: Use `cindra-piccm-regional-definition` to resolve station inclusion/exclusion
2. **Site Setup**: Configure regional profile with selected stations
3. **Quality Control**: Validate each station's Level 2 suitability
4. **Domain Analysis**: 
   - Trend: Regional gridded altimetry map with station-marker overlay
   - Flood-Frequency: Regional station-by-year flood-day matrix with annual totals
5. **Product Assembly**: Regional report-facing products using regional plotting helpers

**Regional Plotting Helpers** (Draft/Experimental):
- `plot_regional_altimetry_trend_map_filled_tide_gauges` — Absolute altimetry background + relative tide-gauge station markers
- `plot_regional_flood_frequency_overview` — Station-by-year flood-day matrix + annual bar chart

**Status**: Regional helpers are Draft/Experimental (not yet production-validated). Validate with project scientists before using in published reports.

---

### National/EEZ Profile

**Use when**: Analyzing sea-level indicators for an entire country's Exclusive Economic Zone (EEZ) or approved national boundary.

**Key workflow steps**:
1. **Boundary Definition**: Use project-approved national or EEZ boundary (see Governance § Boundaries)
2. **Site Setup**: Configure National/EEZ profile with boundary metadata and selected stations
3. **Quality Control**: Validate each station's Level 2 suitability
4. **Domain Analysis**:
   - Trend: Combined magnitude map (absolute altimetry background + relative tide-gauge markers)
   - Flood-Frequency: Station-level flood-day summaries (optionally aggregated by region within EEZ)
5. **Product Assembly**: National-scope report-facing products with boundary provenance

**National/EEZ Plotting Helpers** (Production-ready):
- `plot_magnitude_map` — Combined absolute altimetry and relative tide-gauge magnitude map
- `plot_national_eez_combined_trend_map` — EEZ-scale combined trend map

**Boundary Hierarchy** (see Governance § Boundaries):
1. Project-approved boundary from site setup
2. Pacific/SPREP or Pacific Data Hub boundary
3. Marine Regions / VLIZ EEZ boundary (e.g., MRGID 8315 for Palau)
4. Bounding box (Draft/Experimental only; do not use for final products)

---

### Workflow Comparison

| Aspect | Regional | National/EEZ |
|--------|----------|----------------|
| **Scope** | Station-level domain (e.g., island group) | Country/EEZ boundary |
| **Stations** | Regional mask of selected stations | All available QC-passed stations in EEZ |
| **Boundary** | Station list (not polygonal) | Approved polygonal EEZ/national boundary |
| **Trend Map** | Altimetry grid + station markers | Combined magnitude map (altimetry + stations) |
| **Flood-Freq Product** | Station-by-year matrix | Aggregated summary (may include regional subscales) |
| **Helper Status** | Draft/Experimental | Production-ready |
| **Validation** | Validate with scientists before publishing | Standard QC/provenance procedures |

---

## Downstream Workflow Triggers

After site configuration is saved and validated, use the following domain skills in sequence to generate complete sea-level indicator products:

### 1. Quality Control (Required)

**Skill**: `cindra-quality-control`

**Purpose**: Validate station suitability and identify failing years/months before proceeding to analysis.

**Input**: Site config (station ID, date range, intended method)

**Output**: QC summary with Level 2 pass/fail, missing years, completeness fractions, diagnostic flags

**When to Use**: 
- Required for all products (primary, exploratory, and diagnostic)
- Allows early exit if station fails gate
- Must complete before proceeding to trend, flood-frequency, or assembly workflows

---

### 2. Trend Analysis

**Skill**: `cindra-sea-level-trend`

**Purpose**: Compute absolute altimetry and relative tide-gauge trend analysis.

**Input**: Site config (profile, period, boundary, CMEMS bbox) + QC reference (required)

**Output**: Trend tables (station_trend_summary.csv, altimetry_trend_summary.csv, integrated_trend_comparison.csv); trend maps; time-series plots

**When to Use**: After site setup and required QC; required for all trend products and combined magnitude maps

---

### 3. Flood-Frequency Analysis

**Skill**: `cindra-flood-frequency`

**Purpose**: Compute daily maximum water-level exceedances and storm-year aggregation.

**Input**: Site config (station ID, period) + QC reference (required Level 2 gate status)

**Output**: flood_frequency_summary.csv; storm-year and monthly flood-day tables; flood-frequency plots

**When to Use**: After site setup and required QC; independent of trend analysis (can run in parallel)

---

### 4. Product Assembly

**Skill**: `cindra-product-assembly`

**Purpose**: Package outputs into report-ready products with captions, methods, provenance, and validation.

**Input**: Site config + all domain skill outputs (trend, flood-frequency, QC diagnostics, figures, tables)

**Output**: Product inventory; captions; methods/limitations notes; validation checklist; issue log; report sections

**When to Use**: After all domain skill outputs are generated; final step before review/publication

---

### Workflow Diagram

```
┌─────────────────────────────────────────────────────┐
│  Site Setup (Workflow 1): Create site_config.json  │
└──────────────────┬──────────────────────────────────┘
                   │
         ┌─────────▼──────────────────────┐
         │   Quality Control (required)   │
         │   (Workflow 2)                 │
         └─────────┬──────────────────────┘
                   │
        ┌──────────┴──────────┐
        │                     │
   ┌────▼──────────┐    ┌────▼──────────────┐
   │  Trend        │    │ Flood-Frequency   │
   │(Workflow 3)   │    │ (Workflow 4)      │
   └────┬──────────┘    └────┬──────────────┘
        │                    │
        └──────────┬─────────┘
                   │
         ┌─────────▼──────────────────┐
         │ Product Assembly (Workflow 5)│
         └────────────────────────────┘

* Domain workflows (3, 4) can run in parallel after QC (required)
```

## Validation

Confirm station identity, active product profile, spatial scope, target trend period `1993-2025`, and boundary availability. For Palau validation, confirm Malakal, Palau, UHSLC ID `7`. Regional station-mask inclusion is not QC approval.

## Boundary Metadata for Combined Trend Maps — Consolidated Update

When a site will produce `TR04_magnitude_map_altimetry_tide_gauge`, populate boundary metadata fields sufficient for downstream provenance:

- `site_boundary_name`
- `site_boundary_source`
- `site_boundary_source_url`
- `site_boundary_version_or_date`
- `site_boundary_geometry_type`
- `site_boundary_bbox`
- `site_boundary_identifier` such as Marine Regions `MRGID`
- `site_boundary_use`
- `site_boundary_fallback_used`
- `site_boundary_fallback_rationale`

For Palau, Marine Regions / VLIZ `Palauan Exclusive Economic Zone`, MRGID `8315`, can be used as hierarchy level 3 if no higher-priority approved boundary is furnished.
