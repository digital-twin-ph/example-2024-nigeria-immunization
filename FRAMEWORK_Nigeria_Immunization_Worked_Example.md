# FrameWork Worked Example: Nigeria District-Level Immunization Surveillance

**Status:** Implementation specification / data acquisition plan  
**Version:** 0.1 — 26 October 2026  
**Geography:** Nigeria, ADM2 (Local Government Areas, LGAs)  
**Primary period:** 2000–2024; initial demonstration years 2018, 2021, 2024  
**Data classification:** Public, aggregate, predominantly model-based estimates

## 1. Purpose

Extend FrameWork's South London cholera worked example with a contemporary, reproducible district-level immunization surveillance use case. Demonstrate data discovery, spatial joins, temporal analysis, uncertainty-aware inference, transparent prioritization, and map generation using documented evidence.

**Primary question:** Which Nigerian LGAs have persistently low DTP1 coverage, large estimated numbers of zero-dose children, or both, and how do these patterns relate to health-service geography?

**Important boundary:** The initial release is *surveillance-oriented analysis of modeled estimates*, not a live operational immunization surveillance system. Do not label modeled values as observed administrative reports or individual vaccination records.

## 2. Learning objectives

1. Load versioned public datasets with provenance and licenses.
2. Harmonize ADM2 identifiers and validate polygon topology.
3. Compare DTP1, DTP3, MCV1, and zero-dose estimates over time.
4. Distinguish rates, estimated counts, uncertainty, and changes in denominators.
5. Identify persistent low-coverage areas and quantify spatial clustering.
6. Overlay health facilities and accessibility surfaces without asserting that every facility provides immunization.
7. Generate explainable, uncertainty-aware LGA prioritization results.
8. Export interoperable spatial data, maps, metadata, and a reproducible analysis manifest.

## 3. Verified source inventory

| ID | Dataset | Geography/time | Format / availability | Role | Caveat |
|---|---|---|---|---|---|
| NGA-VAX-01 | WorldPop VaxPop: DTP1, DTP3, MCV1 coverage estimates | ADM0, regional, ADM1, ADM2; 2000–2024 | ZIP with GeoPackage, shapefiles, CSV | Core indicators | Model-based, not administrative counts |
| NGA-VAX-02 | WorldPop VaxPop: zero-dose estimates | ADM0, ADM1, ADM2; 2000–2024 | ZIP with GeoPackage, CSV | Core estimated counts | Uses coverage estimates and under-one population model |
| NGA-VAX-03 | WorldPop coverage rasters | ~1 km; 2000–2024 | GeoTIFF ZIP by antigen | Optional fine-resolution module | Large downloads; uncertainty layers require schema inspection |
| NGA-GEO-01 | GRID3 operational LGA boundaries | ADM2, December 2020 release | GRID3 catalogue | Boundary and spatial joins | Boundary vintage may not match VaxPop ADM2 geometry |
| NGA-GEO-02 | GRID3 health facilities v3.0 and v2.0 | v3 covers selected states; v2 covers all states | GRID3 catalogue | Facility proximity | Coverage varies; service availability unverified |
| NGA-GEO-03 | GRID3 roads and travel-time friction surface | 2025 releases | GRID3 catalogue | Optional accessibility module | Travel-time method and licensing must be inspected |
| NGA-POP-01 | WorldPop Nigeria age-sex estimates | 2024 among available years | GeoTIFF | Optional denominator sensitivity | Do not substitute for the source study's denominator without documentation |

**Primary download:** https://hub.worldpop.org/doi/10.5258/SOTON/WP00880  
**GRID3 catalogue:** https://grid3.org/geospatial-data-nigeria  
**GRID3 data hub:** https://data.grid3.org/  
**WorldPop age-sex data:** https://hub.worldpop.org/geodata/summary?id=82494

WorldPop release details: `NGA_ADM_coverage_estimates.zip` (35.38 MB), `NGA_ADM_zero_dose_estimates.zip` (9.10 MB), and optional DTP1/DTP3/MCV1 raster archives (roughly 550–565 MB each). GIS data use WGS84. WorldPop describes the release as CC BY 4.0; check the individual GRID3 layer terms separately.

