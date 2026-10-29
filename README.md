# DAT status, SAA kinetics and clinical progression in prodromal PPMI participants

Analysis code for linear mixed models of motor and cognitive progression in the PPMI prodromal cohort, by baseline dopamine transporter (DAT) imaging status and by baseline SAA kinetic parameters.

This branch (`dat-groups`) is the same pipeline as `main`, with one change: participants are grouped by **baseline DAT status** instead of baseline SAA status. DAT-positive = abnormal dopaminergic imaging, i.e. visit-level DAT SBR status with an age/sex-expected DaT ratio < 0.75, taken from the curated BL row. SAA status and SAA kinetics are kept as descriptive columns; the kinetics analyses run in DAT-positive participants (instead of SAA-positive).

## Notebooks

Run in order. Each notebook is self-contained and works in Google Colab or locally.

| Notebook | What it does |
|---|---|
| `01_build_dataset.ipynb` | Builds a baseline-anchored analytic dataset from the PPMI curated data cut and the raw SAA files: one wide file (one row per participant) and one long file (one row per visit). DAT status comes from the curated DAT column (`DAT_COL`) on the BL row; participants without it are excluded. SAA status (curated `CSFSAA`, codes 1 and 3 = positive, 0 = negative) is kept, blank if inconclusive or missing. Kinetics come from the baseline visit only. |
| `02_lmm_dat_status.ipynb` | `outcome ~ time × DAT status + covariates + (1 + time \| participant)`, DAT-negative as reference, in all prodromal participants. |
| `03_lmm_saa_kinetics_dat_positive.ipynb` | DAT-positive participants only. Same model with a median-split kinetic group (Fmax, TTT, AUC; Project 237, 24-hour assay) as the exposure. |
| `04_table1_baseline.ipynb` | Table 1: baseline characteristics of DAT-positive vs DAT-negative participants, including baseline serum uric acid, glucose and creatinine, prodromal subgroups and CSF SAA assay. Mann–Whitney U, χ², Fisher's exact or permutation χ²; Holm correction. |
| `05_survival_dat_status_saa_kinetics.ipynb` | Phenoconversion-free survival: Kaplan–Meier curves and Cox models (adjusted for age and sex) by DAT status in all participants, and by median-split Fmax, TTT and AUC in DAT-positive participants. Event: PRIMDIAG = 1 or 5; prevalent cases and baseline-only participants excluded; censored at 12 years. |

Notebooks 02 and 03 each run two analyses of phenoconversion during follow-up (PRIMDIAG = 1, idiopathic PD, or 5, DLB, at any visit):

- **adjust**: converters kept, with `conv` and `time:conv` as covariates.
- **exclude**: converters removed entirely (sensitivity analysis).

Outcomes are MDS-UPDRS Part III, Hoehn and Yahr stage and MoCA. Each model is adjusted for age, sex, family history of PD and the baseline value of the outcome (MoCA models also for education and APOE ε4). Models are fitted by REML with `statsmodels` `MixedLM`, with a random intercept and slope per participant; slope differences have Wald 95% CIs and Holm-corrected P values.

## Data

PPMI data are not included in this repository. They are available to qualified researchers from the Parkinson's Progression Markers Initiative ([ppmi-info.org](https://www.ppmi-info.org)) under the PPMI Data Use Agreement.

Place the downloaded files in a project folder as follows:

```
<project folder>/
└── data/
    ├── PPMI_Curated_Data_Cut_Public_20260511.xlsx
    ├── SAA_Biospecimen_Analysis_Results_29Jul2026.csv
    ├── Kinetic_data_for_SAA_Results_29Jul2026.csv
    └── Blood_Chemistry___Hematology_29Jul2026.csv
```

The notebooks write to `results_baseline_dat/` (notebook 01), `analysis_dat/` (notebooks 02, 03 and 05) and `analysis_by_group_dat/` (notebook 04) inside the same folder, so the SAA results of `main` are never overwritten and both versions can share one data folder.

**DAT column.** Set `DAT_COL` in the configuration of notebook 01 to the curated column holding baseline DAT status (`DAT_MODE = 'status'`, codes `DAT_POSITIVE_CODES` / `DAT_NEGATIVE_CODES`, default 1 / 0), or to the numeric age/sex-expected ratio (`DAT_MODE = 'ratio'`, positive if < `DAT_RATIO_CUT` = 0.75). With `DAT_COL = None` the column is auto-detected only if exactly one DAT status column exists; otherwise the run stops and lists every DaT-related column so you can pick one. If your downloads have different file names, change them in notebook 01 (and the blood chemistry file name in the configuration of notebook 04).

## Running

Set the project folder in the first code cell of each notebook (`BASE_DIR`), or through an environment variable:

```bash
export PPMI_SAA_BASE=/path/to/project_folder
```
Locally:

```bash
pip install -r requirements.txt
jupyter lab
```

All settings (outcomes, covariates, time window, kinetic parameters, multiple-testing method, figure options) are in the **Configuration** cell of each notebook.

## Outputs

For each LMM analysis (notebooks 02 and 03), `analysis/<analysis name>/` contains:

- `*_estimates.csv` and `*_estimates.md`: slopes per group, slope difference, 95% CI, P and adjusted P
- `*_n_per_year_*.csv` / `.md`: participants contributing at each follow-up year
- converter counts (and, for the exclusion analysis, the excluded participant IDs)
- figures as PNG (600 dpi) and PDF: model-estimated trajectories and forest plots of slope differences
