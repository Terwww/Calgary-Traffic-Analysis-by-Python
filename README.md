# Calgary-Traffic-Analysis-by-Python
End-to-end Python analysis of Calgary traffic incidents (2016-2026) joined with weather: data audit, cleaning, statistics, predictive modelling, resource-allocation ranking and Power BI exports.

# Calgary Traffic Incident Analysis

End-to-end analysis of City of Calgary traffic-incident open data (Dec 2016 to Sep 2026) joined with daily weather. The project covers data auditing, cleaning, exploratory analysis, statistical testing, predictive modelling and a resource-allocation ranking, and exports clean tables for a Power BI dashboard.


## Why this project

Operational teams need to know where and when incidents cluster, which incidents stay open longest, and whether weather really matters. This project answers those questions and, just as importantly, documents what the data **cannot** support.

## Business questions

1. Where and when do incidents happen, and which quadrants and time windows carry the highest workload?
2. Which incidents are *prolonged* (open longer than 60 minutes), and which factors are associated with that?
3. How much does weather explain daily incident volumes once season and weekday are controlled for?
4. Which quadrant and time-block pairs could benefit most from temporary support, and from where?

## Data

| Source | Content | Notes |
|---|---|---|
| Open Data Calgary: Traffic Incidents | ~64,600 incident records with start/last-update time (UTC), location and description | Snapshot taken Sept 2026 |
| Daily weather (Environment Canada data via weatherstats-style export) | Temperature, wind, visibility, precipitation, snow | Snow and precipitation are only recorded for ~630 days |

*Add the data licence and attribution statements required by each source here.*

## Data quality issues found (and how they were handled)

| Issue | Evidence | Handling |
|---|---|---|
| Timestamps are in UTC | 26.3% of incidents fall on a different calendar day in Calgary local time | Converted to `America/Edmonton` before any hourly or daily analysis |
| Reporting regime changed | Last-update time and quadrant are missing for 69%, 100% and 73% of records in 2019, 2020 and 2021 | Duration analysis uses 2022 onward only |
| Missing and partial months | 6 of 118 months flagged (2016-12, 2017-01, 2019-07, 2019-08, 2025-06, 2026-09) | Excluded from trend and weather analysis; zero days in these months are data gaps, not safe days |
| Quadrant missing in ~22% of rows | The quadrant suffix in the location text matches the official column in 94.7% of overlapping rows | Filled from location text, then from a spatial k-NN model (92.9% cross-validated accuracy) |
| "EMS" keyword is not overall EMS demand | The keyword appears almost only in pedestrian (97%) and cyclist (96%) incident templates, and not before 2021 | Treated as a vulnerable-road-user indicator, not an EMS-demand measure |
| Duration is a proxy | Duration = last update minus start, not confirmed clearance | Labelled as a proxy everywhere; invalid values (negative or > 12 h) excluded |

## Methods

- **Cleaning and features:** duplicate removal, local-time conversion, incident-type classification from free text, cyclic hour encoding, 2-hour blocks, season, weather join on local date.
- **Exploratory analysis:** monthly trend with gaps highlighted, weekday x hour heatmap, quadrant profiles with Wilson confidence intervals, hotspot grid and density map, weather bands.
- **Statistics:** Welch t-test on log duration, Mann-Whitney, chi-square with effect sizes (Cohen's d, Cramér's V), Holm-corrected pairwise tests, partial correlations controlling for month and weekday, logistic regression (odds ratios), Poisson regression with robust errors (incidence-rate ratios), per-quadrant OLS.
- **Prediction:** chronological split (train up to 2024, test after), compared against a naive baseline; permutation importance and calibration checks.
- **Resource ranking:** workload proxy (average simultaneously open incidents) combined with a shrunk prolonged-incident rate, distance between quadrant centroids, feasibility rules and a sensitivity analysis.

## Key findings

*Numbers below are from the dataset snapshot above. Re-run to reproduce.*

- **When:** the busiest hour is **Wednesday 16:00**, averaging about 2.5 incidents per hour.
- **Where:** the largest hotspot is around **Memorial Drive and Deerfoot Trail SE** (469 incidents in the analysis window).
- **Quadrants:** NW has the highest prolonged-incident rate (30.8%, 95% CI 29.8% to 31.9%) versus 27.5% in SW. The effect is small (Cohen's d = 0.06, Cramér's V = 0.024), so the difference is statistically detectable but operationally modest.
- **Weather:** the raw temperature correlation with daily incidents is r = -0.21 (Spearman -0.05). After controlling for month and weekday it is r = -0.32. A few extreme days drive the linear fit, so it should not be read as a stable linear effect. On snow days there were about 28.4 incidents per day versus 20.1 on other days (unadjusted for season; snow data covers recent years only).
- **Prediction is weak:** no model beat a naive baseline for predicting incident duration (R² of about 0 on the test period). The prolonged-incident classifier reaches ROC AUC 0.60; the highest-risk 10% of incidents are prolonged 49% of the time versus a 35% base rate.
- **Resource ranking:** the highest-priority temporary move is SW to SE during 14:00 to 16:00. This is a proxy-based ranking built on incident workload, not a validated staffing plan.

## Limitations

- Duration is a last-update proxy, not true clearance time, and cannot be predicted from time, place, type and weather alone.
- Data before 2022 is unreliable for severity analysis.
- Quadrant shares are raw counts, not risk. Normalising by traffic volume or road length would need data not in this dataset.
- The resource-allocation ranking has no unit-availability or utilisation data behind it and has not been validated.
- Results are observational associations, not causal effects.

## Power BI

The script exports three tables to `outputs/powerbi/`: `fact_incidents.csv`, `fact_daily_weather.csv` and `dim_date.csv`, ready for a star-schema model. *(Add dashboard screenshots or PDF here.)*

## How to run

**Locally**
```bash
pip install -r requirements.txt
python calgary_traffic_incident_analysis.py --incidents data/Traffic_Incidents_analysis.xlsx
```

**Google Colab**
1. Upload the Excel file and save the script as `analysis.py`.
2. Run `!python analysis.py --incidents Traffic_Incidents_analysis.xlsx --out-dir outputs`.

Outputs go to `outputs/`: `figures/`, `tables/`, `powerbi/`, `key_findings.md` and `run.log`.

## Project history

This started as an Excel dashboard. Rebuilding it in Python exposed a COUNTIFS range that was not locked with `$`, which undercounted daily incidents and gave a correlation of -0.20 versus roughly -0.22 to -0.23 when calculated correctly. The Python version audits those issues explicitly.

## Tools

Python (pandas, NumPy, SciPy, statsmodels, scikit-learn, matplotlib, seaborn), Excel, Power BI.
