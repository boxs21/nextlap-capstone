# Hito 2 · NextLap · Executed experiment, error analysis and SHAP

Team: Lab 8 · Members: Andy Ignacio Jesús Villarroel González, Agustín Eduardo Reyes Pereira · Date: 19 Nov 2026
Code: [`hito2_experiment.ipynb`](hito2_experiment.ipynb) · Saved row-level predictions: [`outputs/validation_predictions.csv`](outputs/validation_predictions.csv) · Run record: notebook §10 · How to run: [`README.md`](README.md)

Every number below is printed by the executed notebook `hito2_experiment.ipynb` in the section cited next to it (tables, plots and run record are saved in its cell outputs); the 1,808 row-level predictions are also saved to `outputs/validation_predictions.csv`. MAE is in seconds.

---

## 0. Hypothesis carried forward from Hito 1

**Hypothesis (unchanged).** The spread of the driver's recent clean laps (`recent_std_s`, ddof=1, same
clean-lap window of up to 3 laps as `recent_mean_s`, inside driver, race and stint) adds information that
the recent average cannot see. Short history (1 clean lap) gets the flag `has_history2 = 0` and the training
median. The definition, the window, the fallback and the decision rule are the ones written in
`Hito1/HITO1.md` §3–4 (commit `453863d`).

**What changed since Hito 1 and why.**
- The open question "should a 2-lap spread count as short history?" was not answered before the deadline. We kept the planned rule (2 laps are enough) and ran the 3-lap rule as a **disclosed sensitivity trial** (T2, §6), not as a replacement.
- The truncation check (Hito 1 check 2) was planned on a random training sample. We extended it to 500 training **and** 200 validation rows, because validation rows are the ones we score. The original entry is kept in `DECISION_LOG.md` (D1).

---

## 1. Evaluation validity

| Item | Evidence |
|---|---|
| **Race split** | Train R1–R7 (5,772 rows), validation R8 Austria (1,066) + R9 Britain (742). Executed table per race: [notebook §1](hito2_experiment.ipynb); asserted in code (notebook §1) |
| **Training-only fitting** | Each version is `R.make_model("OLS", numeric=…)` from the course helper; `.fit()` is called on `train` only (notebook §4). The imputer, scaler and one-hot encoder live inside that Pipeline. The fallback value 0.2579 s is read from the fitted imputer and asserted equal to the training median of `recent_std_s` (notebook §10 (`run → fallback_value_recent_std_s`)) |
| **Availability at *t*** | `history_features()` builds the spread on the **original lap timeline** (the 9 snapshots in `data/`, checksums verified by `R.load_snapshots`), before the clean-pair filter, with `rolling(3)` over laps ≤ *t* in the same driver, race and stint. Non-clean laps are left out of the window (same rule as the helper). `clean_pair` and `next_*` are not inputs |
| **Common row identities** | Predictions merged by `pair_id` with `validate="one_to_one"`; all three versions are scored on the same 1,808 IDs (SHA-256 of the sorted IDs in notebook §10 (run record), identical to Hito 1). The reference reproduces Hito 1 and the instructor's file row by row (max difference 4.3e-14 s and 2.8e-14 s, notebook §10 (`run → reference_reproduction`)) |

**Checks written in Hito 1, executed before reading any extension MAE** ([notebook §3](hito2_experiment.ipynb)):

| # | Check (Hito 1) | Result |
|---|---|---|
| 1 | Our window rebuilds the released `recent_mean_s` and `recent_count` | 7,580 rows, max diff 1.4e-14 s, 0 count mismatches → **pass** |
| 2 | Removing every lap after *t* does not change the feature | 700 rows (500 train + 200 validation, seed 414), 0 changed → **pass** |
| 3 | Last observation used ends at or before `end_time_s` of *t* | 0 of 7,580 rows after *t*; max (last used − end of *t*) = 0.0 s → **pass** |
| 4 | Every row gets a value or the declared fallback, row sets unchanged | 5,772 / 1,808 rows kept, no row dropped → **pass** |

**Coverage of the new input** ([notebook §3](hito2_experiment.ipynb)):

| Population | Rows | Spread observed | Fallback (short history) | Coverage |
|---|---|---|---|---|
| Train | 5,772 | 5,316 | 456 | 92.1% |
| Validation | 1,808 | 1,660 | 148 | 91.8% |
| R8 | 1,066 | 984 | 82 | 92.3% |
| R9 | 742 | 676 | 66 | 91.1% |

The final-evaluation data were never accessed (notebook §10 (`run → final_evaluation_data_accessed: false`)).

---

## 2. Executed comparison and ablation

