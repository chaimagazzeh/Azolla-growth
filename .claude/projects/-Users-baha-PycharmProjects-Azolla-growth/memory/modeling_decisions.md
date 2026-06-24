---
name: Modeling Architecture Decisions
description: Final confirmed modeling framings, features, validation, and exclusions
type: project
---

**Experiment design (v3 data — current source of truth):** `dataset azolla_v3.xlsx`, sheet `Feuil2` (3rd sheet) is the most complete. 24 bacs UNBALANCED (3 NPK + 7 IRR2 + 7 Yoshida + 7 Modified Hoagland), reduced to **n=20** for consistency with biochemical set. Bac identity = Medium + Bloc + Bac (Bloc replaces Table).
- Light (LUX) per Bloc: 1=1141, 2=834, 3=727
- M0 (biomasse_initiale_g) is REAL and VARIES per bac (33–39 g) in v3 — not constant
- v3 Feuil2 has per-row nutrient columns (N,P,K,Ca,Mg,Fe,Mn,Mo,B,Cu,Zn,Co); composition extracted into data/medium_composition.csv
- tmp_and_humidity.txt = single room sensor, aggregated to daily means (temp≈23°C, humidity≈64%)

**n=20 drop rule (RANDOM_STATE=42, reproducible):** drop IRR2_Bloc1_B4 (incomplete pH 12/18) + 3 random from over-represented media (Yoshida_Bloc2_B1, IRR2_Bloc1_B2, Yoshida_Bloc2_B2), NPK control untouched. Final: NPK=3, IRR2=5, Yoshida=5, Hoagland=7.

**Framing 1 — Pre-experimental TCR prediction (n=20, IMPLEMENTED & RUN)**
- Target: TCR = ln(M21/M0)/21 (Hunt 1982)
- Features: 12 elemental concentrations (CSV) + light_lux + pH_mean + M0 + temp_mean + humidity_mean
- Models: Linear, Ridge, RF (max_depth=2, min_samples_leaf=3), XGBoost. Validation LOOCV, metrics RMSE+MAE (R² removed per thesis)
- RESULTS: RF best RMSE 0.0044 vs baseline 0.0088
- CRITICAL CAVEAT: M0 dominates importance because it's IN the TCR formula (TCR=(lnM21-lnM0)/t). Not leakage (M0 known pre-experiment) but partly mechanical. Other features (composition, light, pH) contribute little. temp/humidity importance=0 (zero variance, room sensor).

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