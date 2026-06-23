Azolla pinnata — Project Context Summary for Claude
Who I am
Biotech engineering student, finalizing a master's thesis. Working largely independently due to an absent professional supervisor. Academic supervisor is supportive. Submission deadline: June 10, 2026.

Project Overview
Developing a data-driven system to monitor and predict biomass production of Azolla pinnata in controlled conditions representative of the Tunisian climate. The thesis combines a biological experiment with machine learning modeling and smart agriculture integration.

Experimental Design
Setup:

24 rectangular bacs (50×30×26 cm, 10 cm useful depth, 1650 cm² surface area) lined with black plastic
Located in a culture room with stable conditions

Treatment groups (unbalanced factorial design):

3 bacs — NPK (témoin/control)
7 bacs — IRR2 (most diluted medium, phosphorus-limited)
7 bacs — Yoshida (more concentrated, contains external nitrogen — likely redundant given Azolla's N-fixation via Anabaena azollae symbiosis)
7 bacs — Modified Hoagland (most concentrated, nitrogen-free by design — the most scientifically rigorous choice)

Additional phytotron experiment:

6 smaller bacs harvested from IRR2, inoculated with standardized 20-21g biomass on Day 0 (26/05/2026)
Testing light intensity (high/medium/low positions in room, measured with smartphone lux meter app — values 600–1400 lux, used as relative indicators not absolute)
Humidity is ~40% (suboptimal, Azolla prefers 85%+)
Phytotron light was broken — engineer contacted for repair


Harvest & Sampling Strategy

Partial harvest every 3–4 days: remove 50% of surface coverage
This maintains the culture in exponential growth phase
Experiment duration: 21 days minimum (Day 0 = start of main experiment)
Currently at approximately Day 20–21

Biomass measurements:

Fresh weight measured after 30-second drain
CRITICAL CONSTRAINT: Biomass only measured at Day 0 (initial) and Day 21 (final) — NO intermediate biomass measurements
This is the central architectural constraint for modeling


Daily Measured Features (per bac, per day)
FeatureNotespH_point1, pH_point2, pH_point33 spatial measurements per bacpH_meanAverage of 3 pointspH_stdSpatial variability — potential stress indicatorEC (µS/cm)Measured every 3 days, interpolated linearly for daily valuesTDSCorrelated with EC (r likely >0.95) — may be droppedSalinityAlso correlated with EC — may be droppedTemperature (°C)DailyHumidity (%)Daily if availablePhotoperiod (hours)DailyLight intensity (lux)Per bac positiondistilled_water_added_LLogged when pH dropped too lowcorrection_flagBinary, 1 = water or nutrient additionnutrient_top_up_flagBinary, 1 = nutrient reinoculationharvest_flagBinary, 1 = harvest dayanomaly_flagBinary, 1 = anomalous observationnotesFree text
Static features (per bac):

Medium type (categorical)
Elemental composition of each medium (P, K, N, Mg, Fe, etc. in mmol/L — exact concentrations known from literature/preparation)
Initial biomass (g)
pH target level


Engineered Features (to be computed in Python)

ΔpH per day — rate of acidification, proxy for ammonium uptake and metabolic activity
EC depletion rate — slope of EC over time, proxy for nutrient consumption
pH stability index — σ(pH) over 24h, precedes nutrient collapse
GDD (Growing Degree Days) — cumulative heat index, Σ[(Tmax+Tmin)/2 − Tbase]
Implied growth rate — ln(B₂₁)/ln(B₀)/21, assuming exponential approximation


Biochemical Analyses (endpoint, Day 21)

Chlorophyll a and b (Lichtenthaler 1987, 80% acetone, wavelengths 645 and 663 nm)
Carotenoids (470 nm)
Proline (osmotic stress indicator)
Soluble sugars (photosynthetic output)
MDA / Malondialdehyde (lipid peroxidation, oxidative damage marker)
Antioxidant enzyme activities (SOD, CAT, GPX)

Current status: 16 of 24 bacs measured for chlorophyll. Two distinct groups identified:

High chlorophyll group: ~15.5–15.8 µg/mL total (samples 2–4)
Mid chlorophyll group: ~8–11.5 µg/mL total (samples 5–16)
Chl a/b ratio stable at ~2.0 across all samples — no shade adaptation, quantity difference not quality
Sample 17 excluded (light-degraded)
Remaining 8 bacs still need to be measured


Key Biological Observations Made During Experiment

pH acidified over time (not alkalized as initially expected) — driven by NH₄⁺ uptake releasing H⁺, root exudates, and microbial decomposition. Evaporation concentrates the effect but doesn't cause it
Pink frond coloration observed in IRR2 — anthocyanin accumulation indicating phosphorus deficiency specifically
Best surface coverage observed around 800 lux — consistent with Azolla's shade-adapted biology (photoinhibition above ~1400 lux)
pH range 5.33–5.77 across phytotron bacs on Day 2 — optimal and consistent
pH spatial gradient within bacs correlates with coverage density


Statistical Framework

ANOVA type: One-way ANOVA on final biomass and chlorophyll across mediums. Type III Sum of Squares required due to unbalanced design
Post-hoc: Tukey HSD
Python libraries: pingouin or statsmodels
Validation: LOOCV (Leave-One-Out Cross Validation) due to small sample size (n=24 bacs)


The Core Modeling Problem (Most Recent Discussion)
The architectural flaw identified
Standard approach of training a model on daily rows with environmental features and predicting final biomass treats each row as independent. This is wrong because:

The data is a multivariate time series — daily observations are causally linked
Train-serve skew — model trained on full 21-day trajectories but asked to predict from a single daily snapshot at inference
Random splitting causes temporal data leakage — model trains on day 15, tests on day 8

The critical data constraint
Only two biomass measurements per bac: Day 0 and Day 21. No intermediate measurements. This means:

Option 1 (next-harvest rolling prediction) is not viable — no intermediate ground truth to train on
Option 2 (logistic curve fitting) requires minimum 4–5 points per bac to fit K, r, t₀ reliably — not viable with 2 points
Aggregating daily features into means/stdev per bac collapses 24 bacs to 24 rows — too small for ML, and wastes temporal structure

The two honest prediction framings agreed upon
Framing 1 — Pre-experiment agronomic prediction

Question: Given a chosen medium and expected environmental conditions, what final biomass will I achieve after 21 days?
Features: static/planned values — medium composition, target pH, expected temperature, light intensity
Target: final_biomass_g
Training data: 24 bacs = 24 examples
Validation: LOOCV
Bound to 21 days by design
Use case: help a farmer decide which medium and conditions to use before starting

Framing 2 — In-experiment adaptive prediction (smart monitoring)

Question: Given what I have observed up to day T, what will the final biomass be at day 21?
Uses sliding window approach — for each bac, create multiple training examples at different prediction points (day 3, day 7, day 14...)
Features at prediction day T = cumulative trajectory summaries up to T (mean_pH_0_T, std_pH_0_T, EC_slope_0_T, mean_temp_0_T, days_elapsed, medium composition, initial_biomass...)
From 24 bacs × ~7 prediction points = ~168 training examples
NOT aggregation — cumulative trajectory summarization preserving temporal progression
Architecturally honest: at inference on day T, all features genuinely available
No temporal leakage
Use case: IoT alert system — predict trajectory outcome before day 21, trigger alerts if heading toward poor yield

Models to use
ModelPurposeMultiple Linear RegressionBaseline, interpretable coefficientsRandom Forest (max_depth=3, min_samples_leaf=3)Main model, regularized for small sampleXGBoostChallenger model, compare performanceSHAPExplainability layer on best model
Evaluation metrics

R², RMSE, MAE
LOOCV for Framing 1
Temporal split for Framing 2 (train on days 0–14 trajectories, test on days 15–21 trajectories)


What Needs to Happen Next

Measure remaining 8 bacs for chlorophyll
Complete all biochemical assays (MDA, proline, sugars, antioxidants) — samples at -20°C
Assemble final clean dataset in Excel then export to CSV
Build modeling pipeline in Python implementing Framing 1 and Framing 2
Run ANOVA on chlorophyll and final biomass
Generate SHAP plots
Write results and discussion chapters
Submit by June 10


Thesis Structure

Introduction (complete)
Literature review — Azolla biology, growth factors, smart agriculture, predictive modeling
Materials and Methods — factorial design, sampling protocol, smart system architecture
Biological results — growth curves, chlorophyll, biochemical markers, ANOVA
Predictive modeling — feature engineering, model comparison, SHAP analysis
Discussion and smart agriculture integration recommendations
Conclusion and perspectives (computer vision for biomass estimation as future work)