# Implied Volatility Surface Prediction


## Overview

This project focuses on reconstructing missing values in an implied volatility (IV) surface derived from NIFTY50 options market data.

The objective was to predict missing implied volatility observations across strikes and timestamps while preserving the natural structure of the volatility surface.

The competition was hosted on Kaggle, where participants were required to submit predictions for hidden missing IV values and provide a fully reproducible notebook capable of generating the exact submitted CSV file.

---

## Competition Objective

Given a partially observed implied volatility surface:

* Predict missing implied volatility values.
* Preserve smile and skew characteristics of the IV surface.
* Utilize relationships across strikes and timestamps.
* Avoid look-ahead bias.
* Produce a reproducible solution.

### Evaluation Metric

Mean Squared Error (MSE) between predicted implied volatility values and hidden ground-truth values.

---

## Final Result

### Kaggle Public Leaderboard Score

**0.0000381115**

The final solution achieved this score using a custom volatility surface reconstruction framework based on local interpolation and ensemble smoothing techniques.

---

## Dataset Description

The dataset consists of:

* 975 timestamps
* 28 option contracts
* Single expiry series
* CE and PE option wings
* Approximately 20% missing implied volatility values

### Key Characteristics

* Smooth volatility smile structure
* Persistent downside skew
* High correlation across contracts
* Strong cross-sectional relationships
* Scattered missing observations
* Increased curvature near expiry

---

## Exploratory Data Analysis

### Volatility Smile

The IV surface exhibits a stable smile/skew structure:

* CE implied volatility generally increases with strike.
* PE implied volatility generally decreases with strike.

### Cross-Wing Skew

PE implied volatility is consistently higher than CE implied volatility for comparable strikes.

Near expiry, the skew becomes more unstable and surface curvature increases significantly.

### Correlation Structure

Principal Component Analysis revealed a highly structured surface:

| Components | Explained Variance |
| ---------- | ------------------ |
| PC1        | 73.6%              |
| PC1–2      | 81.3%              |
| PC1–3      | 84.4%              |
| PC1–8      | 92.3%              |
| PC1–10     | 93.9%              |

These results indicate a strong low-dimensional latent volatility structure.

### Missingness Pattern

Missing observations are distributed throughout the surface rather than concentrated in large blocks.

This suggests that local reconstruction methods can effectively leverage neighboring information.

---

## Methodology

### Initial Approaches Evaluated

The following methods were investigated:

* Linear interpolation
* PCHIP interpolation
* Time-based interpolation
* Random Forest
* XGBoost
* LightGBM
* PCA-based reconstruction
* Low-rank matrix completion

Validation consistently showed that local cross-sectional interpolation significantly outperformed machine learning approaches.

---

## Final Model

The final production solution uses a weighted ensemble of three interpolation techniques.

### Local Quadratic Interpolation (Strike Space)

Captures local smile curvature using neighboring strikes.

Weight: **45%**

### Local Quadratic Interpolation (Log-Moneyness Space)

Models the surface relative to the underlying price and improves stability around ATM regions.

Weight: **45%**

### PCHIP Interpolation

Provides monotonic spline stabilization and boundary robustness.

Weight: **10%**

### Final Prediction Formula

Prediction =

0.45 × Local Strike Interpolation

* 0.45 × Local Log-Moneyness Interpolation

* 0.10 × PCHIP Interpolation

Interpolation is performed separately for CE and PE wings at each timestamp.

---

## Validation Framework

A time-aware validation strategy was used to prevent leakage and accurately simulate the competition setting.

### Validation Setup

* TimeSeriesSplit
* Artificial masking of observed values
* Evaluation only on masked observations
* No look-ahead bias

### Metrics

* MSE
* RMSE
* MAE

### Validation Results

| Model                            |  Mean MSE |
| -------------------------------- | --------: |
| Triple Ensemble                  | 0.0000993 |
| Local Polynomial (Strike)        | 0.0001015 |
| Local Polynomial (Log-Moneyness) | 0.0001019 |
| PCHIP                            | 0.0001693 |

The final ensemble reduced validation error by approximately **41%** compared to the PCHIP baseline.

---

## Repository Structure

```text
.
PS-2-Finance-club/
├── dataset.csv
├── filled_dataset_best.csv
├── iv_surface_production_notebook.ipynb
├── research_report.md
├── research_summary.json
├── submission_best.csv
├── validation_summary.csv
└── README.md
```

---

## Reproducing Results

### Open the Notebook

Open:

```text
notebooks/iv_surface_production_notebook.ipynb
```

### Run All Cells

Execute the notebook from top to bottom.

The notebook performs:

1. Data loading and preprocessing
2. Exploratory Data Analysis
3. Validation experiments
4. Surface reconstruction
5. Missing value prediction
6. Submission generation

### Generated Outputs

Running the notebook generates:

* `filled_dataset_best.csv`
* `submission_best.csv`
* Validation summaries
* Research artifacts

---

## Kaggle Submission

The generated submission file:

```text
submission_best.csv
```

can be uploaded directly to the Kaggle competition page.

The notebook fully reproduces the submitted CSV file and satisfies the competition reproducibility requirement.

---

## Key Insights

* Implied volatility reconstruction is primarily a surface reconstruction problem rather than a traditional machine learning problem.
* Local cross-sectional information contains most of the predictive signal.
* Financially informed interpolation methods outperform generic ML models on this dataset.
* Combining strike-space and log-moneyness interpolation substantially improves reconstruction quality.
* Simple models aligned with market structure can outperform significantly more complex alternatives.

---

## Future Work

Potential areas for further improvement include:

* Temporal smoothing across neighboring timestamps
* PCA-based residual correction
* CE–PE joint reconstruction
* Adaptive ensemble weighting
* Submission ensembling
* Expiry-aware local surface adjustments

---

## Competition Requirements

This repository includes:

* Final Kaggle submission file
* Fully reproducible notebook
* Validation framework
* Modeling workflow
* Research report
* Supporting analysis

All preprocessing, feature engineering, modeling, validation, and prediction steps are documented inside the notebook.

---

## Final Kaggle Score

**0.0000381115**
