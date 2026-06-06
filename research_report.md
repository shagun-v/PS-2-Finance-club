# IV Surface Research Report

## Scope audited
Inspected accessible artifacts:
- dataset.csv
- submission-converter.ipynb
- competition_grade_solution.ipynb
- sandbox_solution.csv
- filled_dataset.csv
- analysis_summary.json.txt
- submission.csv
- supplied EDA plots

Note: the prompt mentions a project PDF, but no tool-accessible PDF file was attached in the available file list.

## Core diagnosis
This is primarily a **cross-sectional IV surface reconstruction** problem with scattered missing entries, not a tabular ML problem. The missing pattern is sparse and dispersed, while same-timestamp cross-strike structure is smooth and highly constrained.

## Current baseline audit
The current notebook uses row-wise PCHIP as the real production model, while the tree models are only benchmarked. The final prediction path is effectively:
1. row PCHIP
2. row linear fallback
3. lag-1 fallback
4. row mean / neighbor mean fallback

Main weaknesses:
- PCHIP is good, but it is a single-coordinate interpolator in strike space only.
- It does not explicitly exploit log-moneyness geometry.
- It does not adapt its local neighborhood weighting near expiry where curvature is most unstable.
- The ML benchmarking stack is feature-heavy but structurally mismatched to the problem, which explains why it loses badly.
- The final notebook evaluates many complex models but deploys only a simple interpolator, so there was still room to improve the interpolation layer itself.

## Low-rank / factor evidence
PCA on the mean-imputed standardized 975 x 28 matrix shows:
- PC1 = 73.62%
- First 2 PCs = 81.27%
- First 3 PCs = 84.40%
- First 8 PCs = 92.35%
- First 10 PCs = 93.94%

So the surface is definitely low-dimensional, but low-rank completion alone was still worse than local cross-sectional interpolation because the missingness is scattered and same-row shape is highly informative.

## Validation protocol
- TimeSeriesSplit with 4 folds
- Artificial masking only inside the validation segment
- Mask only originally observed values
- Same-timestamp observed strikes remain available, which matches the real inference task
- Metrics: MSE, MAE, RMSE

## Main model comparison
### Core production comparison
| Model | Mean MSE | Mean MAE | Mean RMSE |
|---|---:|---:|---:|
| triple_ensemble | 0.00009927 | 0.00223012 | 0.00640370 |
| local_poly_gauss | 0.00010152 | 0.00227000 | 0.00645744 |
| local_poly_logm | 0.00010192 | 0.00226680 | 0.00646792 |
| pchip | 0.00016932 | 0.00262561 | 0.00870519 |

### Exploratory tuning
| Method | Mean MSE |
|---|---:|
| triple_ensemble | 0.00009927 |
| gauss_logm_blend | 0.00010127 |
| adaptive_expiry | 0.00010143 |
| local_poly_gauss | 0.00010152 |
| local_poly_logm | 0.00010192 |
| local_poly_k5_inv | 0.00010299 |
| pchip | 0.00016932 |
| lowrank_raw_rank4 | 0.00053100 |
| lowrank_raw_rank6 | 0.00055300 |
| causal_last | 0.00412500 |
| lowrank_total_variance_rank4 | 0.00523000 |

## Expiry analysis
Near expiry is the real pain point.

Expiry-bucket MSE:
- PCHIP, <=1 day: 0.00161779
- Triple ensemble, <=1 day: 0.00096366
- PCHIP, >7 days: 0.00001412
- Triple ensemble, >7 days: 0.00000563

Interpretation:
- The final-day skew distortion is real, but it behaves more like **curvature amplification + interpolation instability** than a separate regime that needs a hand-coded expiry branch.
- Explicit expiry-specific branching was tested, but the smoother triple ensemble still won overall.

## Final selected model
For each missing point within CE and PE wings separately:
- 45% local quadratic fit on strike using Gaussian distance weights
- 45% local quadratic fit on log-moneyness using inverse-distance weights
- 10% PCHIP stabilizer

Why it wins:
- local quadratic captures smooth wing curvature better than pure PCHIP
- log-moneyness fit respects relative geometry around the underlying
- the PCHIP component stabilizes the ensemble at boundaries and under sparse neighborhoods

## Why generic ML lost
The problem is dominated by instantaneous cross-sectional shape, not latent nonlinear feature interactions. Tree models try to learn a global mapping, but the target is already almost recoverable from neighboring strikes within the same timestamp. That is why simple structured interpolation beats RandomForest, LightGBM, and XGBoost by orders of magnitude in validation.

## Delivered artifacts
- Production notebook: iv_surface_production_notebook.ipynb
- Production script: final_solution.py
- Final filled dataset: filled_dataset_best.csv
- Final submission: submission_best.csv
- Validation summary: validation_summary.csv
- Machine-readable research summary: research_summary.json

## Leaderboard expectation
This is a real improvement over the current interpolation baseline in masked time-aware validation. I would rate the probability of beating the existing public score as **meaningful**, but not guaranteed, because leaderboard masking may emphasize a different subset of locations. The method is simple, fast, stable, and directly aligned with the structure of the task.
