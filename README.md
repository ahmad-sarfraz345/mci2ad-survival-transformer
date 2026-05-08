
# ADNI MCI-to-AD Conversion (Longitudinal Survival)

![PyTorch](https://img.shields.io/badge/PyTorch-2.x-red)
![ADNI](https://img.shields.io/badge/Dataset-ADNI-blue)
![Cox-Survival](https://img.shields.io/badge/Model-Survival%20Analysis-green)

This project predicts time-to-conversion from Mild Cognitive Impairment (MCI) to Alzheimer's Disease (AD) using longitudinal structural MRI, modeled as a survival analysis problem with a Cox objective.

##  Paper
This repository accompanies our ICML-style research paper on time-aware longitudinal transformers for Alzheimer’s disease progression prediction.

## Notebooks

- [baseline/baseline.ipynb](baseline/baseline.ipynb) — baseline model with visit-index positional encoding.
- [first_improvement/first_improvement.ipynb](first_improvement/first_improvement.ipynb) - temporal encoding for irregular visit timing.
- [second_improvement/second_improvement.ipynb](second_improvement/second_improvement.ipynb) - ALiBi temporal attention bias in the Transformer.

## Data

- ADNI longitudinal MRI cohort with 411 subjects (155 converters, 256 stable), each with at least four visits.
- Time-to-event is defined as number of days from baseline MRI to AD diagnosis.
- Patient-level stratified split (70%/15%/15%) is used to preserve class balance.
- Right censoring is applied to subjects who did not convert during follow-up.

## Preprocessing

- DICOM series are parsed per visit, slice order resolved using DICOM metadata, and volumes reconstructed into 3D arrays.
- Volumes are center-cropped to 96x96x96 and normalized per volume.
- The first four visits are retained for a fixed-length longitudinal sequence.

## Model and Training

- 3D CNN encoder (32->64->128->256) produces a 128-D visit embedding.
- Transformer encoder (2 layers, 4 heads) aggregates visit embeddings with a CLS token.
- Cox partial likelihood loss is used for survival modeling; early stopping is based on validation C-index.
- Optimizer: AdamW with warmup + cosine learning rate schedule.

## Evaluation

- C-index (primary survival metric).
- Time-dependent AUC at 2y/3y/5y using `cumulative_dynamic_auc` with windows [730, 1095, 1825] days.
- Binary metrics are computed using a threshold on predicted risk scores (median split on training set).

## Results (Test Set)

| Metric | Baseline | Temporal | ALiBi |
|---|---:|---:|---:|
| C-index | 0.7520 | 0.7665 | 0.7770 |
| AUC@2y | 0.9193 | 0.9259 | 0.9582 |
| AUC@3y | 0.6926 | 0.7286 | 0.7457 |
| AUC@5y | 0.7496 | 0.7318 | 0.7405 |
| Binary AUC | 0.7372 | 0.7404 | 0.7532 |
| Accuracy | 0.6825 | 0.6825 | 0.6825 |
| F1 | 0.6429 | 0.6429 | 0.6429 |
| Sensitivity | 0.7500 | 0.7500 | 0.7500 |
| Specificity | 0.6410 | 0.6410 | 0.6410 |

## Key Findings

- Modeling irregular time gaps improves survival ranking (C-index ↑)
- ALiBi provides strongest gains in short-term prediction (AUC@2y ↑)
- Binary classification metrics remain unchanged, confirming improvements are in risk calibration rather than threshold accuracy