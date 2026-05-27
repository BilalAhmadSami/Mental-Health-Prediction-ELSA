# Mental Health Prediction on ELSA

Predicting **depression**, **anxiety**, and **emotional problems** in older adults using the English Longitudinal Study of Ageing (ELSA) — a 10-wave longitudinal cohort dataset spanning 2002–2021.

---

## Project Overview

This project applies binary classification machine learning to predict three mental health outcomes in adults aged 50+ in England. The data comes from ELSA, one of the most comprehensive longitudinal studies of ageing in the world.

The key challenge is that ELSA data changes structure across waves — variable names, coding schemes, and question routing differ between survey rounds — requiring substantial cross-wave harmonisation before any modelling can take place.

### Prediction Targets

| Outcome | Variable | Test-Set Prevalence (Wave 10) |
|---|---|---|
| Depression | `hepsyde` | ~10.5% |
| Anxiety | `hepsyan` | ~9.4% |
| Emotional Problems | `hepsyem` | ~3.0% |

---

## Dataset

- **Source**: [English Longitudinal Study of Ageing (ELSA)](https://www.elsa-project.ac.uk/)
- **Waves**: 10 waves (Wave 1: 2002–03 to Wave 10: 2020–21)
- **Population**: Adults aged 50+ in England
- **Sample**: ~97,000 participant-wave observations after harmonisation
- **Format**: Stata `.dta` files, one per wave

> The raw ELSA data files are not included in this repository as they require registration with the [UK Data Service](https://ukdataservice.ac.uk/). Access is free for academic and research use.

---

## Methodology

### Data Preparation
- **Target harmonisation**: Psychiatric outcome variables are encoded differently in Waves 1–2 (repeated-response coding) vs Waves 3–10 (direct binary flags). Custom derivation logic was implemented for each wave group.
- **Predictor harmonisation**: 10 predictor concepts were mapped to their correct source columns across all 10 waves, with response codes standardised to a common scale.
- **Missing value handling**: ELSA-specific sentinel codes (-1, -2, -8, -9) converted to `NA`; missing predictor values handled via imputation during modelling.

### Train / Validation / Test Split
A **temporal split** is used to respect the longitudinal structure of the data:
- **Training set**: Waves 1–8
- **Validation set**: Wave 9 (used for threshold tuning and model comparison)
- **Test set**: Wave 10 (held out until final evaluation)

### Predictors
The final model uses 10 harmonised predictors:

| Predictor | Description |
|---|---|
| `age_h` | Age (continuous) |
| `sex_h` | Sex (0 = male, 1 = female) |
| `relationship_status_h` | Partnered/married (1 = yes, 0 = no) |
| `long_standing_illness_h` | Long-standing illness (1 = yes, 0 = no) |
| `paid_employment_h` | In paid employment (1 = yes, 0 = no) |
| `pain_h` | Often troubled by pain (1 = yes, 0 = no) |
| `household_size_h` | Number of people in household |
| `self_rated_health_h` | Self-rated health (1 = excellent to 5 = poor) |
| `limiting_illness_h` | Illness limits activities (1 = yes, 0 = no) |
| `smoking_current_h` | Current smoker (1 = yes, 0 = no) |

### Models Evaluated
Multiple classifiers were trained and compared:
- Logistic Regression (baseline)
- Random Forest
- L1 and Elastic Net regularised Logistic Regression
- Histogram Gradient Boosting (HistGradientBoostingClassifier)
- Classic Gradient Boosting (GradientBoostingClassifier)
- LightGBM

Three imputation strategies were also compared:
- Most-frequent (mode) imputation
- Type-specific imputation (median for continuous, mode for binary)
- MICE / Iterative Imputation (IterativeImputer)

A complete-case sensitivity analysis was also conducted (rows with any missing predictor dropped, no imputation).

### Final Model
**Smoking-expanded Logistic Regression with Iterative Imputation** was selected as the final model. It achieved the strongest and most consistent performance across all three outcomes on the held-out Wave 10 test set.

---

## Results Summary

| Outcome | Test ROC-AUC | Test PR-AUC |
|---|---|---|
| Depression | ~0.75 | ~0.30 |
| Anxiety | ~0.74 | ~0.27 |
| Emotional Problems | ~0.76 | ~0.18 |

> PR-AUC values are low relative to ROC-AUC due to severe class imbalance (positive-class prevalence of 3-10%). See the modelling notebook for a detailed discussion of this trade-off.

---

## Repository Structure

```
Mental-Health-Prediction-ELSA/
├── ELSA_Data_Analysis_Annotated.ipynb   # Data exploration, target & predictor harmonisation
├── ELSA_Modelling_Annotated.ipynb       # Model training, comparison & evaluation
└── README.md
```

### Notebook Descriptions

**ELSA_Data_Analysis_Annotated.ipynb**
Covers the full data preparation pipeline:
- Loading all 10 ELSA waves
- Psychiatric target variable identification and cross-wave harmonisation
- Exploratory class-balance analysis and target selection
- Theory-driven candidate predictor definition
- Predictor harmonisation across all waves
- Final baseline predictor set selection

**ELSA_Modelling_Annotated.ipynb**
Covers the complete machine learning pipeline:
- Full harmonisation pipeline (consolidated)
- Baseline logistic regression training and evaluation
- Classification threshold tuning on the validation set
- Systematic comparison of 6 model types
- Imputation strategy comparison (3 strategies)
- Complete-case sensitivity analysis
- Final model selection and test-set evaluation
- Visualisations: ROC curves, Precision-Recall curves, Confusion Matrices, Feature Coefficients, Temporal Prevalence

---

## How to Run

1. Register for ELSA data access at the [UK Data Service](https://ukdataservice.ac.uk/)
2. Download the 10 wave Stata files and place them in a folder on Google Drive
3. Open the notebooks in Google Colab
4. Update the file paths in the data-loading cell to match your Drive folder
5. Run cells in order

Recommended run order: Run ELSA_Data_Analysis_Annotated.ipynb first to understand the data structure and harmonisation decisions, then ELSA_Modelling_Annotated.ipynb for the full modelling pipeline.

---

## Tools and Libraries

- Python 3 (Google Colab)
- pandas, numpy — data manipulation
- scikit-learn — modelling, imputation, evaluation
- lightgbm — LightGBM classifier
- matplotlib, seaborn — visualisation

---

## Author

**Bilal Ahmad Sami**
MSc Artificial Intelligence

