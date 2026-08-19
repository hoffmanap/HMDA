# El Paso, Underwritten
### An HMDA loan-level dashboard for El Paso County, TX (2007–2025)

A self-contained, single-file HTML dashboard built from raw Home Mortgage Disclosure Act (HMDA) Loan/Application Register (LAR) data, in the pudding.cool "highlighter" visual style.

**[Open the dashboard → `https://hoffmanap.github.io/HMDA/`]**

---

## What this is

Every mortgage application filed in El Paso County generates a public HMDA record: who applied, how much they asked to borrow, what the property was worth, and whether the lender said yes. This project harmonizes **547,755 such records across 17 filing years** (2007–2013, 2015, 2017–2025; only 2014 and 2016 are missing) into one dashboard covering:

- **Trends over time**: applications, denial rate, median loan amount, 2007-2025
- **A filterable explorer**: loan purpose mix or denial rate by race, any combination of years
- **A persistent gap**: denial-rate differences by race across the full period, with caveats about small-sample categories and the "Race Not Available" code
- **Loan type mix**: Conventional, FHA, VA, and FSA/RHS shares by year, including El Paso's distinctly high VA loan share (tied to Fort Bliss) and FHA's role during and after the 2008-2012 credit crunch
- **Denial reasons**: which stated reasons (credit history, debt-to-income ratio, collateral, etc.) drive denials, and how that mix has shifted over time
- **Affordability**: median property value vs. median applicant income, 2018 onward
- **Who applies vs. who lives here**: a beeswarm comparing the income distribution of recent applicants (split by outcome and by ethnicity) against El Paso County's actual median household income and demographics, from the American Community Survey
- **Tract-by-tract detail**: all 240 El Paso census tracts, as a sortable bivariate-encoded table, with a **map** driven by the same tract boundary files described below

The dashboard is a single HTML file with all aggregated data **and both charting/mapping libraries (Chart.js, Leaflet) embedded inline**; it works fully offline, with no server, no API calls, and no CDN or external network dependency of any kind. Opening the file is enough; nothing needs to load from the internet except the optional Google Fonts link, which is purely cosmetic and the page still renders correctly without it.

---

## Data sources

| Years | Format | Notes |
|---|---|---|
| 2007–2013, 2015, 2017 | Legacy HMDA LAR schema (78–79 cols) | Statewide Texas files, filtered here to El Paso County (FIPS 48141) |
| 2018–2019, 2020, 2021–2025 | Modern HMDA LAR schema (104–108 cols, post-2017 rule) | Pre-filtered to El Paso County. 2020 arrived as an `.xlsx` file rather than `.csv`, but uses this same modern schema, not the legacy one |
| **2014, 2016** | **Not currently in this archive** | Appear as gaps in trend lines, not a market disruption, just missing source files |

