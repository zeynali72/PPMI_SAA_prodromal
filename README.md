# SAA status, SAA kinetics and clinical progression in prodromal PPMI participants

Analysis code for linear mixed models of motor and cognitive progression in the PPMI prodromal cohort, by baseline CSF α-synuclein seed amplification assay (SAA) status and by baseline SAA kinetic parameters.

## Notebooks

Run in order. Each notebook is self-contained and works in Google Colab or locally.

| Notebook | What it does |
|---|---|
| `notebooks/01_build_dataset.ipynb` | Builds a baseline-anchored analytic dataset from the PPMI curated data cut and the raw SAA files: one wide file (one row per participant) and one long file (one row per visit). SAA status comes from the curated `CSFSAA` field at baseline (codes 1 and 3 = positive, 0 = negative; inconclusive or missing excluded). Kinetics come from the baseline visit only. |
| `notebooks/02_lmm_saa_status.ipynb` | `outcome ~ time × SAA status + covariates + (1 + time \| participant)`, SAA-negative as reference, in all prodromal participants. |
| `notebooks/03_lmm_saa_kinetics.ipynb` | SAA-positive participants only. Same model with a median-split kinetic group (Fmax, TTT, AUC; Project 237, 24-hour assay) as the exposure. |
| `notebooks/04_table1_baseline.ipynb` | Table 1: baseline characteristics of SAA-positive vs SAA-negative participants, including baseline serum uric acid, glucose and creatinine, prodromal subgroups and CSF SAA assay. Mann–Whitney U, χ², Fisher's exact or permutation χ²; Holm correction. |
| `notebooks/05_survival_saa_kinetics.ipynb` | Phenoconversion-free survival: Kaplan–Meier curves and Cox models (adjusted for age and sex) by SAA status in all participants, and by median-split Fmax, TTT and AUC in SAA-positive participants. Event: PRIMDIAG = 1 or 5; prevalent cases and baseline-only participants excluded; censored at 12 years. |

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
    └── Blood_Chemistry___Hematology_13Sep2026.csv
```

The notebooks write to `results_baseline/` (notebook 01), `analysis/` (notebooks 02, 03 and 05) and `analysis_by_group/` (notebook 04) inside the same folder. If your downloads have different file names, change them in notebook 01 (and the blood chemistry file name in the configuration of notebook 04).

## Running

Set the project folder in the first code cell of each notebook (`BASE_DIR`), or through an environment variable:

```bash
export PPMI_SAA_BASE=/path/to/project_folder
```

In Colab, the notebooks mount Google Drive and use `/content/drive/MyDrive/PPMI_Projects/prodromal_SAA_groups` by default.

Locally:

```bash
pip install -r requirements.txt
jupyter lab notebooks/
```

All settings (outcomes, covariates, time window, kinetic parameters, multiple-testing method, figure options) are in the **Configuration** cell of each notebook.

## Outputs

For each LMM analysis (notebooks 02 and 03), `analysis/<analysis name>/` contains:

- `*_estimates.csv` and `*_estimates.md`: slopes per group, slope difference, 95% CI, P and adjusted P
- `*_n_per_year_*.csv` / `.md`: participants contributing at each follow-up year
- converter counts (and, for the exclusion analysis, the excluded participant IDs)
- figures as PNG (600 dpi) and PDF: model-estimated trajectories and forest plots of slope differences