**Acquisition status:** Source pages and package descriptions verified; archives have **not** yet been downloaded or their field schemas inspected. Treat column names below as proposed canonical names, not actual source field names.

## 4. Scope and exclusions

### In scope: MVP
- Nationwide ADM2 maps for selected years.
- DTP1, DTP3, MCV1 coverage and zero-dose counts.
- LGA-level temporal changes, persistence and rankings.
- Uncertainty display when source intervals or SDs are supplied.
- Source-to-canonical field mapping and provenance.
- Boundary mismatch report and join quality metrics.
- Reproducible exports: GeoPackage, GeoJSON, CSV, metadata JSON.

### Phase 2
- Full 2000–2024 animation and time-series explorer.
- 1 km coverage rasters and multiscale comparisons.
- Facility accessibility, road/travel-time modeling.
- Survey-versus-routine administrative data triangulation, *only if separately acquired*.
- Measles notifications/outbreak overlays, *only if independently sourced at a compatible geography*.

### Out of scope
- Individual-level tracking, patient identification, or vaccine eligibility decisions.
- Causal attribution of low coverage to facility access without additional evidence.
- Treating statistical estimates as official district programme performance scores.

## 5. Proposed canonical data model

### `admin_unit`
`admin_id`, `admin_level`, `admin_name`, `parent_id`, `boundary_source`, `boundary_version`, `geometry`, `geometry_crs`, `valid_from`, `valid_to`.

### `immunization_estimate`
`admin_id`, `year`, `antigen_dose` (`DTP1|DTP3|MCV1`), `coverage_pct`, `uncertainty_lower`, `uncertainty_upper`, `uncertainty_type`, `source_dataset`, `source_version`, `estimation_method`, `cohort_definition`.

### `zero_dose_estimate`
`admin_id`, `year`, `zero_dose_count`, `uncertainty_lower`, `uncertainty_upper`, `population_basis`, `source_dataset`, `source_version`.

### `health_facility`
`facility_id`, `facility_name`, `geometry`, `source_version`, `facility_type`, `immunization_service_status` (`confirmed|unknown|not_offered`), `validation_status`.

### `analysis_assertion`
`assertion_id`, `subject_admin_id`, `predicate`, `object_value`, `period`, `rule_id`, `input_record_ids`, `uncertainty`, `evidence_status`, `generated_at`.

**Join policy:** Prefer stable codes from the original packages. Never silently join on normalized names alone; generate an explicit crosswalk and report unmatched/ambiguous records. Preserve source boundary geometries separately from GRID3 boundaries.

## 6. Indicators and interpretation

| Indicator | Definition | Interpretation |
|---|---|---|
| DTP1 coverage | Estimated share of relevant birth cohort receiving DTP1 | Access/initial vaccination proxy |
| DTP3 coverage | Estimated share receiving DTP3 | Three-dose coverage |
| MCV1 coverage | Estimated share receiving first measles-containing dose | Measles vaccine coverage proxy |
| DTP1–DTP3 gap | `DTP1_pct - DTP3_pct` | Percentage-point gap; **not** the standard dropout percentage |
| Approximate DTP1→DTP3 dropout | `(DTP1_pct - DTP3_pct) / DTP1_pct × 100` when DTP1 > 0 | Derived approximation only when denominators/cohorts align; not a replacement for dose-count-based dropout |
| Zero-dose count | Source-modeled count of under-one children without DTP1 | Absolute burden, distinct from coverage rate |
| Coverage change | `coverage_pct(t2) - coverage_pct(t1)` | Percentage-point change, not percent growth |
| Persistence | Number of selected years below a documented threshold | Sensitivity to threshold must be reported |

Do not independently recompute zero-dose counts from coverage and a different population raster and present them as WorldPop's estimates. If recomputed, name the output `scenario_zero_dose_count` and retain the formula, denominator, and source.

## 7. Analytical workflow

