# Azolla Thesis — Implementation Plan (Final)

Thesis submitted. Moving to implementation phase.

---

## Confirmed Modeling Architecture

### Framing 1 — Pre-experimental prediction
- **Target**: TCR = (ln(M21) − ln(M0)) / 21  [j⁻¹]
- **Features**: medium elemental composition (N, P, K, Ca, Mg, Fe in mmol/L), M0_g, light_lux, pH_target, temperature_mean
- **n**: 20 bacs (one row per bac)
- **Models**: Linear Regression, Ridge, Random Forest (max_depth=2, min_samples_leaf=3), XGBoost
- **Validation**: LOOCV — metrics: RMSE, MAE, R²
- **Interpretability**: SHAP on best model
- **Note**: humidity excluded (zero variance across bacs — room-level constant)

### Framing 2 — Operational depletion monitoring
- **Step 1**: Compute k per bac by fitting CE(t) = CE₀·e^(−kt) to observed EC time series (scipy curve_fit or log-linearization)
- **Target**: k [j⁻¹] — EC depletion rate constant
- **Features**: same as Framing 1 + CE_initial_uS
- **n**: 20 bacs (one row per bac)
- **Models**: same 4 algorithms
- **Validation**: LOOCV — same metrics
- **Alert formula**: t* = −ln(0.39) / k = 0.942 / k  [days]
  - seuil = 0.39 (fraction, NOT 39) — derived from IRR2 observation: CE dropped from 666 to 258 µS/cm before frond reddening
- **Note**: k ≈ 0 for Yoshida and Modified Hoagland (high CE₀) — intentional, provides range diversity

---

## Excluded Features — Document for Results Section

| Feature | Excluded from | Reason |
|---|---|---|
| Humidity (%) | Both models | Measured at room level — identical value for all 20 bacs. Zero variance across training examples; provides no discriminating information to the model. Reported as an experimental condition in M&M, not a predictor. |
| TDS (ppm) | Both models | Collinear with CE (r > 0.95 expected). Redundant information. CE retained as the primary conductivity indicator. |
| Salinity (ppt) | Both models | Same reason as TDS — derived from CE measurement, not independently informative. |
| Photoperiod (h) | Both models | Constant across all bacs and all days (controlled room). Zero variance. |

---

## Implementation Steps

### Step 1 — Data preparation script
File: `src/prepare_dataset.py`
- Load `bac_summary` sheet → one row per bac with M0, M21, features
- Load `ec_timeseries` sheet → fit exponential per bac → compute k
- Load `medium_composition` sheet → merge elemental features
- Compute TCR per bac
- Export: `data/model_ready.csv` (20 rows, all features + TCR + k)

### Step 2 — Chlorophyll analysis
File: `src/chlorophyll_analysis.py`
- Load chlorophyll data with medium mapping
- Descriptive stats per medium (mean ± SE)
- One-way ANOVA (Type III SS, pingouin) — Chl a, Chl b, Total Chl, Carotenoids
- Post-hoc Tukey HSD
- Figures: bar chart per medium, Chl a vs Chl b scatter

### Step 3 — Framing 1 model
File: `src/framing1_tcr.py`
- Feature matrix X, target y = TCR
- LOOCV loop over 4 models
- Results table: RMSE, MAE, R² per model
- SHAP plot for best model

### Step 4 — Framing 2 model
File: `src/framing2_k.py`
- Feature matrix X, target y = k
- LOOCV loop over 4 models
- Results table: RMSE, MAE, R² per model
- SHAP plot for best model
- Alert demonstration: given predicted k, compute t* = 0.942/k

### Step 5 — Biomass ANOVA
File: `src/biomass_anova.py`
- One-way ANOVA on final_biomass and TCR across 4 mediums
- Post-hoc Tukey HSD
- pH trajectory plot if daily data available

---

## Required Packages
```
pandas, numpy, scipy, matplotlib, seaborn,
statsmodels, pingouin, scikit-learn, shap, xgboost, openpyxl
```
