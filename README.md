# Paper 1 — Interpretable Stratification of Functional Outcome 
# and Rehabilitation Length of Stay at Admission

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
regression. This makes the predictions interpretable and actionable. Both sets of 
thresholds were computed from the training data only, so no information from the 
test set leaked into the model.

To make sure the models could be trusted, we used SHAP to explain every prediction — 
both at the population level and at the individual patient level.

---

## Dataset

- **Source:** Swiss Paraplegic Centre (SPZ), Nottwil, Lucerne, Switzerland
- **Period:** 2018 to 2020
- **Episodes:** 818 inpatient rehabilitation episodes
- **Unique patients:** 798 (some patients contributed more than one episode)
- **Data type:** Routinely collected clinical and administrative records
- **Privacy:** All records were fully anonymized prior to analysis

---

## Prediction tasks

### Task A — SCIM discharge outcome (3-class)

| Class | Threshold | Clinical meaning |
|---|---|---|
| Low | SCIM ≤ 22 (25th percentile) | High dependence at discharge |
| Medium | SCIM 22–67 | Partial to moderate independence |
| High | SCIM > 67 (75th percentile) | Strong functional independence |

### Task B — Rehabilitation LOS (3-class)

| Class | Threshold | Operational meaning |
|---|---|---|
| Short | LOS ≤ 29.24 days (33rd percentile) | Early discharge |
| Typical | LOS 29.24–183 days | Standard rehabilitation duration |
| Long | LOS > 183 days (66th percentile) | Extended or complex stay |

---

## Models evaluated

XGBoost, LightGBM, ExtraTrees, RandomForest, HistGradientBoosting — all trained 
within a unified preprocessing pipeline using median imputation for numerical 
features and one-hot encoding for categorical features.

---

## Reported results (from the accepted paper)

These are the exact numbers reviewed and accepted at WCCI 2026. They were produced 
on the original unmodified data file used during the research phase.

### Task A — SCIM discharge outcome

| Model | Accuracy | Balanced Acc | Macro F1 |
|---|---|---|---|
| **XGBoost** | **0.80** | **0.78** | **0.80** |
| LightGBM | 0.78 | 0.76 | 0.77 |
| RandomForest | 0.78 | 0.75 | 0.76 |
| HistGB | 0.75 | 0.75 | 0.75 |
| ExtraTrees | 0.72 | 0.68 | 0.70 |

### Task B — Rehabilitation LOS

| Model | Accuracy | Balanced Acc | Macro F1 |
|---|---|---|---|
| **ExtraTrees** | **0.52** | **0.51** | **0.50** |
| RandomForest | 0.48 | 0.47 | 0.44 |
| LightGBM | 0.47 | 0.46 | 0.46 |
| HistGB | 0.47 | 0.46 | 0.46 |
| XGBoost | 0.45 | 0.45 | 0.45 |

---

## Important note on reproducibility

Running this notebook will produce results that are close to but not identical 
to the numbers above. There are two reasons for this.

**Reason 1 — Data refinement**

The input data file has been updated since Paper 1 was submitted. For Paper 2, 
the LOS values, ICU hours, and ICD code counts were re-extracted and verified 
against the administrative source records. These corrections affected a subset 
of rows. The Paper 1 results were produced on the original unmodified file. 
The current file in this repository reflects the corrected version.

When the notebook is run on the current file, the results are:

### Task A — SCIM discharge outcome (rerun)

| Model | Accuracy | Balanced Acc | Macro F1 |
|---|---|---|---|
| **XGBoost** | **0.794** | **0.779** | **0.789** |
| RandomForest | 0.782 | 0.757 | 0.768 |
| LightGBM | 0.770 | 0.759 | 0.766 |
| ExtraTrees | 0.733 | 0.695 | 0.714 |
| HistGB | 0.733 | 0.732 | 0.735 |

### Task B — Rehabilitation LOS (rerun)

| Model | Accuracy | Balanced Acc | Macro F1 |
|---|---|---|---|
| **RandomForest** | **0.585** | **0.579** | **0.578** |
| LightGBM | 0.579 | 0.579 | 0.575 |
| XGBoost | 0.579 | 0.579 | 0.578 |
| HistGB | 0.560 | 0.559 | 0.558 |
| ExtraTrees | 0.547 | 0.546 | 0.546 |

The thresholds, methodology, feature set, and model rankings for Task A are 
consistent between both runs. The LOS improvement in the rerun reflects the 
corrected LOS values from the data refinement carried out for Paper 2.

**Reason 2 — Library version sensitivity**

GroupShuffleSplit uses a random number generator whose behavior can vary slightly 
across scikit-learn versions even with the same random seed. This can shift which 
patients land in training versus test, producing small differences in reported metrics.

The environment used for both runs:
- Python 3.14.3
- scikit-learn 1.8.0
- XGBoost 3.1.3
- LightGBM 4.6.0

---

## SHAP explainability

**SCIM discharge — key findings:**
The most influential predictor was the total SCIM score at rehabilitation admission, 
followed by domain subscores for self-care and sphincter management. The transition 
gap between acute discharge and rehabilitation admission was consistently in the top 
five features. Demographic variables contributed modestly and ranked below functional 
measures in every model tested.

**LOS — key findings:**
LOS predictions were dominated by diagnostic complexity, comorbidity burden, ICU 
exposure during the acute phase, and acute length of stay. Functional scores 
contributed positively but with lower magnitude, confirming that LOS is driven more 
by care burden and institutional factors than by patient function alone.

---

## How to run

### Requirements
```bash
pip install -r requirements.txt
```

### Steps

1. Clone the repository
2. Place your copy of the data file in the `finalTest/` folder
3. Open `paper1_reproducibility.ipynb` in Jupyter
4. Update `INPUT_PATH` in Step 2 to point to your file
5. Run all cells in order

### Outputs
```
model_outputs_paper1/
├── anonymized_features.xlsx
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

## Data availability

Clinical data from the Swiss Paraplegic Centre cannot be shared due to Swiss data 
protection requirements and institutional privacy constraints. The anonymized feature 
schema in `anonymized_features.xlsx` documents all feature columns used in training 
with patient identifiers removed.

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