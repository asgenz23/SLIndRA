# CIndRA Skills Redundancy Analysis & Improvement Recommendations

**Analysis Date**: 2026-08-13  
**Source of Truth**: Functions (`functions/*.py`), Notebooks (`notebooks/historical/`), and Skills (`assistant/skills/`)

---

## Executive Summary

The seven CIndRA skills have **significant overlaps in scope and terminology** that could be consolidated to reduce confusion, improve maintenance, and streamline skill invocation. The skills are organized around **workflow phases and product families** rather than by **user intents**, creating redundant policy statements and guardrails.

**Key findings:**
1. **Policy/Governance rules** are spread across 3+ skills instead of centralized
2. **Data source hierarchy and provenance** duplicated in multiple skills
3. **Validation and QC terminology** appears in both QC and Product Assembly skills
4. **Approved figure policies** present in both Governance and specific workflow skills
5. **Plotting helper references** duplicated across Trend and Flood-Frequency skills

---

## Detailed Redundancy Map

### 1. **Governance Rules Distributed Across Skills**

| Rule / Policy | Current Location | Duplication |
|---|---|---|
| UHSLC as authoritative source | `sea-level-governance` (explicit) | Repeated in `sea-level-trend`, `flood-frequency`, `site-setup` |
| CMEMS altimetry source | `sea-level-governance`, `sea-level-trend` | Appears in 2 places |
| Product profiles (Regional/National/EEZ) | `sea-level-governance`, `product-assembly` | Defined twice |
| Figure policy (approved helpers only) | `sea-level-governance`, `sea-level-trend`, `flood-frequency` | Appears in 3+ skills |
| Review status labels | `sea-level-governance`, `product-assembly` | Identical lists, separate sections |
| Code repository furnishing policy | `sea-level-governance`, `product-assembly` | Duplicated |

**Redundancy Impact**: Users must cross-reference 3+ skills to understand a single policy rule.

---

### 2. **Provenance and Data Source Hierarchy**

**Current State:**
- `sea-level-governance`: Defines source hierarchy (UHSLC priority, CMEMS rules)
- `site-setup`: Repeats coverage source rule, RQDS version handling
- `product-assembly`: Requires "preserve provenance" but doesn't define structure
- `sea-level-trend`: Lists required provenance fields in workflow step 10

**Problem**: Provenance schema is implied across 3 skills; no single authoritative source.

**Action Item**: Consolidate provenance schema in `sea-level-governance` skill, with cross-references in downstream skills.

---

### 3. **Quality Control Rules and Terminology**

**Current Distribution:**

| Term / Rule | QC Skill | Product Assembly | Governance |
|---|---|---|---|
| Level 2 gate definition | Primary | Referenced | Not mentioned |
| Station suitability workflow | Primary | N/A | N/A |
| Daily/monthly/storm-year rules | Primary (detailed) | N/A | N/A |
| QC output requirements | Primary (detailed) | N/A | Generalized as "QC reference" |
| Missing data handling | Primary (NaN preservation) | N/A | Mentioned as "diagnostics" |

**Redundancy**: The `product-assembly` skill requires "QC status/reference" without defining what constitutes valid QC output. Users must reference the `quality-control` skill separately.

**Action Item**: Add QC output specification section to `product-assembly` that explicitly cross-references the `quality-control` skill's output requirements, rather than assuming prior QC knowledge.

---

### 4. **Approved Figure / Plotting Helper Policy**

**Current Duplication:**

| Helper | sea-level-governance | sea-level-trend | flood-frequency |
|---|---|---|---|
| `plot_magnitude_map` | "approved" | Listed in approved figures | N/A |
| `plot_altimetry_trend_timeseries` | "approved helpers in sea_level_plotting.py" | Explicitly listed | N/A |
| `plot_flood_matrix_summary` | General policy | N/A | Explicitly listed |
| `plot_regional_altimetry_trend_map_filled_tide_gauges` | Not mentioned | Not mentioned | N/A (Regional skill needed) |

**Problem**: 
- `sea-level-governance` states "use helpers in `functions/sea_level_plotting.py`" but doesn't enumerate them
- `sea-level-trend` and `flood-frequency` repeat the enumeration
- New helpers like `plot_regional_altimetry_trend_map_filled_tide_gauges` (in `cindra_regional_plotting_helpers.py`) are not mentioned in governance

**Action Item**: 
1. Move comprehensive helper enumeration to `sea-level-governance` 
2. Organize by category: Trend, Flood-Frequency, Anomaly, Rankings, Regional
3. Reference this canonical list in `sea-level-trend` and `flood-frequency` with "See governance for complete list"

---

### 5. **Regional Definition Isolation**

**Current State:**
- `regional-definition` skill is **hyper-specific** to PICCM/Pacific Islands (v0.2)
- Describes itself as "not a final geospatial product boundary"
- Contains manual station overrides and inclusion/exclusion criteria

**Finding**: This skill is appropriate in scope but **lacks integration points** with other skills.