Same model family (OLS), same preprocessing, same training rows, same 1,808 validation IDs, same metric.
The only difference between (b) and (c) is `recent_std_s`. Both are refitted from scratch.

| Version | Model and inputs | Validation n | MAE (s) | Output or run |
|---|---|---|---|---|
| (a) OLS reference | OLS, 6 published inputs | 1,808 | **0.3867** | [notebook §5](hito2_experiment.ipynb), column `pred_a_OLS_reference` in [`outputs/validation_predictions.csv`](outputs/validation_predictions.csv) |
| (c) Candidate | OLS, 6 inputs + `has_history2` + `recent_std_s` | 1,808 | **0.3697** | same files, `pred_c_candidate` |
| (b) Matched control | OLS, 6 inputs + `has_history2` (candidate with the spread removed) | 1,808 | **0.3877** | same files, `pred_b_matched_control` |

**Control MAE − candidate MAE = 0.3877 − 0.3697 = +0.0179 s** (positive = lower candidate MAE on these rows).
Reference − candidate = +0.0170 s. The flag alone does not help: reference − control = −0.0010 s.

**What it supports.** On the 1,808 validation pairs, adding the spread to the same OLS lowers MAE by
0.018 s (4.6% of the control's MAE). The pre-registered rule from Hito 1 — (c) below both (a) and (b),
overall **and** in each race — is met (notebook §10 (`run → decision_rule.hypothesis_supported: true`)). A paired
bootstrap that resamples whole driver-races (43 clusters, 2,000 reps, seed 414) gives a 95% interval of
**[+0.009, +0.028] s**, with all replicates positive (notebook §10 (`run → bootstrap`)).

**One limit.** This is a gain for a linear model on two races (Austria, Britain) and on clean pairs only. It is
small next to typical errors (0.37 s), and §3 shows it comes almost entirely from the ~22% of rows with a
high spread; it does not show that the spread helps on every lap or in other circuits.

**Check defined before implementing → observed result.** See the table in §1 (all four pass) and the decision
rule above (met). The checking procedure changed once (check 2 extended to validation rows); the original entry
is kept in `DECISION_LOG.md` D1.

---

## 3. Errors by context

**By race** ([notebook §5](hito2_experiment.ipynb)):

| Race | n | MAE control (b) | MAE candidate (c) | Control − candidate |
|---|---|---|---|---|
| R8 Austria | 1,066 | 0.3716 | 0.3588 | **+0.0128** |
| R9 Britain | 742 | 0.4107 | 0.3854 | **+0.0253** |
| Pooled (row-weighted) | 1,808 | 0.3877 | 0.3697 | **+0.0179** |

Row-weighted pooling: (1,066 × 0.0128 + 742 × 0.0253) / 1,808 = 0.0179 s, equal to the overall value (asserted in
code). The pattern agrees with the pooled result: both races improve, and the gain in Britain is about twice the
gain in Austria. Britain is also the race where all models err more.

**Declared slices** ([notebook §6](hito2_experiment.ipynb); spread terciles use **training** thresholds 0.170 s and 0.403 s):

| Slice (R8+R9) | n | MAE control | MAE candidate | Control − candidate |
|---|---|---|---|---|
| High spread (> 0.403 s) | 397 | 0.5707 | 0.4858 | **+0.0850** |
| Mid spread (0.170–0.403 s] | 627 | 0.3363 | 0.3391 | −0.0028 |
| Low spread (≤ 0.170 s) | 636 | 0.3106 | 0.3080 | +0.0025 |
| Short history (fallback) | 148 | 0.4455 | 0.4534 | **−0.0080** |

The gain is concentrated in high-spread rows, in both races (R8 +0.079, n=234; R9 +0.094, n=163). Short-history rows
do not improve, and in R9 they get worse (−0.023 s, n=66).

**Inspected validation case: `2026-R9-23-L7`** (Britain, driver 23, Williams, HARD tyres, tyre life 6).
Selection rule fixed before looking: the validation row where the spread moved the prediction the most,
|pred(c) − pred(b)|. Laps from the original timeline ([notebook §7](hito2_experiment.ipynb)):

| Lap | 3 | 4 | 5 | 6 | **7 = t** | 8 = t+1 (target) |
|---|---|---|---|---|---|---|
| Time (s) | 95.816 | 96.460 | 96.299 | 98.070 | **105.128** | 97.882 |

At *t* the window holds laps 5–7: mean 99.832 s, spread 4.671 s.

| | Prediction (s) | Observed (s) | Absolute error (s) |
|---|---|---|---|
| (a) reference | 102.034 | 97.882 | 4.152 |
| (b) control | 102.110 | 97.882 | 4.228 |
| (c) candidate | 99.121 | 97.882 | **1.239** (= \|97.882 − 99.121\|) |

Lap 7 was a one-off slow lap (7–9 s slower than laps 3–6, yet green flag and "clean"). The control extrapolates it; the
candidate reads the high spread and pulls the prediction 2.99 s down, so the error falls from 4.23 s to 1.24 s.

**Diagnosis (concrete limitation).** The spread is **symmetric and lags**. It says "the window is unstable" but not
*which* lap was the outlier or in which direction. The same driver shows the two failure sides:
- `2026-R9-23-L6` is the **largest candidate error** in validation (8.25 s): the slow lap 7 is the *target*, and nothing observable at lap 6 (spread 0.98 s) announces it. No history feature can fix this.
- `2026-R9-23-L12`: lap *t* = 100.539 s with spread 1.96 s; the candidate pulls the prediction to 98.05 s but the driver stayed slow (99.33 s), so (c) is **1.15 s worse than (b)** — the row where the spread hurts most.
Together with the short-history slice (no spread, worse in R9), this says the gain is a **reversion-after-outlier**
correction, not a general "pace is unpredictable" signal.

**Next check.** Replace the symmetric spread by a **signed** deviation, `current_lap_s` minus the mean of the
*previous* clean laps in the same window (direction-aware, same availability at *t*), and compare it with (c) as a
matched ablation on the same 1,808 IDs, per race and in the high-spread and short-history slices. Pre-stated
criterion: lower MAE than (c) in both races **and** no loss in the short-history slice. We will declare it as a new
trial in the decision log before running it.

---

## 4. SHAP · local explanation

**Model explained:** (c) candidate, fitted on the 5,772 training rows. We explain (c) because the ablation (§2)
leads us to retain it. **Explainer:** `shap.LinearExplainer` with an interventional
`shap.maskers.Independent` masker; **background = all 5,772 training rows** (transformed by the train-fitted
preprocessing). Inputs are explained after scaling and one-hot encoding; the mapping of each encoded column to its
original input is in [notebook §8](hito2_experiment.ipynb). One-hot columns are summed back to
`current_compound` / `current_team`. Because the target is seconds, contributions are in seconds.

**Row:** `2026-R9-23-L7` (the case in §3) · [notebook §8](hito2_experiment.ipynb)

| Input | Value at *t* | SHAP contribution (s) |
|---|---|---|
| `recent_mean_s` | 99.832 | +8.183 |
| `current_lap_s` | 105.128 | +6.668 |
| `recent_std_s` | 4.671 | **−2.467** |
| `current_compound` | HARD | +0.043 |
| `has_history2` | 1 | +0.031 |
| `current_team` | Williams | −0.027 |
| `current_lap_number` | 7 | −0.031 |
| `current_tyre_life` | 6 | −0.012 |
| **Base value** (mean training prediction) | | **86.734** |
| Base + sum of contributions | | 86.734 + 12.386 = **99.121** |
| Model prediction | | **99.121** |

**Reconstruction check:** base + contributions − prediction = 0.0 s for this row; max |error| over all 1,808 rows
= 1.4e-14 s (notebook §10 (`run → shap`)). As an independent check, the SHAP values equal coef × (x − background mean) for
every row (max difference 0.0).

**Meaning.** The base (86.73 s) is the average training prediction. `recent_mean_s` and `current_lap_s` push this row
up by +8.2 s and +6.7 s, mostly because Silverstone laps (~97 s) are longer than the average training lap. The spread of
4.67 s is far above its training mean, and it pushes the prediction **2.47 s below** what the other inputs give: for
this model, a very unstable window is associated with a faster next lap than the slow last lap suggests. That is a
statement about the model's prediction, not a causal effect and not an error amount. **The observed error is separate:**
|97.882 − 99.121| = **1.239 s**.

---

## 5. SHAP · global summary

**Sample:** all 1,808 validation rows (R8 1,066, R9 742). **Explainer and background:** as in §4.
**Aggregation:** per row, sum the SHAP values of the encoded columns of each original input, take the absolute value,
then average over rows. Table, bar plot and beeswarm of numeric inputs: [notebook §8](hito2_experiment.ipynb).

| Original input | Mean \|SHAP\| (s), all | R8 | R9 |
|---|---|---|---|
| `recent_mean_s` | 7.405 | 8.704 | 5.539 |
| `current_lap_s` | 4.273 | 5.005 | 3.220 |
| `recent_std_s` | 0.149 | 0.150 | 0.148 |
| `current_team` | 0.064 | 0.059 | 0.072 |
| `has_history2` | 0.058 | 0.056 | 0.060 |
| `current_compound` | 0.031 | 0.037 | 0.024 |
| `current_lap_number` | 0.022 | 0.025 | 0.017 |
| `current_tyre_life` | 0.008 | 0.008 | 0.008 |

**What it suggests.** The prediction is driven by the current pace (`recent_mean_s`, `current_lap_s`); their large values
mostly measure how far each circuit's lap time is from the average training lap (R8 laps ≈ 73 s are below the 86.7 s base,
R9 laps ≈ 97 s are above). `recent_std_s` is the third input, with a similar size in both races (≈0.15 s), and the beeswarm
shows its direction: high spread → lower predicted time. The other inputs move predictions by less than 0.07 s on average.

