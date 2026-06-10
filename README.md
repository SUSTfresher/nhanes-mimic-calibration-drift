# Calibration drift and local recalibration of transported diabetes mortality models

This repository contains the analysis code for a dual-database study of diabetes
one-year mortality model transportability, calibration drift, local recalibration,
and decision-curve utility using NHANES and MIMIC-IV.

## Data access

This repository does not include raw or processed data.

- NHANES data are publicly available from the US National Center for Health
  Statistics: https://www.cdc.gov/nchs/nhanes/
- NHANES linked mortality files are available from the NCHS data linkage program:
  https://www.cdc.gov/nchs/linked-data/mortality-files/index.html
- MIMIC-IV v3.1 is available from PhysioNet to credentialed users who complete
  the required training and data-use agreement:
  https://physionet.org/content/mimiciv/3.1/

The scripts expect a processed NHANES analysis table at:

```text
data/nhanes/processed/nhanes_2005_2018_diabetes_ckd_mortality_scan.csv
```

MIMIC-IV source files can be made available in either of two ways:

```powershell
$env:MIMIC_ROOT = "replace-with-local-mimic-iv-3.1-path"
```

or by placing the credentialed local copy under:

```text
data/mimic-iv-3.1/
```

Do not commit MIMIC-IV source files, MIMIC-derived processed data, model
artifacts, or generated intermediate tables.

## Environment

Python dependencies are listed in `requirements.txt`.

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install -r requirements.txt
```

R figure scripts require the packages listed in `environment_r.txt`. After
installing the R packages, run `sessionInfo()` and replace `environment_r.txt`
with the exact session output before publication.

## Recommended analysis order

Run commands from the repository root.

### 1. Inventory and MIMIC preprocessing

```powershell
python scripts/local_data_inventory.py
python scripts/build_mimic_diabetes_light_cohort.py
python scripts/extract_mimic_labs_24h_duckdb.py
python scripts/extract_mimic_labs_window_duckdb.py --start-hours -24 --end-hours 24
python scripts/compare_mimic_lab_windows.py
python scripts/merge_existing_mimic_labs.py
```

### 2. Domain shift and transportability analyses

```powershell
python scripts/domain_shift_nhanes_mimic.py
python scripts/source_classifier_roc.py
python scripts/nhanes_mimic_oneyear_mortality_transport.py
python scripts/bootstrap_transportability_ci.py
```

Optional background analyses:

```powershell
python scripts/mimic_inhospital_mortality_model.py
python scripts/nhanes_mimic_transportability_stress_test.py
```

### 3. Local recalibration, subgroup, and decision utility analyses

```powershell
python scripts/local_recalibration_simulation.py
python scripts/local_recalibration_by_event_count.py
python scripts/recalibration_uncertainty_intervals.py
python scripts/calibration_slope_intercept_summary.py
python scripts/subgroup_transportability_analysis.py
python scripts/decision_curve_transportability.py
python scripts/feature_signal_stability.py
```

### 4. Manuscript tables and figures

```powershell
python scripts/make_manuscript_tables.py
python scripts/make_feasibility_figures.py
python scripts/make_nature_style_main_figure_preview.py
python scripts/make_calibration_landscape_preview.py
```

R figure previews:

```powershell
Rscript r_scripts/make_r_nature_style_2d_figure.R
Rscript r_scripts/make_r_recalibration_event_count_figure.R
Rscript r_scripts/make_npj_main_figures.R
```

## Main outputs

The main manuscript tables are generated under:

```text
outputs/manuscript_tables/
```

The first-pass figure set is generated under:

```text
outputs/figures_feasibility/
```

Nature-style figure previews are generated under:

```text
outputs/nature_style_main_figure_preview/
outputs/r_nature_style_figure_preview/
outputs/npj_main_figures/
```

## Reproducibility notes

- All Python scripts use repository-relative paths through `PROJECT_ROOT`.
- Scripts that read MIMIC-IV source files use `MIMIC_ROOT`; set this environment
  variable if the MIMIC-IV copy is outside the repository.
- No MIMIC-IV source data or MIMIC-derived outputs should be committed.
- The final public repository should include a working GitHub or Zenodo URL in
  the manuscript Code availability statement.
