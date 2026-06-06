---
name: Azolla Thesis Project Context
description: Core facts about the Azolla pinnata experiment and thesis scope
type: project
---

Master's thesis: data-driven system to monitor and predict Azolla pinnata biomass production in controlled conditions (Tunisian climate simulation). Submission deadline: 2026-06-10.

**Experiment:** 24 bacs (3 NPK, 7 IRR2, 7 Yoshida, 7 Modified Hoagland) + 6 phytotron bacs testing light intensity. Daily measurements: pH (3 spatial points), EC (every 3 days, interpolated), temperature, humidity. Biomass only at Day 0 and Day 21 — this is the central constraint.

**Key data files:**
- `data/dataset azolla.xlsx` — raw pH/EC data (183 rows, no day column yet, no biomass yet)
- `data/Chlorophyll+ Carotenoid abs azolla.xlsx` — chlorophyll/carotenoid values for 30 samples (samples 1-30, all calculated)

**Critical biological findings:**
- Chl a/b ratio in actual data is 4-6 (NOT ~2.0 as stated in previous conversation's summary) — indicates stress (suboptimal humidity 40%, P-limitation in IRR2, high EC in Hoagland)
- pH acidified over time (not alkalized) — NH4+ uptake drives it — valid IoT proxy for metabolic activity
- Pink fronds in IRR2 = anthocyanin = P-deficiency stress
- Optimal light ~800 lux, photoinhibition > 1400 lux

**Why:** Thesis is about demonstrating a data-driven smart agriculture monitoring methodology, not building a production system. ML is positioned as proof-of-concept validated on experimental data.