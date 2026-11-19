# SNODAS vs. SNOTEL Snow Water Equivalent (2004–2024)

MSU Spring 2026 Data Science term project. It compares modeled snow water equivalent (SWE)
from NOAA's **SNODAS** product with ground observations from an NRCS **SNOTEL** station
(at about 45.4° N, 110.95° W, southwest Montana). Both cover March 1 – June 30 of every year from 2004 to 2024.

The main question: how well do SNODAS's annual **peak SWE** and **peak timing (day of year)** match the SNOTEL station?

## Key results

| Metric | Peak SWE (m) | Peak DOY (days) |
|---|---|---|
| Mean bias (SNODAS − SNOTEL) | 0.014 | 6 |
| MAE | 0.088 | 8 |
| RMSE | 0.111 | 11.7 |
| OLS R² (SNOTEL ~ SNODAS) | 0.49 | 0.56 |

On daily values (2,557 matched days), the Pearson correlation between SNODAS and SNOTEL is **0.89**.
Figures and the full statistics are in [`notebooks/04_DataAnalysis.ipynb`](notebooks/04_DataAnalysis.ipynb).

## Repository layout

```
.
├── notebooks/
│   ├── 01_SNODAS_DataProcessing.ipynb    # download SNODAS rasters, extract SWE at the station point
│   ├── 02_SNOTEL_DataProcessing.ipynb    # clean SNOTEL export, convert in → m, annual peaks
│   ├── 03_Combined_DataProcessing.ipynb  # merge both sources, QA, fill missing dates, add peak DOY
│   └── 04_DataAnalysis.ipynb             # peaks, correlation, error metrics, regression, trends, figures
├── data/
│   ├── SNODAS/   # SNODAS_swe_<year>.csv, SNODAS_swe_all.csv, SNODAS_swe_missingdates.csv
│   ├── SNOTEL/   # raw + cleaned SNOTEL SWE, annual SNOTEL peaks
│   └── MERGED/   # swe_merged.csv (daily, both sources), peaks_swe.csv (annual peaks)
├── results/      # created when you run 04 (CSV stats + figures/); not tracked in git
├── requirements.txt
└── README.md
```

## Data sources

- **SNODAS**: NSIDC G02158, masked daily SWE grids
  (<https://noaadata.apps.nsidc.org/NOAA/G02158/masked/>). Notebook 01 downloads each daily
  tarball, pulls out the SWE layer (product code `1034`), converts it to GeoTIFF, samples the station point,
  and then deletes the intermediate files. Only the extracted point values are kept.
- **SNOTEL**: NRCS Report Generator 2.0 (<https://wcc.sc.egov.usda.gov/reportGenerator/>),
  daily SWE exported as CSV into `data/SNOTEL/SNOTEL_swe_20042024.csv`.

All processed CSVs are already in `data/`, so you can run notebooks 03 and 04 without downloading anything.

## Running it

```bash
python -m venv .venv
.venv\Scripts\activate            # Windows  (macOS/Linux: source .venv/bin/activate)
pip install -r requirements.txt
jupyter lab
```

Open the notebooks **from the `notebooks/` folder**. All paths are relative to it: the notebooks
`chdir` into `../data` and write their outputs to `../results`.

Suggested order: `01` → `02` → `03` → `04`. One cross-dependency: the last cell of `03` adds
day-of-year columns to `data/MERGED/peaks_swe.csv`, and that file is created in `04`. So on a fresh run,
run `04`'s peak-table cell first, or just use the `peaks_swe.csv` already in the repo.

Notebook 01 downloads about 2,500 daily SNODAS archives and takes a long time. Skip it unless you need
to rebuild the SNODAS series.
