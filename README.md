# Detecting Low-Frequency Network Attacks in Imbalanced Public Network Traffic Datasets

**Author:** BAJI BABU GUDIPATI  
**Programme:** MSc Advanced Computer Networking  
**Institution:** Sheffield Hallam University  

## Project Overview

This repository contains the final machine-learning implementation for the MSc dissertation:

**“Detecting Low-Frequency Network Attacks in Imbalanced Public Network Traffic Datasets.”**

The project investigates whether imbalance-aware machine-learning approaches can improve detection of low-frequency network attack classes in the **UNSW-NB15** benchmark while also measuring the false-alarm cost on genuinely Normal traffic.

The work is an **analytical machine-learning prototype**, not a deployable intrusion-detection system.

## Repository Contents

```text
.
├── final-notebook1.ipynb
├── Final_unsw_nb15_low_frequency_attack_project/
│   ├── figures/
│   ├── validation_model_comparison.csv
│   ├── final_test_metrics.csv
│   ├── final_test_classification_report.csv
│   ├── final_test_per_class_metrics.csv
│   ├── rare_class_final_test_results.csv
│   ├── duplicate_cross_split_overlap_audit.csv
│   ├── generalisation_overlap_sensitivity.csv
│   ├── deduplicated_training_sensitivity.csv
│   ├── final_test_normal_false_alarm_breakdown.csv
│   ├── final_test_average_precision_by_class.csv
│   ├── final_test_top_confusion_pairs.csv
│   ├── final_model_feature_importance.csv
│   ├── final_test_predictions.parquet
│   ├── selected_final_pipeline.joblib
│   ├── run_manifest.json
│   └── REPORT_SUMMARY.md
└── README.md
```

> Output filenames may differ slightly depending on the final notebook execution.

## Dataset

The project uses the **UNSW-NB15** intrusion-detection dataset.

- Original source: https://research.unsw.edu.au/projects/unsw-nb15-dataset
- Kaggle version used: https://www.kaggle.com/datasets/dhoogla/unswnb15

Prepared files:

```text
UNSW_NB15_training-set.parquet
UNSW_NB15_testing-set.parquet
```

Dataset used in the final run:

- Training rows: **175,341**
- Testing rows: **82,332**
- Target: `attack_cat`
- Number of classes: **10**
- Predictors used: **34**
  - 31 numerical
  - 3 categorical: `proto`, `service`, `state`

The binary `label` variable is excluded because it directly reveals Normal-versus-Attack status and would introduce target leakage into the multiclass experiment.

## Low-Frequency Attack Definition

A class is treated as low-frequency when it represents **less than 2% of the official training set**.

Low-frequency classes:

- Analysis
- Backdoor
- Shellcode
- Worms

## Machine-Learning Workflow

```text
Dataset loading
      ↓
Data-quality and target-leakage checks
      ↓
Class-imbalance analysis
      ↓
Duplicate and cross-split overlap audit
      ↓
Stratified development/validation split
      ↓
Leakage-safe preprocessing
      ↓
Baseline and imbalance-aware modelling
      ↓
Validation-based model selection
      ↓
Refit selected model on full training data
      ↓
Official test evaluation
      ↓
Class-wise error and false-alarm analysis
      ↓
Overlap-free / deduplicated sensitivity analysis
      ↓
Feature importance and reproducibility outputs
```

## Preprocessing

### Numerical features
- Median imputation
- `RobustScaler`

### Categorical features
- Most-frequent imputation
- `OneHotEncoder(handle_unknown="ignore")`

### Leakage controls
- Official test data are not used during model selection.
- Preprocessing is fitted only within training pipelines.
- Resampling is applied only to training data.
- The binary `label` feature is excluded.
- Duplicate and exact cross-split overlap are explicitly audited.

## Models Evaluated

1. Dummy Most-Frequent
2. Logistic Regression
3. Class-Weighted Logistic Regression
4. Random Over-Sampling + Logistic Regression
5. Random Under-Sampling + Random Forest
6. Balanced Random Forest
7. XGBoost + Balanced Sample Weights

## Model Selection

The selection hierarchy was:

1. Validation Macro-F1
2. Validation rare-class Macro-F1
3. Balanced Accuracy
4. Lower Normal-to-Attack false-alarm rate as a final tie-breaker

Selected model:

**XGBoost + Balanced Sample Weights**

## Final Test Results

| Metric | Result |
|---|---:|
| Accuracy | 0.6890 |
| Balanced Accuracy | 0.6231 |
| Macro-F1 | 0.5027 |
| Weighted F1 | 0.7417 |
| MCC | 0.6267 |
| Rare-Class Macro Recall | 0.5737 |
| Rare-Class Macro-F1 | 0.2721 |
| Normal-to-Attack False-Alarm Rate | 0.3789 |
| Macro ROC-AUC | 0.9603 |
| Macro Average Precision | 0.5610 |

## Main Findings

