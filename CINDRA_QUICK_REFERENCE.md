# CIndRA Skills Quick Reference Guide

**One-page workflow cheat sheet for sea-level indicator products**

---

## Overall Workflow (4 Steps)

```
1. SITE SETUP          2. QUALITY CONTROL    3. DOMAIN ANALYSIS    4. ASSEMBLY
   └─ site_config.json    ✓ Level 2 gate       ├─ Trend              └─ Products
                          ✓ Passing years      └─ Flood-Frequency       (captions,
                                                  (parallel)             methods,
                                                                        validation)
```

**Duration**: Setup (30 min) → QC (15 min) → Domain (1-2 hrs) → Assembly (30 min)

---

## Step-by-Step Quick Guide

### 1️⃣ SITE SETUP (`cindra-site-setup`)
**Purpose**: Create site_config.json with station + boundary metadata

**Input**: Site name, coordinates, station ID (or auto-select by proximity)

**Output**: `site_config.json` with:
- Selected UHSLC station ID, name, country
- Product profile: Regional OR National/EEZ
- Spatial scope, boundary metadata, CMEMS extraction bounds
- Analysis period (default: 1993-2025)

**Choose Profile**:
| Regional | National/EEZ |
|---|---|
| Multiple stations in region | Single country/EEZ boundary |
| Example: PICCM Pacific Islands | Example: Palau EEZ |
| Regional helpers (Draft) | National helpers (Production) |
| Requires scientist validation | Standard QC procedures |

→ See Site Setup § "Regional vs. National/EEZ Product Profile Workflows"

---

### 2️⃣ QUALITY CONTROL (`cindra-quality-control`) — **MANDATORY**
**Purpose**: Validate station suitability; flag failing years

**Input**: site_config.json + UHSLC station data

**Output**: QC reference with:
- Level 2 gate: PASS or FAIL
- Passing years (≥75% complete)
- Missing years (zero data)
- Completeness fractions, diagnostic flags

**Decision**:
- ✅ **Station PASSES Level 2** → Proceed to Domain Analysis
- ❌ **Station FAILS Level 2** → Choose alternative (narrower period, different station, or mark as exploratory)

→ See Quality Control skill for detailed rules

---

### 3️⃣ DOMAIN ANALYSIS (Run Either or Both in Parallel)

#### 3A. Trend Analysis (`cindra-sea-level-trend`)
**Purpose**: Compute sea-level trends (absolute altimetry + relative tide gauge)

**Input**: site_config.json + QC reference (required)

**Output**:
- `station_trend_summary.csv` — Trend rate (mm/yr), sea-level change (cm), p-value
- `altimetry_trend_summary.csv` — Absolute altimetry trends
- `integrated_trend_comparison.csv` — Station vs. altimetry side-by-side
- Trend maps and time series (PNG)

**Key Outputs for Product Assembly**:
- Trend magnitude (mm/yr or cm/epoch)
- Uncertainty/p-value
- Analysis period, units, datum

---

#### 3B. Flood-Frequency Analysis (`cindra-flood-frequency`)
**Purpose**: Compute daily max water-level exceedances; storm-year aggregation

**Input**: site_config.json + QC reference (required)

**Output**:
- `flood_frequency_summary.csv` — Annual flood-day counts (May-April storm year)
- Storm-year & monthly flood-day tables
- Flood-frequency plots (PNG)

**Key Outputs for Product Assembly**:
- Annual flood-day counts by year
- May-April storm-year convention (confirmed)
- Monthly counts vs. percentages (labeled correctly)

---

### 4️⃣ PRODUCT ASSEMBLY (`cindra-product-assembly`)
**Purpose**: Package outputs for review + publication

**Input**: All outputs from Steps 1-3 plus:
- Figures (PNG/PDF) from Trend + Flood-Frequency
- QC diagnostics
- Boundary provenance (for National/EEZ)

**Output**:
- Product inventory (Phase A)
- Captions (Phase B)
- Methods & limitations notes (Phase C)
- Metadata & provenance (Phase D)
- Validation checklist (Phase E)
- Report sections (Phase F)

