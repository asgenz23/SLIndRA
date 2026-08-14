---
name: cindra-sea-level-governance
description: "Use for global CIndRA sea-level MVP rules: scope, source hierarchy, controlled parameters, product profiles, provenance, approved figure policy, review labels, and unsupported-request handling. Use before domain workflow skills when a request needs routing or policy interpretation."
---

# CIndRA Sea-Level Governance

## Purpose

Apply this skill for global rules that govern all CIndRA Phase 1 sea-level products. This skill controls scope, product profiles, data-source hierarchy, provenance, figure policy, and review status. Domain workflow skills must not override these rules.

## Product Profiles

Active sea-level product profiles are:

- `Regional`
- `National/EEZ`

Both profiles share scientific calculations, data-source hierarchy, QC rules, method IDs, provenance schema, and validation categories. Profiles differ only in spatial scope, aggregation/report layout, table granularity, required report-facing inventory, captions, limitations, and validation checks.

## Approved Data Sources

### Tide gauges

Use **UHSLC** as the working authoritative source for station metadata, station-relative trends, and minor flood frequency. NOAA CO-OPS may be used for U.S.-affiliated station context, datum/threshold comparison, or explicit NOAA API requests, but must not replace UHSLC unless approved.

### Altimetry

Use the project-provided CMEMS/Copernicus monthly `0.25 degree` gridded altimetry NetCDF as the working authoritative source for CIndRA altimetry products:

- URL: `https://uhslc.soest.hawaii.edu/mwidlans/dev/SEA/SEAdata/cmems_altimetry.nc`
- Format: NetCDF
- Cadence: monthly
- Resolution: `0.25 degree`

Do not substitute another altimetry product unless explicitly requested and labeled as a method update or exploratory sensitivity test.

### Boundaries

For `National/EEZ` products, use project-approved national or EEZ boundaries where available. Preferred hierarchy:

1. Project-approved boundary from site setup.
2. Approved Pacific/SPREP or Pacific Data Hub boundary.
3. Marine Regions / VLIZ EEZ boundary.
4. Documented EEZ-scale bounding box for `Draft / Experimental` diagnostics only.

Record boundary name, source, URL/path, version/date, geometry type, bounding box, use, fallback rationale, and limitations.

## Mandatory Quality Control Workflow

Quality Control (using `cindra-quality-control` skill) is a **required workflow step** for all CIndRA sea-level products. The QC gate must be applied after site setup and before proceeding to trend, flood-frequency, or product assembly analysis.

**Mandatory sequence**:
1. Site Setup → Create and validate site configuration
2. **Quality Control** → Evaluate station-level Level 2 completeness gate
3. Domain Workflows → Trend and/or flood-frequency analysis (if QC passes)
4. Product Assembly → Package outputs for review

Station-level Level 2 gate is a **blocking gate**, not optional filtering. If a station fails the gate, choose alternatives (narrower period, different station, or explicitly labeled exploratory sensitivity product).

## Temporal Aggregation and Data Completeness Rules

### Common Thresholds and Conventions

#### Storm Year

- **Definition**: May 1 – April 30 (inclusive)
- **Labeling**: Label by starting year (e.g., storm year 2020 = May 1, 2020 – April 30, 2021)
- **Completeness Rule**: Storm year passes if usable-day coverage ≥ 75% of expected days
- **Diagnostic Flags**: Month-level failures carry forward as diagnostics, not automatic exclusions

#### Calendar Year

- **Definition**: January 1 – December 31 (inclusive)
- **Completeness Rule**: Calendar year passes if usable-day coverage ≥ 75% of expected days
- **Missing Year Definition**: Zero usable daily values = missing year (not partial year below 75%)

#### Monthly

- **Completeness Rule**: Month passes if usable-day coverage ≥ 75% AND maximum consecutive missing days ≤ 7
- **Failed Days**: Record and carry forward; do not silently discard

#### No-Data Preservation

- **True no-data periods**: Represent as missing/empty/`NaN` — never convert to valid zero-count periods
- **Partial-data periods**: Carry forward with QC diagnostics; do not exclude silently
- **Rationale**: Preserves interpretability and enables downstream validation of data availability

### Reference Documentation

For detailed daily-interval rules, six-hour windows, and Level 2 station-level gates, see **`cindra-quality-control` § Daily Rule, Monthly Rule, Storm-Year Rule, Calendar-Year Rule, Station-Level Level 2 Gate**.

## UHSLC Coverage Source Rule

For station inventories, use this hierarchy:

1. UHSLC Fast Delivery / hybrid coverage where available.
2. If unavailable, UHSLC Research Quality data.
3. For Research Quality, inspect versions and use only the most recent version.
4. Do not combine Research Quality versions.
5. Record available versions, selected version, and coverage status in provenance.

## Provenance Requirements

For every output table, metric, figure, map, or narrative claim, preserve or report dataset name, provider, access path/repository, version, access date, DOI/citation, station/source ID, variable names, temporal resolution, units, datum/reference, spatial extraction method, source priority, and source status.

Machine-readable backbone files are the source of record, including product inventory, parameters, source versions, method versions, provenance, validation checklist, and issue log. PDF/DOCX/Markdown report files are assemblies only.

