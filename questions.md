# Open Questions — Must Resolve Before Implementation

These are blocking questions that require your input. Grouped by priority.
Read this file and answer each question before we start writing code.

---

## PRIORITY 1 — Blocking for Chlorophyll Analysis (can start immediately)

**Q1. Chlorophyll sample-to-bac mapping**
The file has samples 1–30 with calculated Chl a, b, Total Chl, Carotenoids.
Which sample number corresponds to which medium and which replicate?
Example format: "Sample 1 = NPK rep 1, Sample 2 = NPK rep 2, ..., Sample 25 = Phytotron bac 1..."
→ Without this mapping, ANOVA cannot be run.

**Q2. Are all 30 chlorophyll samples now measured?**
CLAUDE.md says 16 of 24 were measured when that was written, and sample 17 was excluded (light-degraded).
The current file has computed values for all 30 rows (samples 1–30).
- Are samples 1–30 the complete final dataset?
- Is sample 17 still to be excluded, or does the file already handle it?
- Note: looking at the actual calculated values, "sample 17" in the file has Total Chl = 9.31 µg/mL which does not look anomalous. Clarify.

**Q3. Chl a/b ratio discrepancy**
CLAUDE.md says Chl a/b was ~2.0 across all samples.
The actual file data shows ratios of **4.0–6.4** across all 30 samples.
Which is correct? (The actual calculation from the absorbances gives 4–6, consistent with the Lichtenthaler 1987 formula at these absorbance ratios.)
→ Literature says Azolla Chl a/b ~2–3 healthy, 4–6 under stress. This is biologically significant (stress indicator) and must be interpreted correctly in the thesis.

**Q4. Are phytotron bacs included in the chlorophyll file?**
24 main bacs + 6 phytotron = 30. Does sample 1–24 = main experiment bacs and 25–30 = phytotron?
Or is another ordering used?

---

## PRIORITY 2 — Blocking for Biomass ANOVA and Framing 1 Modeling

**Q5. Final biomass (Day 21) per bac**
Provide the fresh weight (grams, after 30-sec drain) for each of the 24 main bacs.
Format: Medium, Replicate, Initial_biomass_g, Final_biomass_g
→ This is the primary target variable. Nothing can be modeled without it.

**Q6. Initial biomass (Day 0) per bac**
Same 24 bacs. If the inoculation was standardized, give the actual measured values.

**Q7. Elemental composition of each medium**
Provide the mmol/L concentrations of each element (N, P, K, Ca, Mg, Fe, etc.) for:
- NPK
- IRR2
- Yoshida
- Modified Hoagland
→ These are key features in Framing 1 (richer than just medium type as a dummy variable).

---

## PRIORITY 3 — Blocking for Daily Time-Series and Framing 2

**Q8. Day assignment in the main dataset**
The current Excel file has 183 rows of pH/EC data with no date or day column.
Looking at the data, there appear to be ~3 measurement occasions:
- Rows 0–50: first occasion (~51 rows covering all bacs)
- Rows 51–~120: second occasion
- Rows ~120–182: third occasion
Is this correct? What day does each occasion correspond to?
Example: "Occasion 1 = Day 0, Occasion 2 = Day 7, Occasion 3 = Day 14, ..."
→ Without this, Framing 2 cannot be built and pH trajectory analysis cannot be done.

**Q9. What were the actual measurement days?**
Given partial harvest every 3–4 days, what were the exact days on which you measured pH (and EC)?
Example: "pH measured on Day 0, 3, 7, 10, 14, 17, 21"
"EC measured on Day 0, 7, 14, 21"

**Q10. Temperature and humidity — were they recorded daily?**
Were these the same for all bacs (room-level sensors), or per-bac?
Do you have the daily values (21 days)?

**Q11. Light intensity values per bac**
Light intensity (lux) for each of the 24 main bacs — was this fixed per bac position, or did it vary?
And the 6 phytotron bacs — give exact positions (high/medium/low) and the lux values measured.

---

## PRIORITY 4 — Design decisions (answer before we start modeling)

**Q12. Are the phytotron bacs part of the main ML model or analyzed separately?**
The phytotron bacs (n=6) test light intensity specifically and had standardized initial biomass.
Recommendation: analyze them separately (light intensity effect analysis) rather than mixing with the 24 main bacs.
Do you agree?

**Q13. Drop TDS and Salinity from the model?**
TDS and Salinity are very highly correlated with EC (r > 0.95 expected).
Including all three inflates dimensionality without information gain.
Recommendation: keep only EC, drop TDS and Salinity.
Do you agree?

**Q14. Is there any correction/intervention data (water additions, nutrient top-ups)?**
Were correction_flag or nutrient_top_up_flag events recorded?
If yes, do you have the log? These are relevant features (correction count = proxy for how much the system drifted).

**Q15. Biochemical data status**
The CLAUDE.md mentions: Proline, Soluble sugars, MDA, SOD, CAT, GPX.
Are any of these measured yet, or are samples still at -20°C?
These would strengthen the thesis results chapter significantly if available.

---

## DATA FORMAT REQUEST

When you add data from your notebook, please add it as:
1. A new Excel sheet in `dataset azolla.xlsx` called `bac_summary` with columns:
   `medium | replicate | initial_biomass_g | final_biomass_g | light_lux`
   (one row per bac, 24 rows)

2. A new Excel sheet called `daily_measurements` with columns:
   `bac_id | medium | replicate | day | pH_p1 | pH_p2 | pH_p3 | pH_mean | pH_std | EC_uS | temperature_C | humidity_pct | photoperiod_h | light_lux | correction_flag | harvest_flag`
   (one row per bac per measurement day)

This structure will let us implement both framings directly.