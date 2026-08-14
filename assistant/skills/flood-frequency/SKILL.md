---
name: cindra-flood-frequency
description: Use for CIndRA minor flood-frequency analyses using UHSLC hourly water levels, MHHW datum, 30 cm above MHHW threshold, daily maxima, storm-year flood counts, regional flood matrices, and approved flood-frequency plots.
---

# CIndRA Flood Frequency

## Purpose

Run or explain CIndRA minor sea-level flood-frequency workflows using UHSLC hourly water levels, MHHW datum, daily maxima, and May-April storm-year counts.

## Source of Truth

- Notebook: `notebooks/historical/c_sea_level_ff.ipynb`
- Core module: `functions/sea_level.py`
- Plotting module: `functions/sea_level_plotting.py`
- Downloaders: `functions/data_downloaders.py`

## Default Prototype Period

For Malakal/PICCM prototype products, use storm year `1983` through the latest complete storm year available unless the user specifies otherwise. A storm year runs May 1-April 30 and is labeled by starting year.

## Definition

Phase 1 minor flood frequency uses:

- UHSLC hourly tide-gauge water levels where available;
- MHHW datum where available and compatible;
- threshold: `30 cm above MHHW`;
- event basis: daily maximum water-level exceedance;
- aggregation: May-April storm year;
- primary metric: flood days/year.

A flood day occurs when daily maximum water level reaches or exceeds `30 cm above MHHW`. Do not calculate flood days from daily means. If MHHW is unavailable, do not fabricate datum or threshold context.

## Workflow

**Prerequisites**: Site configuration and Quality Control screening must be completed before starting flood-frequency analysis. See `cindra-quality-control` and `cindra-site-setup` skills.

Workflow steps:

1. Load site configuration and QC reference (Level 2 pass/fail status, passing years, missing years).
2. For stations passing Level 2 gate, retain all available years/storm-years/months; carry QC diagnostics forward.
3. Confirm MHHW datum availability.
4. Convert water levels to height relative to MHHW where needed.
5. Apply `30 cm above MHHW` threshold.
6. Compute daily maxima from hourly values.
7. Count flood days (daily maximum ≥ threshold).
8. Aggregate by May-April storm year (see Governance § Temporal Aggregation and Data Completeness Rules) and optionally by month.
9. Preserve true no-data periods as missing/empty/`NaN`; do not convert to zero-count periods (see Governance § No-Data Preservation).
10. Save tables and provenance.

**Handling QC Failures**: Stations failing Level 2 gate should not be included in primary products. To generate exploratory sensitivity products for failed stations, explicitly label as exploratory and document the QC failure reason.

## Approved Figures

Use only approved helpers listed in **Governance § Approved Plotting Helpers Reference § Flood-Frequency Helpers**. See that reference for complete parameter documentation.

For `Regional`, the report-facing product is the station-year flood-day matrix with annual regional total-count bar chart using `plot_regional_flood_frequency_overview`.

Do not create final flood-frequency figures with ad hoc plotting code. If a required figure is unsupported, follow the proposal process in Governance § Approved Plotting Helpers Reference § Requesting New Helpers.

## Monthly Products

Use storm-year month order: `May, Jun, Jul, Aug, Sep, Oct, Nov, Dec, Jan, Feb, Mar, Apr`.

Distinguish monthly count figures from percent contribution figures. Percent contribution is monthly flood days divided by total flood days across all months in the analyzed period, multiplied by 100. Raw counts must never be labeled as percentages.

## Required Outputs Where Data Support Them

- Annual storm-year flood-day table and plot.
- For `Regional`, station-year matrix plus annual regional total-count bar chart.
- `flood_frequency_summary.csv`.
- Station-level Level 2 pass/fail table.
- Station-year/storm-year flood-day table with QC diagnostics.
- Total flood days, average flood days/year, maximum year, and trend/p-value where approved.
- Monthly contribution table if monthly contribution figures are generated.

## Validation

Confirm:
- QC status: Station passes Level 2 gate for intended period (see Quality Control skill)
- MHHW availability and datum compatibility
- Retention of partial-data periods as QC diagnostics; true no-data periods as NaN
- Storm-year aggregation: May 1 – April 30 (see Governance § Temporal Aggregation)
- Daily maxima (not daily means)
- Monthly vs. percent-contribution labels (raw counts never labeled as percentages)

## Flood Matrix Summary Product — Consolidated FF04 Update

For station flood-frequency packages where data support annual and monthly storm-year products, include the composite flood matrix summary as a report-facing product.

### Product

- `product_id`: `FF04_flood_matrix_summary`
- `product_family`: `Minor Flood Frequency from Stations`
- `product_role`: `required primary` when UHSLC hourly data, MHHW, daily maxima, storm-year annual counts, monthly matrix counts, and Level 2 QC are available; otherwise `deferred`
- `plotting_helper`: `plot_flood_matrix_summary`

### Requirements

The figure must distinguish:

- annual storm-year flood-day counts;
- monthly flood-day **count** cells;
- monthly percent-contribution bars;
- ONI/ENSO contextual labels, if included.

ONI context must be retrieved with `get_climate_index("ONI")` when used. ONI is contextual only and must not alter flood-day counts. Save supporting tables such as matrix inputs, annual flood-day/ONI mode table, and product metrics.

Captions must state `30 cm above MHHW`, daily maxima, May-April storm-year convention, count-versus-percent distinction, ONI context if present, and review status.
