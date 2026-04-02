# Paper 1 — Interpretable Stratification of Functional Outcome and Rehabilitation Length of Stay at Admission

**Published at:** IEEE World Congress on Computational Intelligence (WCCI 2026)  
**Location:** Maastricht, Netherlands  
**Conference:** June 21–26, 2026

---

## What this paper is about

Spinal cord injury rehabilitation is complex and resource-intensive. Clinicians and 
administrators at the Swiss Paraplegic Centre need early, reliable estimates of two 
things when a patient arrives for rehabilitation: how much functional independence 
the patient is likely to recover by discharge, and how long they are likely to stay.

This paper addresses both questions using only data available at rehabilitation 
admission — no post-admission measurements, no discharge information, no variables 
collected during the rehabilitation stay itself. The goal was to build something 
that could realistically be used on the day a patient arrives.

We framed both outcomes as three-class classification problems rather than continuous 
regression. This makes the predictions interpretable and actionable — a clinician or 
bed manager can work with "this patient is likely to have a short stay" more easily 
than with a predicted number of days. Both sets of thresholds were computed from the 
training data only, so no information from the test set leaked into the model.

To make sure the models could be trusted, we used SHAP to explain every prediction — 
both at the population level (which features matter most overall) and at the individual 
patient level (why did the model predict this outcome for this specific person).

---

## Dataset

- **Source:** Swiss Paraplegic Centre (SPZ), Nottwil, Lucerne, Switzerland
- **Period:** 2018 to 2020
- **Episodes:** 818 inpatient rehabilitation episodes
- **Unique patients:** 798 (some patients contributed more than one episode)
- **Data type:** Routinely collected clinical and administrative records
- **Privacy:** All records were fully anonymized prior to analysis

The dataset covers both the acute hospitalization phase and the subsequent inpatient 
rehabilitation phase for each patient. Features were restricted to variables available 
at or before rehabilitation admission to simulate real clinical deployment conditions.

---

## Prediction tasks

### Task A — SCIM discharge outcome (3-class)

Functional independence at rehabilitation discharge was measured using the Spinal Cord 
Independence Measure (SCIM III), a validated 100-point instrument covering self-care, 
respiration and sphincter management, and mobility.

Discharge SCIM scores were stratified into three ordinal classes using percentile-based 
thresholds derived from the training set:

| Class | Threshold | Clinical meaning |
|---|---|---|
| Low | SCIM ≤ 22 (25th percentile) | High dependence at discharge |
| Medium | SCIM 22–67 | Partial to moderate independence |
| High | SCIM > 67 (75th percentile) | Strong functional independence |

The thresholds broadly align with clinically meaningful independence levels. The low 
category captures patients with substantial dependence across SCIM domains. The high 
category captures patients approaching or achieving functional independence. The medium 
category spans the wide heterogeneity typical of SCI rehabilitation outcomes.

### Task B — Rehabilitation LOS (3-class)

Length of stay was stratified using asymmetric percentiles to account for the strongly 
right-skewed LOS distribution:

| Class | Threshold | Operational meaning |
|---|---|---|
| Short | LOS ≤ 29.24 days (33rd percentile) | Early discharge |
| Typical | LOS 29.24–183 days | Standard rehabilitation duration |
| Long | LOS > 183 days (66th percentile) | Extended or complex stay |

---

## Features

Only variables available at or before rehabilitation admission were included. 
Features were organized into four groups:

**Functional status**
SCIM total scores and domain subscores at acute discharge and rehabilitation 
admission. Domain-level subscores captured self-care (AS_SUB), respiration and 
sphincter management (SP_SUB), and mobility (MH_SUB, MA_SUB). Normalized domain 
ratios were computed to characterize the balance of functional preservation across 
domains.

**Transition features**
The gap between acute discharge SCIM and rehabilitation admission SCIM, and a binary 
flag for early functional deterioration between the two phases. These features capture 
the momentum of early recovery before formal rehabilitation begins.

**Administrative and demographic features**
Age at rehabilitation admission, sex, primary ICD-10 diagnosis, number of distinct 
ICD-10 codes as a comorbidity proxy, ICU exposure during the acute phase, and 
calendar month and year of rehabilitation admission.

**Post-hoc derived features**
SCIM gain columns and LOS-dependent features were excluded from the respective 
prediction tasks to prevent leakage. All preprocessing was fitted on training data 
only and applied unchanged to the test set.

The anonymized feature schema is provided in `anonymized_features.xlsx`.

---

## Models evaluated

Five tree-based ensemble models were trained and evaluated within a unified 
preprocessing pipeline. All models used median imputation for numerical features, 
mode imputation and one-hot encoding for categorical features.

| Model | Description |
|---|---|
| XGBoost | Extreme Gradient Boosting with L1/L2 regularisation |
| LightGBM | Histogram-based gradient boosting, leaf-wise tree growth |
| ExtraTrees | Extremely Randomized Trees with random split thresholds |
| RandomForest | Bagged decision trees with feature subsampling |
| HistGB | Sklearn Histogram-Based Gradient Boosting |

---

## Results

### Task A — SCIM discharge outcome

