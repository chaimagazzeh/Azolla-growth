---
name: Modeling Architecture Decisions
description: Final confirmed modeling framings, features, validation, and exclusions
type: project
---

**Experiment design (actual v2 data):** 24 bacs, UNBALANCED: 3 NPK + 7 IRR2 + 7 Yoshida + 7 Modified Hoagland. No témoin in v2. Bac identity = Medium + Table + Bac number (Bac repeats across Tables).
- Light (LUX) is per Table: Table 1 = 1141, Table 2 = 834, Table 3 = 727
- Initial biomass M0 = constant 36 g ± 2g for all bacs (no per-bac M0)
- "Biomasse finale" = last-day biomass (the pasted date is wrong/irrelevant)
- Experiment start = 2026-05-11 (Day 0)

**Framing 1 — Pre-experimental prediction (n=24, IMPLEMENTING)**
- Target: final biomass M21 (g). TCR = ln(M21/36)/t is a monotonic transform of M21 since M0 constant — report both but they rank identically.
- Features: medium elemental composition (from medium_composition sheet, pending), light_lux (by Table), pH_mean per bac
- Models: Linear Regression, Ridge, Random Forest (max_depth=2, min_samples_leaf=3), XGBoost
- Validation: LOOCV — metrics: RMSE, MAE, R²
- Interpretability: SHAP / feature importance on best model
- CAVEAT: only 4 distinct medium compositions → model learns ~4 group means, not a continuous relationship. Position SHAP as exploratory.

**Framing 2 — DROPPED from implementation.**
- Reason: EC data is single-timepoint per bac (no time series, no day column in old dataset). Cannot fit CE(t)=CE₀·e^(−kt) — k undefined with one point.
- Old dataset has 28 EC values but ~1 per bac, plus broken mapping (Medium+Replicate vs Medium+Table+Bac).
- KEEP Approach 2 in thesis as PROPOSED methodology / future work only — needs denser EC sampling for empirical validation.

**Excluded features (document in results):**
- Humidity: room-level constant, zero variance across bacs
- TDS / Salinity: collinear with EC, redundant
- Photoperiod: constant across all bacs and days

**Statistical analyses:**
- One-way ANOVA (Type III SS — unbalanced design) on biomass and chlorophyll across 4 mediums
- Post-hoc Tukey HSD, effect size η²
- Libraries: pingouin, statsmodels, scikit-learn, shap, xgboost, scipy

**What NOT to do:**
- Do NOT treat daily pH rows as independent observations (pseudoreplication) — aggregate to per-bac
- Do NOT use humidity, TDS, salinity, photoperiod as features
- Do NOT overstate Framing 1 predictive power (only 4 distinct feature profiles)