**Scope.** This ranks the associations of **this fitted OLS** on **these 1,808 R8–R9 clean pairs** relative to a
training background. It is not a causal ranking, not proof that an input lowers error (that claim belongs to the ablation in
§2), and it may change with other circuits, a different background or a non-linear model.

---

## 6. Decision and disclosed trials

**Decision: keep the spread** (version (c)) as our model for the final report. Scope of the claim: OLS, the planned
window (3 laps, min 2 clean laps), validation R8–R9 clean pairs, pooled gain +0.018 s (95% cluster-bootstrap interval
+0.009 to +0.028 s), concentrated in high-spread rows; no gain for short-history rows.

Additional trials, run **after** the planned comparison and **not** used to choose the reported model
([notebook §9](hito2_experiment.ipynb), control − candidate in s):

| Trial | Change | MAE all | All | R8 | R9 |
|---|---|---|---|---|---|
| T1 (planned) | as above | 0.3697 | +0.0179 | +0.0128 | +0.0253 |
| T2 | spread only with 3 clean laps (flag = has 3) | 0.3711 | +0.0156 | +0.0131 | +0.0192 |
| T3 | spread without the flag (control = reference) | 0.3652 | +0.0215 | +0.0160 | +0.0294 |

T2 is slightly worse than the planned rule, so the planned rule stays. T3 is lower, consistent with the flag alone not
helping (§2) and short-history rows not improving (§3). We did **not** switch to T3: it was chosen after seeing validation
results, so adopting it would be tuning on validation. It is recorded as a candidate for the next check.