Raw source: [FFIEC/CFPB HMDA Data Browser](https://ffiec.cfpb.gov/data-browser/)

### Known schema differences, harmonized in the pipeline
- **Property value**: not collected before 2018. Pre-2018 years show `null` for this field. Loan amount is *not* treated as a substitute, since down payments and refinance balances vary too widely to proxy home price reliably.
- **Race/ethnicity**: legacy years report a single applicant race/ethnicity code; modern years use CFPB's `derived_race`/`derived_ethnicity`, which can reflect joint applications (e.g., "Joint," "Two or more minority races"). These are harmonized to a common label set but are not perfectly equivalent across eras.
- **Loan purpose**: "Refinance" combines legacy code `3` with modern codes `31` (rate/term refinance) and `32` (cash-out refinance).
- **Census tract ID**: legacy files store a short tract number (e.g., `40.04`); this is converted to the full 11-digit GEOID (`48141004004`) to match the modern format.
- **Denial rate**: denied applications ÷ all *decisioned* applications (originated, approved-not-accepted, denied, or preapproval outcomes). Withdrawn, incomplete, and purchased-loan records are excluded from the denominator.
- **Small-sample suppression**: race/ethnicity breakdowns with fewer than 5 records in a given year are dropped from that year's `by_race` breakdown to avoid unstable rates.

---

## Completing the map: tract boundary files

The tract explorer table works today with no further setup. The **choropleth map** needs one more ingredient the pipeline can't fetch on its own: geographic boundary polygons for El Paso County's census tracts.

Census tract lines are redrawn after each decennial census, so this dashboard needs **three vintages**, matched to the years that used them:

| Vintage | Covers these data years | Download (Texas statewide, ZIP shapefile) |
|---|---|---|
| 2000 tracts | 2007–2011 | `https://www2.census.gov/geo/tiger/PREVGENZ/tr/tr00shp/tr48_d00_shp.zip` |
| 2010 tracts | 2012, 2018–2021 | `https://www2.census.gov/geo/tiger/GENZ2019/shp/cb_2019_48_tract_500k.zip` |
| 2020 tracts | 2022–2025 | `https://www2.census.gov/geo/tiger/GENZ2020/shp/cb_2020_48_tract_500k.zip` |

Census distributes these as shapefiles, not GeoJSON, so there is one extra conversion step, no GIS software required:

1. Download the `.zip` for a vintage
2. Go to **[mapshaper.org](https://mapshaper.org)** and drag the zip onto the page
3. In the console, clip to El Paso County only: `-filter 'COUNTYFP=="141"'`
4. **Export → GeoJSON**
5. Save the result as `tracts_2000.geojson`, `tracts_2010.geojson`, or `tracts_2020.geojson` (matching the vintage) in the same folder as `index.html`

Repeat for all three vintages. The dashboard checks for these files automatically on load (`fetch('tracts_2000.geojson')`, etc.); whichever are present light up, and missing ones silently fall back to the table view, so this can be done incrementally.

---

## File structure

```
index.html            the dashboard (self-contained, data embedded inline)
README.md             this file
tracts_2000.geojson   (optional, user-supplied) 2000-vintage tract boundaries, El Paso County
tracts_2010.geojson   (optional, user-supplied) 2010-vintage tract boundaries, El Paso County
tracts_2020.geojson   (optional, user-supplied) 2020-vintage tract boundaries, El Paso County
```

---

## Regenerating the data

The embedded data was built from raw LAR files via a two-stage pipeline (harmonize → aggregate). If new years of source data become available:

1. **Harmonize**: read each raw file (legacy `.xlsx` or modern `.csv`), map to a common field set, filter to El Paso County (FIPS 48141), and convert tract numbers to full GEOIDs where needed.
2. **Aggregate**: compute year-level summaries (`yearly.json`) and tract-year summaries (`tract_year.json`, with count, median loan amount, median property value, median income, denial rate, and tract demographic snapshot per tract per year).
3. Re-embed both JSON outputs into `index.html` in place of the existing data blocks.

Row-level detail is intentionally *not* retained in the shipped dashboard, only the aggregates needed to drive the charts and table, keeping the file small and fast to load.

---

## External benchmarks

The "who applies vs. who lives here" section compares HMDA applicants against county-level figures that do not come from HMDA:

- Median household income ($59,888) and Hispanic/Latino population share (82.7%): U.S. Census Bureau, American Community Survey, 2024 estimates for El Paso County, TX
- Veteran population share (7.9%): same source

These are population-level statistics, not loan-level data, and are not embedded as raw records, only as fixed reference values used in the narrative and as a marker line on the beeswarm chart.

## Limitations

- **Not a causal analysis.** Denial rate differences by race, income, or geography reflect the raw HMDA record only; they do not control for credit score, debt-to-income ratio, loan-to-value ratio, or other underwriting factors also captured (inconsistently) in the source data. Read disparities as *patterns worth investigating*, not conclusions about discrimination or its absence.
- **Small-sample volatility.** Some tracts and race categories have very few applications in a given year; rates built on small denominators swing widely and are flagged accordingly where possible, but always sanity-check the underlying count before drawing conclusions from a single cell.
- **2014 and 2016 are gaps**, not zeros. The trend resumes on either side.
- **"Race Not Available"** is largely a data-quality artifact (common for loans purchased from another originator, which don't carry the original applicant demographics); treat it as missing data, not as its own demographic group.
