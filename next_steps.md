# Azolla Thesis — Framing Assessment & Implementation Plan

Deadline: June 10, 2026

---

## 1. Framing Assessment

### Framing 1 — Pre-Experiment Agronomic Prediction
**"Given a medium and planned conditions, what final biomass will I get after 21 days?"**

**Status: VALID. Implement as the primary model.**

- n = 24 (one row per bac)
- Features: medium elemental composition (P, K, N, Mg, Fe in mmol/L — continuous, not dummy), initial_biomass_g, light_lux
- Target: final_biomass_g
- Temperature, humidity, photoperiod are room-level constants → no discriminating power across bacs → drop from this framing
- Models (in order of appropriateness for n=24):
  1. Multiple Linear Regression — interpretable baseline, valid for thesis
  2. Ridge Regression — regularized, better for small n with correlated features
  3. Random Forest (max_depth=3, min_samples_leaf=3) — include as comparison; LOOCV will show its instability at this n
  4. XGBoost — same; show its performance collapses or overfits vs. linear models
- Validation: LOOCV (leave-one-bac-out = leave-one-out, since n=24)
- SHAP on Ridge or RF (whichever wins LOOCV) to explain which elemental inputs drive biomass

**Why this framing is honest and defensible:**
Medium composition is the only thing a farmer controls before starting. The model answers exactly the agronomic question: given your nutrient choices, what yield can you expect? The fact that n=24 requires regularized models is a real constraint that should be stated explicitly — this is scientific honesty, not a flaw.

**One important correction to the previous plan:**
The "Implied growth rate ln(B₂₁)/ln(B₀)/21" formula in CLAUDE.md has a typo.
Correct formula: `RGR = ln(B_final / B_initial) / days`
This is the Relative Growth Rate (RGR), not growth rate from separate logs.
Use RGR as an alternative target variable alongside final_biomass_g.

---

### Framing 2 — In-Experiment Adaptive Prediction (Smart Monitoring)
**"Given what I have observed up to day T, what will the final biomass be at Day 21?"**

**Status: VALID. Implement as the secondary model / proof-of-concept.**

**Architecture (cumulative trajectory summarization):**
For each bac b and each prediction checkpoint T ∈ {3, 7, 10, 14, 17} days:
```
features_at_T = {
    medium_composition (P, K, N, Mg, Fe),  # static
    initial_biomass_g,                       # static
    light_lux,                               # static
    days_elapsed = T,                        # changes with T
    mean_pH_0_to_T,                          # cumulative summary
    std_pH_0_to_T,                           # spatial + temporal variability
    delta_pH = pH_day0 - pH_dayT,           # total acidification proxy
    pH_slope_0_to_T,                         # linear trend (metabolic proxy)
    mean_EC_0_to_T,                          # cumulative summary
    EC_slope_0_to_T,                         # depletion rate
    GDD_0_to_T,                             # growing degree days
    correction_count_0_to_T,                 # interventions needed (stress proxy)
}
target = final_biomass_day21               # same for all T of this bac
```

24 bacs × 5 prediction checkpoints = 120 training examples (sufficient for regularized ML).

**Critical validation correction from previous plan:**
The CLAUDE.md says "temporal split: train on days 0–14, test on days 15–21."
This is WRONG. It uses the same bacs in train and test, which leaks bac-specific information.

**Correct validation: Leave-One-Bac-Out CV (LOBOCV)**
- Fold k: hold out ALL T-values for bac k, train on all T-values for the remaining 23 bacs
- This properly tests "can the model generalize to a new bac it has never seen?"
- 24 folds, each fold has 23 bacs × 5 T-values = 115 training rows and 5 test rows

**Secondary analysis (within-sample, no CV needed):**
Plot R²(T) vs. T — how does prediction accuracy improve as more of the trajectory is observed?
This is the "prediction horizon" curve and is the most visually compelling output of Framing 2.
It tells us: "By Day 10, we can predict final yield with X% accuracy → trigger IoT alert before it's too late."

**Why this framing matters for the thesis:**
- Justifies the smart agriculture / IoT monitoring angle
- Even with n=24 bacs, the demonstration of METHODOLOGY is the contribution
- Literature precedent: hybrid mechanistic + ML approaches are published with similar n (see CEA studies)
- Position explicitly as "proof of concept validated on controlled experiment data"

---

### Recommended Addition: Framing 3 — Feature Importance / Environmental Driver Analysis
**"Which environmental and nutritional factors most strongly determine Azolla yield?"**

**Status: RECOMMENDED. Complements both framings without adding implementation burden.**

- Correlation matrix: {mean_pH, EC_depletion, temperature, light_lux, P_concentration, N_concentration} vs. final_biomass
- Partial correlations controlling for medium type
- Feature importances from best Framing 1 model (SHAP values)
- Result: ranked list of factors → directly informs IoT sensor priority recommendations

This is the most publishable analysis at n=24 because correlation/partial correlation has no minimum sample size assumption (it just has lower power), and it directly answers an agronomic question.

---

### What NOT to Do (traps to avoid)

1. **Do not train daily-row models** — treating each of 504 daily rows as an independent observation is pseudoreplication. The statistical unit is the bac (n=24), not the measurement.

2. **Do not use logistic curve fitting** — B₀ and B₂₁ give only 2 anchor points per bac; fitting a 3-parameter sigmoid (K, r, t₀) is underdetermined.

3. **Do not use random splits** on the multivariate time series — mixing different bacs' days in train/test is leakage-free but ignores the temporal structure and bac-level autocorrelation.

4. **Do not overstate the ML results** — at n=24, R² from LOOCV will be noisy. Frame as "preliminary evidence" and let the ANOVA + biological results be the scientific core.

---