**Verify Before Submitting**:
- ✅ All products supported by QC screening
- ✅ Captions state: relative (tide gauge) vs. absolute (altimetry)
- ✅ Figures generated from approved helpers (see Governance)
- ✅ All data sources, versions, access dates recorded
- ✅ Missing/deferred products documented (not hidden)

---

## Common Questions → Which Skill?

| Question | Answer | Skill |
|----------|--------|-------|
| **"Where are all plotting helpers?"** | 23+ helpers organized by category (Trend, Flood-Freq, Anomaly, Rankings, Regional) | **Governance** § Approved Plotting Helpers Reference |
| **"Is my station suitable?"** | Validate Level 2 gate (≥20 passing years, ≤5 missing years) | **Quality Control** |
| **"What's the 75% completeness rule?"** | Storm-year/Calendar-year/Monthly passes if ≥75% usable days | **Governance** § Temporal Aggregation & Completeness Rules |
| **"What's the workflow after site setup?"** | Setup → QC (required) → [Trend + Flood-Freq parallel] → Assembly | **Site Setup** § Downstream Workflow Triggers |
| **"Which skill generates `station_trend_summary.csv`?"** | Trend Analysis via `a_sea_level_trend.ipynb` | **Product Assembly** § Input Sources by Upstream Skill |
| **"Can I skip QC for exploratory products?"** | No. QC is mandatory for all products (primary, exploratory, diagnostic) | **Governance** § Mandatory Quality Control Workflow |
| **"How do I request a new plotting helper?"** | Add to `functions/sea_level_plotting.py`, then update governance | **Governance** § Approved Plotting Helpers Reference § Requesting New Helpers |
| **"Regional or National/EEZ?"** | Regional = station mask + Draft helpers; National = boundary + production helpers | **Site Setup** § Regional vs. National/EEZ Product Profile Workflows |
| **"What does 'May-April storm year' mean?"** | May 1 – April 30; labeled by starting year (e.g., storm year 2020 = May 1, 2020–April 30, 2021) | **Governance** § Temporal Aggregation & Completeness Rules |
| **"How do I preserve no-data periods?"** | Represent as NaN (never convert to 0); carry forward as QC diagnostics | **Governance** § No-Data Preservation |

---

## Key Terminology

### Storm Year
- **Definition**: May 1 – April 30 (inclusive)
- **Label**: By starting year (e.g., 2020 = May 2020–April 2021)
- **Completeness**: ≥75% usable days
- **Application**: Flood-frequency, sea-level rankings

### Level 2 Gate (Mandatory QC Blocking Gate)
- **Passes if**: ≥20 calendar years with ≥75% daily coverage AND ≤5 completely missing years
- **If FAILS**: Choose alternative station/period OR explicitly label as exploratory
- **Cannot override**: Not optional filtering; blocking gate

### Relative vs. Absolute Sea Level
- **Tide Gauge** = Relative sea level (measured at station, relative to land)
- **Altimetry** = Absolute sea level (satellite measurement, geocentric frame)
- **Never merge** these into single metric without approved method

### Approved Plotting Helpers
- **Production-ready**: `functions/sea_level_plotting.py` (23+ helpers)
- **Draft/Experimental**: `functions/cindra_regional_plotting_helpers.py` (require scientist validation)
- **Ad hoc plotting**: NOT allowed; use approved helpers only

---

## Decision Tree: Which Product Profile?

```
START: "I need to create sea-level products"
│
├─ "Multiple stations in geographic region?"
│  └─ YES → REGIONAL PROFILE
│     ├─ Use: cindra-piccm-regional-definition (for station mask)
│     ├─ Helpers: plot_regional_* (Draft/Experimental status)
│     └─ Validation: Requires scientist review
│
└─ "Single country or EEZ?"
   └─ YES → NATIONAL/EEZ PROFILE
      ├─ Use: Project-approved boundary (hierarchy: project → SPREP → VLIZ)
      ├─ Helpers: plot_magnitude_map, plot_national_eez_* (Production-ready)
      └─ Validation: Standard QC procedures
```