---

## 7. Reproducibility

Command, environment, data manifest and the map from each section to its output: [`README.md`](README.md).
Decisions and trials: [`DECISION_LOG.md`](DECISION_LOG.md).

---

## 8. Each partner's short explanation

**Andy Ignacio Jesús Villarroel González:** I ran the four leakage and coverage checks before looking at any extension
MAE, and I set up the reproducibility side (README, data hashes, a notebook that passes Run All). The decision I defend is
extending the truncation check to validation rows instead of training rows only. Validation rows are the ones we score, so a
leak there would bias the (b) vs (c) comparison directly. With 500 training and 200 validation rows (seed 414), deleting every
lap after *t* changed 0 values, and our window rebuilt the released `recent_mean_s` on 7,580 rows with a max difference of
1.4e-14 s and 0 `recent_count` mismatches. Limitation: the truncation check uses a 700-row sample, not every row, and there
are only two validation races, so a leak that only shows up in rare rows could slip through. Running it on all 7,580 rows
is cheap and would be the next check. AI: I used Claude to draft `history_features()` and the assertions. I verified them
by checking that the rebuilt `recent_mean_s` matched the published column and that the reference reproduced Hito 1 row by
row (max difference 4.3e-14 s).

**Agustín Eduardo Reyes Pereira:** I ran the ablation (a)/(b)/(c), the error analysis by race and by slices, and the
local and global SHAP. The decision I defend is keeping (c) instead of switching to T3, even though T3 has a lower MAE
(0.3652 vs 0.3697 s). We ran T3 after seeing validation results, so switching would be tuning on validation. The planned (c)
already meets the pre-registered rule: control − candidate = +0.018 s, positive in R8 (+0.013) and R9 (+0.025), with a
cluster-bootstrap interval of [+0.009, +0.028] s. The T3 − T1 gap (0.0045 s) is small compared with that interval.
Limitation: almost all of the gain comes from high-spread rows (+0.085 s, n=397), and short-history rows get worse in R9
(−0.023 s, n=66). Next I would test a signed deviation instead of the symmetric spread. AI: I used Claude for the
cluster bootstrap and the SHAP code. I verified that base + contributions rebuilds every prediction (max 1.4e-14 s), and I
re-read case `2026-R9-23-L7` in the raw lap file to confirm laps 5–7 and the target lap 8.