### Step A — Acquire and inspect
1. Download both ADM ZIP files from the WorldPop DOI page.
2. Save original ZIPs unchanged and calculate SHA-256 checksums.
3. Inventory internal CSV, GeoPackage and shapefile members.
4. Inspect actual field names, units, year encoding, confidence/credible interval conventions, missing-value markers and identifier keys.
5. Inspect GRID3 LGA and facility releases and their licenses.
6. Write `data_inventory.json` and `schema_mapping.yml` before any transformation.

### Step B — Harmonize
1. Convert selected source tables to canonical long-form records.
2. Confirm whether coverage is encoded as 0–1 or 0–100 before conversion.
3. Preserve uncertainty statistics and their exact meanings.
4. Build and review source-LGA-to-GRID3-LGA crosswalk.
5. Check duplicates for (`admin_id`, `year`, `antigen_dose`).
6. Report missing years and unmatched LGAs; do not fill missing values with zero.

### Step C — Analyze
1. Map DTP1, DTP3, MCV1 and zero-dose estimates for 2024.
2. Compare 2018, 2021 and 2024, subject to actual data availability.
3. Compare lowest-coverage LGAs against highest-zero-dose-count LGAs.
4. Quantify persistence and changes with uncertainty caveats.
5. Optionally calculate Moran's I / local spatial clusters, with spatial weights, multiple-testing policy and edge effects documented.

### Step D — Contextualize
1. Overlay facilities; count nearby facilities only as a descriptive metric.
2. Only label a facility as an immunization provider with validated service metadata.
3. Use a travel-time surface for access analysis after checking methodology.
4. Flag that 2025 roads/facilities and 2020 boundaries are not contemporaneous with all vaccination years.

### Step E — Explain and export
1. Record the evidence inputs for every generated district-level assertion.
2. Produce maps with legends, years, sources, units, and uncertainty notes.
3. Export a machine-readable manifest with versions, hashes, transformations, and parameters.
4. Generate a concise interpretive report emphasizing descriptive association rather than causation.

## 8. Illustrative decision rules — not programme policy

```yaml
ruleset: nigeria_immunization_demo_v0
status: illustrative
rules:
  - id: low_dtp1
    when: "DTP1 coverage < 70 percent"
    output: "low modeled DTP1 coverage"
  - id: high_zero_dose_burden
    when: "estimated zero-dose count > 5000"
    output: "high estimated zero-dose burden"
  - id: priority_review
    when: "low_dtp1 AND high_zero_dose_burden"
    output: "priority for data review and contextual investigation"
```

These thresholds are arbitrary demonstration parameters. Show a sensitivity analysis and do not interpret classifications as validated official priorities. Unknown inputs produce `unknown`, not `false`.

## 9. FrameWork declarative reasoning contract

FrameWork should be able to answer:

- Which LGAs had the lowest modeled DTP1 coverage in 2024?
- Which LGAs had the highest estimated number of zero-dose children?
- Which LGAs were below an adjustable threshold in all three demonstration years?
- Where did DTP1 improve but zero-dose counts increase, and what denominator effects might explain the divergence?
- Which comparisons are uncertain because credible intervals overlap or data definitions differ?
- Which facilities are spatially near an LGA, and which are *verified* to provide immunization?
- Which map features and indicator assertions came from each dataset, version and transformation?

### Example output assertion (illustrative structure, not actual Nigeria data)

```json
{
  "subject_admin_id": "<LGA_CODE>",
  "predicate": "has_low_modelled_DTP1_coverage",
  "period": 2024,
  "object_value": true,
  "rule_id": "low_dtp1",
  "input_record_ids": ["<SOURCE_RECORD_ID>"],
  "evidence_status": "derived_from_modelled_estimate",
  "uncertainty": "<SOURCE_INTERVAL_OR_NULL>"
}
```

## 10. Suggested VS Code project structure