## 2. Statistical Analysis Plan (Biological Results Chapter)

### 2a. Chlorophyll Analysis (data already available)
1. Descriptive stats per medium: mean ± SE of Chl a, Chl b, Total Chl, Carotenoids, Chl a/b ratio
2. Visualization: bar chart with error bars, grouped by medium; scatter of Chl a vs Chl b
3. One-way ANOVA (Type III SS) for unbalanced design:
   - Response variables: Chl a, Chl b, Total Chl, Carotenoids, Chl a/b ratio (separately)
   - Factor: Medium (4 levels: NPK, IRR2, Yoshida, Modified Hoagland)
   - Check: Levene's test for homogeneity of variance, Shapiro-Wilk normality
4. Post-hoc: Tukey HSD
5. Effect size: η² (eta-squared)
6. Note on Chl a/b ratio: values of 4–6 indicate stress response (literature: healthy Azolla ~2–3, stressed 4–6). Interpret in context of suboptimal humidity (40%), phosphorus limitation (IRR2), high EC (Modified Hoagland).

### 2b. Biomass Analysis (needs Day 21 data)
1. One-way ANOVA on final_biomass and RGR across 4 mediums (same structure as chlorophyll)
2. Post-hoc Tukey HSD
3. pH trajectory visualization: mean pH per medium per day (line plot, error bands)
4. Correlation: mean_pH vs. final_biomass; EC_depletion vs. final_biomass

### 2c. Phytotron Analysis (separate section)
1. Light intensity (lux) vs. final biomass for 6 bacs
2. Simple linear/polynomial regression (n=6, just describe the relationship)
3. Compare to literature: optimal light ~300–1000 lux, photoinhibition > 1400 lux

---

## 3. Implementation Plan & Priority Order

Given deadline June 10 (4 days), execute in this order:

### Phase 1 — Chlorophyll Analysis (can start TODAY, no missing data)
**Files to create:**
- `analysis/chlorophyll_analysis.py` — full analysis + figures

**Outputs:**
- Figure 1: Bar chart of Total Chlorophyll per medium (with Tukey groups)
- Figure 2: Chl a vs Chl b scatter colored by medium
- Figure 3: Carotenoid content per medium
- Table: ANOVA + Tukey HSD results
- Chl a/b ratio interpretation

**Blocker**: Need sample-to-medium mapping (Q1 in questions.md)

---

### Phase 2 — Dataset Assembly (user action required first)
User provides:
- Final biomass per bac (Q5)
- Day assignment for measurement occasions (Q8)
- Light intensity per bac (Q11)

**File to create:**
- `data/clean_dataset.csv` — assembled, cleaned, ready for modeling

**Script to create:**
- `data/prepare_dataset.py` — reads raw Excel, adds provided data, interpolates EC, computes engineered features, exports clean CSV

---

### Phase 3 — Biomass ANOVA + pH Trajectory Analysis
**Files to create:**
- `analysis/biomass_anova.py` — ANOVA on final biomass and RGR
- `analysis/ph_trajectory.py` — pH over time per medium, pH-biomass correlations

---

### Phase 4 — Framing 1 Modeling
**File to create:**
- `modeling/framing1_preexperiment.py`
  - Feature matrix: medium composition (elemental, not dummy) + initial_biomass + light_lux
  - LOOCV for Linear Regression, Ridge, Random Forest, XGBoost
  - SHAP values for best model
  - Output: model performance table + SHAP plot

---

### Phase 5 — Framing 2 Modeling
**File to create:**
- `modeling/framing2_inExperiment.py`
  - Build cumulative trajectory dataset (24 bacs × 5 T-values)
  - LOBOCV
  - R²(T) curve
  - SHAP for feature importance

---

## 4. Required Python Packages

Add to requirements.txt:
```
pandas
numpy
matplotlib
seaborn
scipy
statsmodels
pingouin
scikit-learn
shap
xgboost
openpyxl
```

---

## 5. Key Biological Insights to Frame in the Thesis

1. **Chl a/b ratio 4–6**: Not ~2.0 as previously estimated. Literature confirms Azolla Chl a/b > 3 indicates stress. In our case: humidity at 40% (optimal >85%), phosphorus limitation in IRR2, high EC in Modified Hoagland. This is a finding, not a flaw. Interpret in Discussion.

2. **pH acidification, not alkalization**: NH₄⁺ uptake releases H⁺. Well-documented in literature. The daily pH trajectory is therefore a valid proxy for metabolic activity and should be highlighted as a potential real-time IoT signal.

3. **Anthocyanin accumulation in IRR2**: Phosphorus deficiency stress marker. Biologically significant — IRR2 plants are P-limited, diverting carbon from growth to stress response.

4. **Optimal light ~800 lux, photoinhibition > 1400 lux**: Consistent with Azolla's shade-adapted biology. Validates the phytotron experiment design.

5. **pH spatial gradient within bac**: Reflects coverage heterogeneity. The spatial std of pH (pH_std) is a genuine biological indicator of uneven growth and should be used as a feature.

---

## 6. Thesis Positioning of the ML Work

This is critical for the defense:

The modeling section contributes a **methodological demonstration**: it shows HOW a data-driven monitoring system for Azolla production would work, validated on experimental data from a controlled trial. The n=24 constraint is explicitly stated and the models are framed as preliminary. The scientific novelty is:
- First application of cumulative trajectory ML to Azolla biomass prediction (from the available literature)
- Feature importance analysis identifying which parameters most drive yield (actionable for farmers)
- SHAP explainability providing interpretable model outputs suitable for IoT alert systems

The biological results (ANOVA, chlorophyll, pH trajectories) are the primary scientific contribution. The ML is the applied/smart agriculture contribution.