**Gap**: `site-setup` skill mentions "use the CIndRA regional station mask skill" but doesn't explain how Regional definition output feeds into a `National/EEZ` product profile workflow.

**Action Item**: Add workflow diagram to `site-setup` showing how regional masks vs. national/EEZ boundaries are used together.

---

### 6. **Product Assembly Phases A-F Scope Creep**

**Current State:**
- `product-assembly` skill combines 6 phases (inventory, captions, methods, provenance, validation, assembly)
- Phases reference upstream "CIndRA skills" but don't clearly delineate which skill is responsible for what input

**Problem Example**:
- Phase D (Provenance) says "preserve or reference ... `station_trend_summary.csv`" but doesn't say which skill generates this file
- Phase E (Validation) references "QC reference" but QC is managed by a separate skill
- Phase D says "Do not furnish Code repository bundle unless requested" — same policy stated in `sea-level-governance`

**Action Item**: Add matrix to `product-assembly` showing which skill (Setup, QC, Trend, etc.) produces each required input file.

---

### 7. **Daily/Storm-Year Aggregation Rules Duplicated**

**Current Duplication:**

| Aggregation Rule | Quality-Control | Flood-Frequency | Notes |
|---|---|---|---|
| May-April storm year definition | Not defined | "runs May 1-April 30, labeled by starting year" | QC skill is silent; FF skill defines |
| Daily maximum convention | Defined in QC ("usable daily") | "daily maximum water-level exceedance" | FF skill repeats concept differently |
| 75% completeness threshold | "at least 75% usable-day coverage" | Not explicitly stated | Only in QC skill |
| Preservation of no-data as NaN | Explicitly stated | "Do not... convert to valid zero-count periods" | Same rule, stated twice |

**Finding**: The `flood-frequency` skill assumes familiarity with QC terminology but repeats practical details. A user reading only `flood-frequency` might not understand the 75% rule.

**Action Item**: Create a **shared aggregation/completeness reference** (new section in `sea-level-governance` or cross-reference card) that both QC and Flood-Frequency point to.

---

## Concrete Improvement Recommendations

### **Priority 1: Consolidate Governance & Policy (High Impact)**

**Create a NEW section in `sea-level-governance`:**

```markdown
## Approved Plotting Helpers Reference

### Canonical Helper Locations
- `functions/sea_level_plotting.py` — core trend, flood, anomaly, rankings helpers
- `functions/cindra_regional_plotting_helpers.py` — regional-domain helpers (Draft/Experimental)

### By Category

#### Trend Analysis
- plot_magnitude_map
- plot_magnitude_map_background
- plot_altimetry_trend_timeseries
- plot_tide_gauge_trend_timeseries
- plot_combined_trends
- plot_enso_scatter
- plot_national_eez_combined_trend_map
- plot_regional_altimetry_trend_map_filled_tide_gauges (Draft)

#### Flood Frequency
- plot_histogram_with_threshold
- plot_flood_counts_with_trend
- plot_flood_counts_with_oni
- plot_flood_days_heatmap
- plot_flood_matrix_summary
- plot_oni_only
- plot_monthly_contribution_vertical
- plot_regional_flood_frequency_overview (Draft)

#### [Continue for Anomaly, Rankings, Regional...]
```

**Then update downstream skills:**
- `sea-level-trend`: "See Governance § Approved Plotting Helpers for complete list"
- `flood-frequency`: "See Governance § Approved Plotting Helpers for complete list"

**Impact**: Single source of truth; easier to add new helpers; clearer to users.

---

### **Priority 2: Unify QC & Completeness Rules (Medium Impact)**

**Add new section to `sea-level-governance`:**

```markdown
## Aggregation & Completeness Rules

### Common Thresholds
- **Usable-day coverage**: ≥75% of expected days for month/year/storm-year
- **Storm year**: May 1 – April 30, labeled by starting year
- **No-data preservation**: True gaps remain NaN; do not convert to 0

### Reference
For detailed daily-interval rules, six-hour windows, and Level 2 station-level gates, see `cindra-quality-control`.
```

**Impact**: Reduces cross-skill references; clarifies "why 75%?" for users reading Product Assembly.

---

### **Priority 3: Refactor Product Assembly (Medium Impact)**

**In `product-assembly`, add under "Required Inputs":**

```markdown
### Input Sources by Skill

| Input | Generated By | Defined In | Examples |
|---|---|---|---|
| Site configuration | `cindra-site-setup` | `site-setup` skill | Selected stations, boundary metadata |
| QC status & diagnostics | `cindra-quality-control` | `quality-control` skill | Level 2 pass/fail, missing years |
| Station trend/magnitude | `cindra-sea-level-trend` | `sea-level-trend` skill | station_trend_summary.csv |
| Flood frequency tables | `cindra-flood-frequency` | `flood-frequency` skill | flood_frequency_summary.csv |
| Figure outputs | All domain skills | See Governance § Plotting Helpers | PNG/PDF files + metadata |
| Provenance records | All skills | `sea-level-governance` § Provenance | machine-readable JSON/CSV |
```