```text
framework-examples/
  nigeria-immunization/
    README.md
    docs/
      data-sources.md
      methodology.md
      limitations.md
    config/
      sources.yml
      schema_mapping.yml
      rules.yml
    data/
      raw/                 # immutable downloads; gitignored
      interim/             # crosswalks and QC
      processed/           # normalized outputs
    scripts/
      01_inventory.py
      02_normalize.py
      03_validate_joins.py
      04_indicators.py
      05_export.py
    tests/
      test_units.py
      test_unique_keys.py
      test_join_coverage.py
      test_missingness.py
    outputs/
      maps/
      tables/
      reports/
      provenance/
```

The Python filenames are an implementation suggestion, not a requirement. A Rust/WASM pipeline can implement the same contracts if that is the preferred FrameWork runtime.

## 11. Validation and acceptance criteria

- [ ] Both WorldPop ADM archives downloaded, hashed and inventoried.
- [ ] Actual source schema documented; no guessed columns used in production code.
- [ ] Coverage units and uncertainty semantics verified.
- [ ] ADM2 record counts and geographic coverage reported for every selected year.
- [ ] LGA joins have a published match rate and unresolved-case report.
- [ ] No duplicate `(admin_id, year, antigen_dose)` records.
- [ ] No missing estimate silently converted to zero.
- [ ] Model-derived and observed indicators have distinct type/provenance labels.
- [ ] Three demonstration years render correctly and export reproducibly.
- [ ] Ranking output includes both rate and absolute burden perspectives.
- [ ] Each generated assertion traces to source records and an explicit rule.
- [ ] Maps include dataset vintage, units, and modeled-estimate caveat.
- [ ] Licenses and required attribution accompany exported outputs.

## 12. Implementation milestones

**M1 — Data feasibility:** Download WorldPop ADM archives; inspect schemas, source IDs, coverage and uncertainty; verify GRID3 downloads.

**M2 — Reproducible ADM2 baseline:** Normalize, join, QC, and export 2024 DTP1/DTP3/MCV1 and zero-dose maps.

**M3 — Longitudinal analysis:** Add 2018/2021 comparisons, persistence, trend charts, and explainable rules.

**M4 — Geospatial augmentation:** Add facility locations, service-status validation, and optional accessibility surfaces.

**M5 — FrameWork integration:** Wire datasets to the framework's actual map rendering, declarative queries, evidence graph, and reproducibility interfaces. Interface names and APIs must be derived from the repository rather than invented here.

## 13. Risks and mitigation

| Risk | Mitigation |
|---|---|
| Different ADM2 boundary vintages | Retain source geometries; build reviewed crosswalk; document topology changes |
| Coverage rate interpreted as observed administrative value | Typed provenance; prominent modeled-estimate labels |
| Uncertainty omitted in rankings | Display source uncertainty; test ranking sensitivity |
| Mixed-age or mixed-cohort indicators | Preserve cohort definition per source; block incompatible comparisons |
| Facility dataset is incomplete | Record completeness limitations and unknown service status |
| Time-mismatched infrastructure | Label vintage; restrict causal interpretation |
| Large raster downloads | Start with two ADM ZIPs; add rasters in Phase 2 |
| Unverified file fields | Require schema inventory before writing ingestion mappings |

## 14. Source references and attribution

1. Utazi C. E., Megheib M., Olowe I. D., Chaudhuri S., Tejedor-Garavito N., and Tatem A. J. (2026). *Modelled vaccination coverage and zero-dose estimates for Nigeria, 2000–2024*, version 1.0. WorldPop, University of Southampton. DOI: [10.5258/SOTON/WP00880](https://doi.org/10.5258/SOTON/WP00880).
2. GRID3. *Geospatial data Nigeria* (catalogue, October 2026): https://grid3.org/geospatial-data-nigeria
3. GRID3. *Data Hub*: https://data.grid3.org/
4. WorldPop. *Nigeria age and sex structures*: https://hub.worldpop.org/geodata/summary?id=82494
5. GRID3. *Piloting the Geospatial Microplanning Toolkit in Kano and Kaduna*: https://grid3.org/publications/piloting-the-gmt-in-nigeria-insights-from-kano-and-kaduna

**Next concrete action:** Download the two WorldPop ADM ZIP files and inspect their real schemas. Do not start implementation against hypothetical field names.