---

## Approved Data Sources (Authoritative)

| Data | Source | Version | Notes |
|------|--------|---------|-------|
| **Tide Gauge Stations** | UHSLC | Fast Delivery or latest Research Quality | Do not mix RQDS versions |
| **Altimetry** | CMEMS/Copernicus | Monthly 0.25° SSH L4 | URL in Governance skill |
| **Boundaries** | Project-approved (Regional: PICCM; National: VLIZ/SPREP) | Latest available | See Governance § Boundaries |

---

## Checklist: Before Submitting Products

- [ ] Site setup completed with correct profile (Regional vs. National/EEZ)
- [ ] Quality Control passed (Level 2 gate = PASS)
- [ ] Trend analysis completed (if required) with QC reference included
- [ ] Flood-frequency analysis completed (if required) with QC reference included
- [ ] All figures generated using approved helpers (see Governance)
- [ ] Captions state: tide gauge = relative; altimetry = absolute
- [ ] Storm-year convention verified: May 1–April 30
- [ ] No-data periods preserved as NaN (not converted to 0)
- [ ] Provenance includes: source, version, access date, DOI, spatial method
- [ ] Missing/deferred products documented (not hidden)
- [ ] Review status assigned (Experimental / Draft / Scientist-reviewed / Approved for report use)

---

## Skill Quick Links

| Skill | Best For | Key Sections |
|-------|----------|---|
| **Governance** | Rules, policies, helpers | Mandatory QC, Approved Helpers, Temporal Aggregation, Boundaries |
| **Site Setup** | Starting a workflow | Downstream Triggers, Regional vs. National, Boundary Metadata |
| **Quality Control** | Station validation | Level 2 Gate, Daily/Monthly/Storm-Year Rules, Output Requirements |
| **Trend Analysis** | Sea-level trends | Required Products, Approved Figures, Combined Magnitude Map |
| **Flood-Frequency** | Flood-day counting | May-April Storm-Year, Monthly Products, Flood Matrix Summary |
| **Product Assembly** | Packaging outputs | Input Sources Matrix, Phases A-F, Validation Checklist |
| **Regional Definition** | Regional station masks | PICCM Inclusion/Exclusion, Regional Boundaries, Limitations |

---

## When Stuck: Diagnostic Guide

| If... | Then... | Skill |
|------|--------|-------|
| "I don't know where to start" | Follow 4-step workflow (Setup → QC → Domain → Assembly) | Site Setup |
| "My station keeps failing QC" | Read Level 2 rules; evaluate narrower period or different station | Quality Control |
| "I can't find a plotting function" | Search Governance § Approved Helpers by category | Governance |
| "My product doesn't validate" | Check Product Assembly Phase A (inventory) for role-specific requirements | Product Assembly |
| "Is 75% the right threshold?" | Yes (universal for month/year/storm-year). See Governance § Temporal Aggregation | Governance |
| "Can I skip QC?" | No. QC is mandatory for all product types. | Governance |
| "What output files should I have?" | Check Product Assembly § Input Sources Matrix for expected files | Product Assembly |
| "Regional or National?" | See Site Setup § Regional vs. National decision tree | Site Setup |

---

## Estimated Time Investment

| Step | Time | Notes |
|------|------|-------|
| **1. Site Setup** | 15-30 min | Faster if stations pre-selected |
| **2. Quality Control** | 10-20 min | Depends on station quality |
| **3. Trend Analysis** | 30-60 min | Includes figure generation |
| **3. Flood-Frequency** | 30-60 min | Parallel to Trend; includes figures |
| **4. Product Assembly** | 20-40 min | Captions, methods, validation |
| **TOTAL** | 2-3 hours | Full workflow from setup to assembly |

---

**For detailed documentation**, see individual skill documents in `assistant/skills/` or analysis documents (SKILLS_REDUNDANCY_ANALYSIS.md, PHASE_1-2_REVIEW.md, PHASE_3_INTEGRATION_SUMMARY.md).