## Scientific Distinctions

Keep tide-gauge and altimetry products distinct:

- Tide gauges measure **relative sea level** at a station.
- Satellite altimetry measures **absolute sea level** in a geocentric frame.
- Combined displays may show both but must not merge them into a single metric unless an approved method exists.

Do not average tide-gauge stations without a documented aggregation rule.

## Figure Policy

Final CIndRA figures must be produced by approved code in `PICCM_SeaLevel`, especially helpers in `functions/sea_level_plotting.py`, or by a new helper first added to that module. Do not create final CIndRA figures using ad hoc inline plotting code. If a requested figure is unsupported, propose a new plotting helper with name, inputs, outputs, and method purpose.

## Approved Plotting Helpers Reference

### Canonical Helper Locations

- **`functions/sea_level_plotting.py`** — Core helpers for trend, flood-frequency, anomaly, and rankings workflows (Production-ready)
- **`functions/cindra_regional_plotting_helpers.py`** — Regional-domain specialized helpers (Draft/Experimental status)

### By Workflow Category

#### Trend Analysis Helpers

- `plot_magnitude_map` — Combined absolute altimetry and relative tide-gauge sea-level change magnitude map
- `plot_magnitude_map_background` — Background-only absolute altimetry magnitude map
- `plot_altimetry_trend_timeseries` — Absolute altimetry trend time series with linear fit
- `plot_tide_gauge_trend_timeseries` — Relative tide-gauge trend time series with linear fit
- `plot_combined_trends` — Side-by-side comparison of altimetry and tide-gauge trends
- `plot_enso_scatter` — ENSO index versus sea-level anomaly scatter plot with regression
- `plot_combined_altimetry_tide_gauge_trend_map` — Combined map for National/EEZ profile (same as `plot_magnitude_map`)
- `plot_national_eez_combined_trend_map` — National/EEZ combined trend map (see TR04 consolidated product)
- `plot_regional_altimetry_trend_map_filled_tide_gauges` — Regional absolute altimetry trend map with relative tide-gauge station markers (Draft)

#### Flood-Frequency Helpers

- `plot_histogram_with_threshold` — Hourly water-level histogram with 30 cm MHHW threshold line
- `plot_flood_counts_with_trend` — Annual flood-day counts with trend line and p-value
- `plot_flood_counts_with_oni` — Annual flood-day counts colored by ENSO phase
- `plot_flood_days_heatmap` — Storm-year by month heatmap of flood-day counts
- `plot_flood_matrix_summary` — Composite flood matrix with annual counts and monthly percent contribution (see FF04 consolidated product)
- `plot_oni_only` — ENSO phase bar chart for context only
- `plot_monthly_contribution_vertical` — Monthly flood-day count or percent-contribution vertical bar chart
- `plot_regional_flood_frequency_overview` — Regional station-by-year flood-day matrix with annual totals (Draft)

#### Anomaly and Decadal Analysis Helpers

- `plot_tg_rsl_anomaly_annual` — Relative tide-gauge sea-level anomaly by year
- `plot_anomaly_decadal_maps` — Gridded anomaly maps for decadal periods
- `plot_anomaly_station_series` — Station anomaly time series with ENSO context

#### Rankings and Extremes Helpers

- `make_plotly_figure_rankings` — Interactive Plotly rankings visualization
- `make_rankings_static_figure` — Static rankings figure
- `plot_simple_timeseries` — Basic time series plot
- `plot_daily_max_timeseries` — Daily maximum water level time series
- `plot_annual_range_fill` — Annual range (min/max) with mean fill

### Requesting New Helpers

If a required figure is not listed above:

1. Propose the new helper with name, inputs, outputs, and method purpose
2. Add it to `functions/sea_level_plotting.py` or `functions/cindra_regional_plotting_helpers.py`
3. Update this reference section with category and description
4. Mark as Draft/Experimental if not yet validated

## Review Status

Use these labels only:

- `Experimental`
- `Draft`
- `Scientist-reviewed`
- `Approved for report use`
- `Deferred`
- `Optional`

Do not mark products `Approved for report use` without human scientific review and complete validation/provenance records.

## Code Repository Furnishing Policy

Do not furnish a `Code repository/` bundle by default. Preserve code/repository reference, method IDs, source files, run record, and provenance by default. Furnish annotated notebooks, product-code crosswalks, and code bundle contents only when explicitly requested or contractually required. If not requested, record code bundle status as `not_furnished_unless_requested`, not as missing or failed.

## Boundary-Dependent Combined Trend Maps — Consolidated Update

For National/EEZ trend maps that combine gridded altimetry and tide-gauge markers, use the approved boundary hierarchy. Marine Regions / VLIZ EEZ boundaries may be used as hierarchy level 3 when higher-priority project-approved or Pacific regional boundaries are not furnished. Preserve provider, layer/version, URL/path, identifier such as MRGID, geometry type, bounding box, access date, source priority, and source status.

Do not fabricate EEZ boundaries. If no approved boundary is available, mark boundary-dependent map products as `Deferred`. Bounding boxes may be used only for clearly labeled Draft/Experimental diagnostics, not as final National/EEZ map boundaries.
