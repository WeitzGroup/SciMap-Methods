---
editor_options: 
  markdown: 
    wrap: 72
---

# SciMap-Methods

Code to accompany the manuscript "Economic Loss due to Health Funding
Cuts as Distributed Across Geospatial Units" bv Mallory J Harris, Alyssa
H Sinclair, Clio Andris, and Joshua S Weitz.

The two scripts used to conduct all analyses NIH_impacts.Rmd (estimate
county and district-level economic losses from IDC cap and terminated
grants) and NIH_analysis.Rmd (used to generate figures and calculate all
results reported in the manuscript).

Input data used in the analysis are in the data folder, outputs csvs are
in the output folder, and output figures are in the figs folder. More
detailed descriptions follow. Code developed for R version 4.4.1. Each
script should take 1-5 minutes to run.

## Project Structure

```
├── data/                                   # Raw data
│   ├── 2023-ACP-Typology.csv
│   ├── CD_pop2024.csv
│   ├── county_gdp2024.csv
│   ├── county_pop.csv
│   ├── geoid_dictionary_July4.csv
│   ├── JHU_wTract.csv
│   ├── UCLA_wTract.csv
│   ├── Tract_work_County_home_JHUpts.csv
│   ├── Tract_work_County_home_UCLApts.csv
│   ├── JT00_mostrecent
│   ├── org_names_corrected.csv
│   ├── RAC_countyAll.csv
│   ├── Ruralurbancontinuumcodes2023.csv    
│   ├── WAC_tract.csv
│   ├── repeated_orgs.csv
│   ├── state-multiplier-urm.csv
│   ├── state_abbrev.csv
│   ├── state_and_county_fips_master.csv
│   ├── state_fips_master.csv
│   ├── tract_dictionary.csv
│   └── us_representatives_119th_congress.csv
├── output/                                 # Processed data from NIH_impacts.Rmd        
│   ├── CD_partisanship.tex                 
│   ├── NIH_clean_fips.csv                  
│   ├── NIH_impact_cong.csv                 
│   ├── NIH_impact_county-22.csv            
│   ├── NIH_impact_county-ctc.csv
│   ├── NIH_impact_county-natlmult.csv
│   ├── NIH_impact_county-sens.csv
│   ├── NIH_impact_county.csv
│   ├── NIH_impact_county_broad.csv
│   ├── NIH_impact_county_narrow.csv
│   ├── NIH_impact_state.csv
│   ├── comp_natlmult_IDC_FIPS.csv
│   ├── county_commute_NIH_inst.csv
│   ├── county_urb.tex
│   ├── recent_flows.csv
│   ├── raw_loss.csv
│   ├── static_loss_cong.csv
│   ├── static_loss_county.csv
│   ├── static_vs_commuter-CD.csv
│   ├── static_vs_commuter_IDC_FIPS.csv
│   ├── static_vs_commuter_term_FIPS.csv
│   └── terminated_points.csv
├── NIH_analysis.Rmd                        # Generate figures and tables
├── NIH_impacts.Rmd                         # Data processing
└── README.md
```

## Data

-   2023-ACP-Typology - county FIPS and ACP typology category
-   county_gdp2024 - GDP at the county level
-   county_pop - county FIPS, state name, county name, and population
    size for each county (currently used to match county FIPS to name
    and state)
-   fips_dictionary - lists the county FIPS code of all lat/lon pairs
-   geoid_dictionary_July4 - lists the Congressional District GEOID
    (New119Fips) and district name (NAMELSAD) of all lat/lon pairs
-   JT_congress_July30 - commuter flows at the district level. COMMUTES
    reports the number of people commuting from Cong_ORIGIN GEOID to
    Cong_DESTINATION GEOID.
-   NIH_prior - grant information from NIH RePORTER for grants active
    for several years prior to FY2024 (used to geolocate terminated
    grants)
-   NIH_raw - grant information from NIH RePORTER for grants active in
    FY2024 (used to estimate direct and indirect costs)
-   nih_terminations_airtable - grant terminations compiled by Grant
    Witness (as of August 19, 2025)
-   OD_countySum001_2016 - as above, commuter flows at the county level.
-   org_names_corrected - helper file to correct capitalization and
    abbreviations in names for terminated grants. Generated manually,
    does not affect this analysis.