- Imbalance-aware learning improved detection of several low-frequency attack classes.
- Performance differed substantially across minority classes.
- Shellcode achieved very high recall but relatively low precision.
- Backdoor achieved moderate recall but poor precision.
- Analysis remained difficult to classify.
- Worms produced encouraging results, but test support was very small.
- The main operational weakness was the **37.89% Normal-to-Attack false-alarm rate**.
- The dominant error was **Normal → Fuzzers**, with **10,572** Normal observations misclassified as Fuzzers.
- Exact train-test overlap was substantial, but stricter overlap-free sensitivity analysis showed that the central performance pattern remained broadly stable.

## Robustness Checks

The notebook includes:

- duplicate-row analysis
- predictor overlap analysis
- predictor-plus-target signature overlap analysis
- unique / overlap-free test evaluation
- deduplicated-training sensitivity analysis
- fixed random seed
- saved model pipeline
- execution manifest

The strict unique and overlap-free test subset contained **48,400 observations**.

## Visualisations

The notebook generates figures for:

- training class distribution
- train/test class proportions
- duplicate and cross-split overlap
- protocol distribution
- service distribution
- state distribution
- numeric-feature skewness
- Spearman correlation
- validation model comparison
- rare-class Macro-F1
- confusion matrices
- per-class precision / recall / F1
- false-positive rates
- precision-recall curves
- average precision
- actual vs predicted counts
- Normal false-alarm breakdown
- top misclassification pairs
- XGBoost feature importance
- robustness comparisons

## Main Python Libraries

```text
pandas
numpy
scikit-learn
imbalanced-learn
xgboost
matplotlib
plotly
joblib
pyarrow
```

## Running the Notebook on Kaggle

1. Open Kaggle.
2. Create a new notebook.
3. Add the dataset:
   https://www.kaggle.com/datasets/dhoogla/unswnb15
4. Upload or import `final-notebook1.ipynb`.
5. Confirm the two prepared Parquet files are available.
6. Run the notebook from top to bottom.

## Running Locally

Install the required packages:

```bash
pip install pandas numpy scikit-learn imbalanced-learn xgboost matplotlib plotly joblib pyarrow
```

Start Jupyter:

```bash
jupyter notebook
```

Open:

```text
final-notebook1.ipynb
```

Update the dataset path if the files are stored somewhere else locally.

## Reproducibility

The final workflow uses:

```text
random_state = 42
```

The notebook also exports reproducibility artefacts including:

- `selected_final_pipeline.joblib`
- `run_manifest.json`
- CSV metrics
- prediction outputs
- report-ready figures

## Ethical Scope

This project uses **secondary public data only**.

It does not involve:

- human participants
- surveys or interviews
- live network traffic capture
- penetration testing
- active system probing
- access to private organisational systems

The work is intended for academic research and evaluation only.

## Limitations

- UNSW-NB15 is a controlled and dated benchmark.
- Some rare classes contain very small numbers of examples.
- The benchmark contains duplicate and cross-split overlap.
- One stratified validation split was used for model selection.
- The final model still produces a substantial false-alarm burden.
- Feature importance represents predictive influence, not causality.
- The model has not been validated as a production IDS.

## Future Work

Possible extensions include:

- repeated or nested cross-validation
- class-specific decision thresholds
- probability calibration
- cost-sensitive threshold optimisation
- contemporary intrusion-detection datasets
- temporal and external validation
- SHAP-based model interpretation
- analyst evaluation under appropriate ethics approval
- deployment-oriented latency and resource testing

## Project Contribution

The main contribution is a **reproducible imbalance-aware multiclass intrusion-detection workflow** that evaluates not only overall performance, but also:

- low-frequency attack recall
- class-specific precision and F1
- Normal-traffic false alarms
- precision-recall behaviour
- duplicate/cross-split overlap
- robustness under stricter evaluation conditions

The results show that improving rare-attack sensitivity can create significant false-alarm costs, so overall accuracy alone is not sufficient for evaluating imbalanced intrusion-detection models.

## References

Moustafa, N., & Slay, J. (2015). UNSW-NB15: A comprehensive data set for network intrusion detection systems. *Military Communications and Information Systems Conference (MilCIS)*.

Moustafa, N., & Slay, J. (2016). The evaluation of Network Anomaly Detection Systems: Statistical analysis of the UNSW-NB15 data set and the comparison with the KDD99 data set. *Information Security Journal: A Global Perspective, 25*(1-3), 18-31.

Chen, T., & Guestrin, C. (2016). XGBoost: A scalable tree boosting system. *Proceedings of the 22nd ACM SIGKDD International Conference on Knowledge Discovery and Data Mining*.

Saito, T., & Rehmsmeier, M. (2015). The precision-recall plot is more informative than the ROC plot when evaluating binary classifiers on imbalanced datasets. *PLoS ONE, 10*(3), e0118432.

## Author

**BAJI BABU GUDIPATI**  
MSc Advanced Computer Networking  
Sheffield Hallam University

## Disclaimer

This repository is provided for academic research and educational purposes. It is not a production intrusion-detection system and should not be used as the sole basis for operational cybersecurity decisions.