**Impact**: Users know exactly which skill to consult before calling Product Assembly.

---

### **Priority 4: Regional Definition Integration (Low Impact, High Clarity)**

**In `site-setup`, add workflow section:**

```markdown
## Regional vs. National/EEZ Workflow

### Regional Profile (e.g., PICCM Pacific Islands)
1. Use `cindra-piccm-regional-definition` to resolve regional station mask
2. Pass mask result to site-setup
3. `product_profile: Regional` in site config
4. Downstream analysis uses regional plotting helpers (Draft)

### National/EEZ Profile
1. Use project-approved EEZ boundary (see Governance § Boundaries)
2. Pass boundary metadata to site-setup
3. `product_profile: National/EEZ` in site config
4. Downstream analysis uses national plotting helpers
```

**Impact**: Clarifies when to invoke regional-definition skill.

---

### **Priority 5: Site Setup Input Clarification (Quick Win)**

**In `site-setup`, clarify when each downstream skill is needed:**

```markdown
## Downstream Workflow Triggers

After creating site config, use:

- **`cindra-quality-control`** — to validate station suitability and flag failing years
- **`cindra-sea-level-trend`** — to compute trend analysis using the validated site config
- **`cindra-flood-frequency`** — to compute flood frequency using validated site config
- **`cindra-product-assembly`** — to package outputs into report-ready products
```

**Impact**: Explicit sequencing reduces user confusion.

---

## Summary Table: Redundancy Status by Skill

| Skill | Redundancy Level | Primary Issue | Recommendation |
|---|---|---|---|
| `sea-level-governance` | **HIGH** | Policy scattered, helpers not enumerated | Consolidate + enumerate |
| `site-setup` | **MEDIUM** | Assumes downstream workflow knowledge | Add explicit trigger matrix |
| `quality-control` | **MEDIUM** | Completeness rules not cross-referenced | Point to shared governance section |
| `sea-level-trend` | **HIGH** | Duplicates helpers and governance rules | Reference governance; reduce detail |
| `flood-frequency` | **HIGH** | Duplicates helpers, QC terminology, NaN rules | Reference governance + QC |
| `product-assembly` | **MEDIUM-HIGH** | Input sources unclear; provenance scattered | Add input-source matrix |
| `regional-definition` | **LOW** | Isolated; integration pathway unclear | Add workflow diagram to site-setup |

---

## Implementation Priority

**Phase 1 (Immediate):**
1. Centralize approved plotting helpers in `sea-level-governance`
2. Consolidate review-status labels and code-repository policy in `sea-level-governance`

**Phase 2 (Near-term):**
3. Add aggregation/completeness reference to `sea-level-governance`
4. Add input-source matrix to `product-assembly`
5. Add downstream-workflow triggers to `site-setup`

**Phase 3 (Medium-term):**
6. Simplify `sea-level-trend` and `flood-frequency` by removing duplicated policy; add cross-references
7. Add regional/national workflow diagram to `site-setup`

---

## Files to Update

| File | Priority | Changes |
|---|---|---|
| `assistant/skills/sea-level-governance/SKILL.md` | P1 | Add Approved Plotting Helpers section; consolidate review-status labels |
| `assistant/skills/product-assembly/SKILL.md` | P2 | Add input-source matrix under "Required Inputs" |
| `assistant/skills/site-setup/SKILL.md` | P2 | Add downstream workflow triggers; add regional/national diagram |
| `assistant/skills/sea-level-trend/SKILL.md` | P3 | Remove duplicated helpers list; add reference to governance |
| `assistant/skills/flood-frequency/SKILL.md` | P3 | Remove duplicated helpers list; reference governance for QC rules |
| `assistant/skills/quality-control/SKILL.md` | P2 | Add cross-reference to governance for completeness thresholds |
| `assistant/skills/regional-definition/SKILL.md` | P2 | Add integration note pointing to site-setup workflow |

---

## Validation Against Source of Truth

All recommendations are based on actual codebase inspection:

✅ **Functions verified:**
- `functions/sea_level.py`: `process_trend_with_nan()`, `select_uhslc_station()`, completeness rules
- `functions/sea_level_plotting.py`: 20+ approved helpers
- `functions/cindra_regional_plotting_helpers.py`: Regional helpers (Draft)
- `functions/data_downloaders.py`: UHSLC/CMEMS download logic

✅ **Notebooks verified:**
- `notebooks/historical/National/sea_level/`: 0_site_setup, a_sea_level_trend, b_sea_level_anomaly, c_sea_level_ff, d_sea_level_rankings
- `notebooks/historical/Regional/`: 00_regional_setup, regional_plots

✅ **No conflicts** between function signatures and skill descriptions found.

---

## Next Steps

1. **Open skills for editing** in VS Code
2. **Implement Phase 1** changes to `sea-level-governance` (add helpers enumeration, consolidate labels)
3. **Review and test** with team; run a sample product-assembly workflow using updated skills
4. **Iterate on Phases 2-3** based on feedback

