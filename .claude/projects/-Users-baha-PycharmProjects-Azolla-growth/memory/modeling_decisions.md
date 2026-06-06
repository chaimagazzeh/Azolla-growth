---
name: Modeling Architecture Decisions
description: Agreed modeling framings, validation strategies, and what to avoid
type: project
---

**Framing 1 — Pre-experiment prediction (n=24)**
- Features: medium elemental composition (P, K, N, Mg, Fe mmol/L) + initial_biomass + light_lux
- Target: final_biomass_g (and RGR = ln(B21/B0)/21)
- Models: Linear Regression, Ridge, RF (max_depth=3), XGBoost
- Validation: LOOCV (leave-one-bac-out)
- SHAP on best model

**Framing 2 — In-experiment adaptive prediction (24 bacs × 5 T-values = 120 rows)**
- For each bac at checkpoint T ∈ {3,7,10,14,17}: cumulative trajectory summaries as features
- Features: {medium_composition, initial_biomass, light_lux, days_elapsed, mean_pH_0_T, std_pH_0_T, delta_pH, pH_slope, mean_EC_0_T, EC_slope, GDD, correction_count}
- Target: final_biomass_day21
- Validation: Leave-One-Bac-Out CV (LOBOCV) — all T-values of same bac must be held out together
- Key output: R²(T) vs T prediction horizon curve

**IMPORTANT — what NOT to do:**
- Do NOT use temporal train/test split within same bacs (that's what CLAUDE.md originally said — it's wrong)
- Do NOT treat daily rows as independent observations (pseudoreplication)
- Do NOT fit logistic curves (only 2 anchor points per bac)
- Do NOT aggregate → 24 rows for ML (wastes temporal structure)
- TDS and Salinity are redundant with EC — drop them

**Framing 3 (recommended addition):**
- Correlation + partial correlation of environmental features vs. final_biomass
- Identifies which factors drive yield — most publishable analysis at n=24

**Statistical analysis (biological results chapter):**
- Type III SS ANOVA for unbalanced design (3 NPK, 7×3 others)
- Tukey HSD post-hoc
- Libraries: pingouin or statsmodels