| Model | Accuracy | Balanced Acc | Macro F1 |
|---|---|---|---|
| **XGBoost** | **0.80** | **0.78** | **0.80** |
| LightGBM | 0.78 | 0.76 | 0.77 |
| RandomForest | 0.78 | 0.75 | 0.76 |
| HistGB | 0.75 | 0.75 | 0.75 |
| ExtraTrees | 0.72 | 0.68 | 0.70 |

XGBoost achieved the best performance across all three metrics. Most errors occurred 
between adjacent classes rather than across extreme categories, which is consistent 
with the continuous nature of functional recovery.

### Task B — Rehabilitation LOS

| Model | Accuracy | Balanced Acc | Macro F1 |
|---|---|---|---|
| **ExtraTrees** | **0.52** | **0.51** | **0.50** |
| RandomForest | 0.48 | 0.47 | 0.44 |
| LightGBM | 0.47 | 0.46 | 0.46 |
| HistGB | 0.47 | 0.46 | 0.46 |
| XGBoost | 0.45 | 0.45 | 0.45 |

LOS stratification achieved moderate performance, which is consistent with prior 
literature showing that discharge timing is heavily influenced by non-clinical factors 
such as discharge coordination and social support availability. These factors are not 
captured in routine administrative data. Prolonged stays were identified more reliably 
than short stays, which is the most operationally useful outcome for bed management.

---

## SHAP explainability

Model behavior was analyzed using TreeSHAP. Both global and local explanations were 
generated for the best-performing model on each task.

**SCIM discharge — key findings:**
The most influential predictor was the total SCIM score at rehabilitation admission, 
followed by domain subscores for self-care and sphincter management. The transition 
gap between acute discharge and rehabilitation admission was consistently in the top 
five features across all models. Demographic variables contributed modestly and ranked 
below functional measures in every configuration tested.

**LOS — key findings:**
LOS predictions were dominated by diagnostic complexity (ICD main codes), comorbidity 
burden, ICU exposure during the acute phase, and acute length of stay. Functional scores 
contributed positively but with lower magnitude than in the SCIM task, confirming that 
LOS is driven more by care burden and institutional factors than by patient function alone.

Global SHAP bar charts and local waterfall plots are saved to the `TASK_A_SCIM/` and 
`TASK_B_LOS/` output folders when the notebook is run.

---

## How to run

### Requirements
```bash
pip install -r requirements.txt
```

Tested with:
- Python 3.14.3
- scikit-learn 1.8.0
- XGBoost 3.1.3
- LightGBM 4.6.0
- SHAP 0.47+

### Steps

1. Clone the repository
2. Place your copy of `ML_Ready.xlsx` in the `finalTest/` folder
3. Open `paper1_reproducibility.ipynb` in Jupyter
4. Update `INPUT_PATH` in Step 2 to point to your file
5. Run all cells in order

### Outputs

Running the notebook produces the following in `model_outputs_paper1/`:
```
model_outputs_paper1/
├── anonymized_features.xlsx        ← feature schema, no patient IDs
├── TASK_A_SCIM/
│   ├── metrics_SCIM_3class.xlsx
│   ├── SCIM_thresholds.txt
│   ├── SCIM_3class_shap_global_bar.png
│   ├── SCIM_3class_waterfall_correct.png
│   └── SCIM_3class_waterfall_wrong.png
└── TASK_B_LOS/
    ├── metrics_LOS_3class.xlsx
    ├── LOS_thresholds.txt
    ├── LOS_3class_shap_global_bar.png
    ├── LOS_3class_waterfall_correct.png
    └── LOS_3class_waterfall_wrong.png
```

---

## Reproducibility note

The results in the paper were produced on Python 3.14.3 with scikit-learn 1.8.0. 
Re-running the notebook on the same versions and the same input file will produce 
results within rounding of the reported numbers. Minor numerical differences may 
occur across environments due to differences in how GroupShuffleSplit handles 
random number generation. The thresholds, methodology, and model rankings are 
fully reproducible.

The original data file used for Paper 1 has since been refined for Paper 2 with 
corrected LOS, ICU, and ICD values sourced from the administrative records. 
The Paper 2 pipeline uses a separate corrected file. The Paper 1 notebook 
intentionally uses the original unmodified file to preserve exact reproducibility 
of the reported results.

---

## Data availability

Clinical data from the Swiss Paraplegic Centre cannot be shared due to Swiss data 
protection requirements and institutional privacy constraints. The anonymized feature 
schema (`anonymized_features.xlsx`) documents all 56 features used in training with 
patient identifiers removed.

Researchers wishing to conduct similar studies should contact the Swiss Paraplegic 
Centre directly regarding data access.

---

## Citation
```bibtex
@inproceedings{bamfo2026sci,
  title     = {Interpretable Stratification of Functional Outcome and 
               Rehabilitation Length of Stay at Admission},
  author    = {Bamfo, Kenneth Asamoah and Polanco, Boris and 
               Metzger, Stephan and Mancera, Jos{\'e} and 
               Alonso-Moral, Jos{\'e} M. and Ter{\'a}n, Luis},
  booktitle = {Proceedings of the IEEE World Congress on 
               Computational Intelligence (WCCI)},
  year      = {2026},
  address   = {Maastricht, Netherlands},
  publisher = {IEEE}
}
```

---

## Acknowledgements

This work was supported by the Swiss National Science Foundation under 
Grant No. 228818. The authors thank the Swiss Paraplegic Centre for data 
access and clinical guidance throughout the study.