---
name: Azolla Thesis Project Context
description: Core facts about the Azolla pinnata experiment, submitted thesis, and implementation scope
type: project
---

Master's thesis submitted 2026-06-10. Now implementing the Python code to produce the results described in the thesis.

**Experiment:** 20 bacs, balanced design: 4 Témoin (distilled water) + 4 NPK + 4 IRR2 + 4 Yoshida + 4 Modified Hoagland. Plus 6 phytotron bacs (analyzed separately).
**n=20, balanced** — ANOVA is simpler (Type I = Type III SS for balanced design).
Biomass measured only at Day 0 (M0) and Day 21 (M21). EC measured every 3 days. pH measured daily (3 spatial points per bac).

**Key data files:**
- `data/dataset azolla.xlsx` — raw pH/EC measurements
- `data/Chlorophyll+ Carotenoid abs azolla.xlsx` — 30 samples (samples 1–30) with Chl a, Chl b, Total Chl, Carotenoids
- User will add bac_summary and ec_timeseries sheets — see dataset_spec.md

**EC depletion threshold (established from data):**
- IRR2 bac: CE dropped from 666 µS/cm → 258 µS/cm before frond reddening appeared
- P_déplétion = 61% → seuil = 0.39 (fraction remaining, NOT 39)
- Alert formula: t* = −ln(0.39)/k = 0.942/k days

**Key biological findings:**
- Chl a/b ratio 4–6 across all samples (stress indicator — suboptimal humidity 40%, P-limitation IRR2, high EC Hoagland)
- pH acidified over time (NH4+ uptake) — valid IoT monitoring proxy
- Pink/red fronds in IRR2 = anthocyanin = phosphorus deficiency
- Optimal light ~800 lux, photoinhibition > 1400 lux
- EC depletion only observable in dilute media (IRR2, NPK) — k≈0 for Yoshida/Hoagland by design (high CE₀)