-   repeated_orgs - helper file to merge suborganizations of a single
    organization in the same county and congressional district that.
    Generated manually, does not affect this analysis.
-   Ruralurbancontinuumcodes2023 - metro/non-metro classificationof
    counties.
-   state_abbrev - state names and their two-letter postal abbreviations
-   state_and_county_fips_master - dictionary of county FIPS code,
    county name, and state name
-   state_fips_master - dictionary of county FIPS code, county name, and
    state name (only used to construct state_FIPS \> state name mapping)
-   state_multiplier-urm - breakdown of NIH awards and economic activity
    generated at state level (from URM)
-   us_representatives_119th_congress - representative name and party
    affiliation by district
-   JHU/UCLA_wTract - tract assignment for each satellite location for JHU and UCLA
-   Tract_work_County_home_JHU/UCLApts - tract-to-county flows for each tract containing a satellite JHU or UCLA location

## Output

The main output files are NIH_impact_cong, NIH_impact_county, and
NIH_impact_state, which report losses at different geographic scales.
Detailed descriptions of the fields within those files follow.

Column names are constructed as follows:

**[loss source]** **[loss type] [self-reporting?] [log scale?]**

loss source:

-   terminated: grants that have been terminated according to
    [grant-watch.us](https://grant-watch.us/nih-data.html)
-   IDC: estimated losses due to the proposed cap on indirect costs,
    based on FY2024 funding
-   grant_funds: total grant funding for FY2024
-   overlap: indirect costs associated with a terminated grant (this is
    an overlap between the two loss sources, avoids double-counting in
    estimating total losses)
-   combined: losses associated with both the proposed cap on indirect
    costs and terminated grants (accounting for potential overlap)

loss type:

-   loss: raw financial loss
-   econ_loss: economic loss, calculated by applying \$2.56 economic
    multiplier estimated by United for Medical Research
    ([UMR](https://www.unitedformedicalresearch.org/annual-economic-report/))
-   job_loss: job loss, estimated by applying multiplier estimated from
    UMR report

self-reporting

-   no_self: column names containing "noself," indicate that
    self-reported terminations were excluded from calculations (i.e.,
    only terminations reported by governmental sources are considered).
    Otherwise, self-reported terminations are included (only relevant
    for terminated, overlap, and combined columns).

log-scale

-   log: column names with the suffix "log" are log-scaled for
    visualization purposes.

All three files contain the following geographic identifiers: state
(full state name), state_code (state abbreviation), and state_FIPS
(two-digits state FIPS code). Additional identifiers for Congressional
Districts are GEOID (numeric ID for congressional districts), rep_name
(name of Congressional Representative), and pol_party (political party
of Congressional Representative). Additional identifiers for counties
are FIPS (five-digit county FIPS code, note that leading zeroes should
be present in some cases), and name (county name).

### Additional output files

Intermediate outputs are also provided. They are: losses estimated under
static models at the county and district level (static_loss_cong,
static_loss_county); losses from terminated grants at the point
(institution)-level (terminated_points); losses from the IDC cap at the
grant level (NIH_clean_fips); flows of economic losses between county
origin-destination pairs (county_commute_NIH); and flows of economic
losses into counties, districts, and states based on commuter flows
(NIH_impact_county, NIH_impact_cong, NIH_impact_state); comparisons of
static versus commuter estimates for counties and districts
(static_vs_commuter, static_vs_commuter-CD); and sensitivity analyses of
county-level estimates using state-specific economic multipliers or
varying commuter influence $\rho$ (NIH_impact_county-diffmult,
NIH_impact_county-sens), raw losses estimated without applying a multiplier (raw_loss), and economic losses estimate using the most recent commuter flows (recent_flows).

## Figs

Folder contains outputs relevant to manuscript figures. Files are:
institutions located within a given focal county (fig1-points, for
Figure 1), economic losses in surrounding counties originating from a
given focal county (fig1-vals, for Figure 1), the pdf output of Figure
S2 (figs2), the pdf output of Figure 3 (fig3), a latex intermediate for
Table 1, and supplemental outputs excluding economic loss maps.