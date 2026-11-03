# DTP coverage and zero-dose children in Nigeria

A worked example for [Fieldwork](https://github.com/digital-twin-ph/fieldwork): five questions about 774 Local
Government Areas (LGAs), built from an empty canvas.

**These are modelled estimates**, not administrative counts or vaccination records. They are for
exploring questions, and do not grade any LGA or programme.

## The questions

| Workflow | Nodes | What it shows |
| --- | --- | --- |
| Where is DTP1 coverage lowest? | 4 | Modelled DTP1 coverage, 2024 |
| Where are the most zero-dose children? | 4 | Estimated children with no DTP1, 2024 |
| What changed from 2005 to 2024? | 6 | Change in coverage over 20 years, in proportion points |
| Which LGAs have low coverage and a high burden? | 7 | A rule over both values |
| Where is coverage lowest, on the ground? | 4 | DTP1 coverage on an interactive map, over OpenStreetMap tiles |

Coverage and burden are different questions: a populous LGA can have good coverage and many zero-dose
children, and a small one the reverse.

The rule in the fourth workflow is an **illustration**: DTP1 below 0.7 and more than 5,000 zero-dose
children. It marks 127 LGAs. **The thresholds are arbitrary and are not programme policy.** The result is a
list for data review.

## Files

| File | What it holds |
| --- | --- |
| `data/nga-lga-2024.csv` | One row per LGA: mean, low and up for DTP1, DTP3 and MCV1, and `zd_est`, for 2024 |
| `data/nga-lga-dtp1-by-year.csv` | Mean DTP1 for each year from 2005 to 2024 |
| `data/nga-lga-zero-dose-by-year.csv` | `zd_est` for each year from 2005 to 2024, not yet used by a workflow |
| `geo/nga-lgas.geojson` | The 774 boundaries, with `ID`, `shapeName` and `shapeID` |
| `notes/report-notes.json` | The authored notes of the illustrative report, and the works each one cites |
| `fieldwork-project.json.gz` | The Fieldwork project |
| `FRAMEWORK_Nigeria_Immunization_Worked_Example.md` | The original plan, kept as written. Its field names were guesses |

## What was changed from the source

- The last 20 of the 25 years were kept, 2005 to 2024, and the two archives were joined on `ID` and `Year`. They agree on every key.
- `sd` and `median` were left out. The values kept are unchanged.
- Geometry appears on every row of the source. It is carried once, from 2024, with coordinates rounded to 6 decimal places, about 0.1 m.

## Reading the data

- **Join on `ID`, never on the name.** There are 769 names for 774 LGAs: Surulere, Ifelodun, Irepodun, Bassa and Obi each name two places.
- **Coverage runs from 0 to 1.**
- **In the source's LGA zero-dose file, the column called `State` holds the LGA name.**
- **`zd_est` has no uncertainty at LGA level, and no population.**
- **LGA zero-dose counts sum to within 0.11% of the national figure** in 2018, 2021 and 2024. The difference is not explained here.

## The basemap

The fifth workflow draws over an OpenStreetMap extract, read from [Protomaps' PMTiles archive](https://data.source.coop/protomaps/openstreetmap/v4.pmtiles).
The project holds the source, boundary, zoom range and build date, not the tiles. On a new device, connect,
select the map node and choose **Import tiles**. At zoom 10 that was 1,344 tiles and 49.2 MB, build of 2026-03-31.
Map data © OpenStreetMap contributors, ODbL. The tiles are context and no value depends on them.

## What this does not establish

- **Uncertainty is not on these maps.** Each coverage estimate has an interval, and for 22 of the 127 LGAs on the priority map that interval crosses 0.7. Read `low_dtp1` and `up_dtp1` before reading anything into one LGA.
- **The model used covariates, and distance to urban areas is one.** It is significant and negative for DTP1, so a link between low coverage and remoteness can partly reflect the model's inputs. Travel time to a facility is not a covariate. The others include wealth, maternal education, skilled birth attendance and climate. Read from the authors' [DTP1 code](https://github.com/EdsonUtazi/NGA_paper_space_time_analysis) and [preprint](https://www.medrxiv.org/content/10.64898/2026.01.19.26344414v1.full), which is not peer reviewed.
- **Values are by birth cohort, not calendar year,** and the model smooths strongly over time. A campaign in one year should not be read as a step in that year.
- **No causes.** Nothing here says why coverage is low in any place. These are ecological summaries.
- **No persistence, facilities, access or measles.** The plan lists them. They are not built.
- **The boundaries are the source's, and may not match the official LGA boundaries** of any given year.

Fieldwork declares no study design for this project.

## An illustrative report, 2005 to 2024

The project carries a write-up, as if this example were reporting on the last 20 years. Open **Notes** and
choose **Run the rest** to assemble it.

- **Authored notes** are in `notes/report-notes.json`, and are entered through the note dialog.
- **Machined notes** are written by the widgets on each run.
- **References** are assembled from the works the notes cite, each entered in the note's Citations tab.

It is an illustration, not a programme assessment.

## References

- Utazi CE, Megheib M, Olowe ID, Chaudhuri S, Tejedor-Garavito N, Mwinnyaa G, Kawakatsu Y, Boyda D, Lorin J, Tatem AJ. Mapping DTP1,3 and MCV1 coverage and zero-dose prevalence in Nigeria: a spatiotemporal analysis (2000-2024). medRxiv preprint, not peer reviewed, 2026, with supplementary information. [doi:10.64898/2026.01.19.26344414](https://doi.org/10.64898/2026.01.19.26344414)
- Utazi CE, Aheto JMK, Wigley A, et al. Mapping the distribution of zero-dose children to assess the performance of vaccine delivery strategies and their relationships with measles incidence in Nigeria. *Vaccine*. 2022, corrected proof. [doi:10.1016/j.vaccine.2022.11.026](https://doi.org/10.1016/j.vaccine.2022.11.026)
- Masresha BG, Weldegebriel GG, Lebo EJ, Wanyoike S, Bwaka AM, Vuo-Masembe Y. Insights from 25 years of measles and measles-rubella vaccination campaigns in the WHO African Region (2001-2025). *Vaccines*. 2026;14(6):549. [doi:10.3390/vaccines14060549](https://doi.org/10.3390/vaccines14060549)
- National Population Commission (NPC) [Nigeria] and ICF. *Nigeria Demographic and Health Survey 2018*. Abuja, Nigeria, and Rockville, Maryland, USA: NPC and ICF; 2019.
- Croft TN, Allen CK, Zachary BW, et al. *Guide to DHS Statistics*. Rockville, Maryland, USA: ICF; 2023.
- World Health Organization. Measles and rubella supplementary immunization activities, Nigeria (`V_SIA_MAIN_MR`). Immunization Data portal, accessed 2 November 2026. Figures reported by Nigeria. Cited, not held here.

## Source

Utazi CE, Megheib M, Olowe ID, Chaudhuri S, Tejedor-Garavito N, Tatem AJ. *Modelled vaccination coverage and
zero-dose estimates for Nigeria, 2000-2024*, version 1.0. WorldPop, University of Southampton, 2026.
[doi:10.5258/SOTON/WP00880](https://doi.org/10.5258/SOTON/WP00880). Licensed CC BY 4.0.

Source archives and their digests are in `provenance.json`.
