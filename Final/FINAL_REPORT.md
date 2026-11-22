# NextLap · Final report

**IIT414W Artificial Intelligence Workshop · Team Lab 8 (G08)**
**Members:** Andy Ignacio Jesús Villarroel González · Agustín Eduardo Reyes Pereira
**Repository:** https://github.com/boxs21/nextlap-capstone
**Commit (SHA):** `<FULL_COMMIT_SHA>`
**Date:** 22 November 2026
**Nominated candidate (improvement bonus):** OLS + `has_history2` + `recent_std_s` (version (c) = T1), frozen since Hito 2

| Evidence | Where |
|---|---|
| Planned experiment, error analysis, SHAP | [`Hito2/hito2_experiment.ipynb`](../Hito2/hito2_experiment.ipynb) (§ numbers below = notebook sections) |
| Revision after feedback (R1–R4) | [`Final/final_revision.ipynb`](final_revision.ipynb) |
| Row-level predictions | [`Hito2/outputs/validation_predictions.csv`](../Hito2/outputs/validation_predictions.csv), [`Final/outputs/final_validation_predictions.csv`](outputs/final_validation_predictions.csv) |
| How to run, data, environment | [`Final/README.md`](README.md) |
| Decisions, trials, contributions, AI use | [`Final/DECISION_LOG.md`](DECISION_LOG.md) (Hito 2 log: [`Hito2/DECISION_LOG.md`](../Hito2/DECISION_LOG.md)) |

MAE is in seconds on the 1,808 validation clean pairs (R8 Austria 1,066 · R9 Britain 742) unless stated otherwise.

---

## Summary

At the end of lap *t* we predict the same driver's next lap time (`next_lap_s`). Our hypothesis: the **spread** of the
driver's last up-to-3 clean laps (`recent_std_s`) adds information that the recent average cannot see.

**Decision: keep the spread.** Against the matched control (same OLS without the spread) it lowers MAE by **0.0179 s**
(0.3877 → 0.3697), in both races, with a driver-race cluster-bootstrap 95% interval of **[+0.009, +0.028] s**. It also beats
the six-input OLS reference (0.3867) by 0.0170 s. The gain is concentrated in windows with three clean laps and a high spread;
rows with short history do not improve.

**Revision after feedback.** The Hito 2 next check (a signed deviation) was redundant: it is an exact multiple of
`current_lap_s − recent_mean_s`, already in OLS (verified; executed, it changes MAE by −0.0001 s). We now state what the
spread adds that OLS cannot build (the *size* of lap *t*'s deviation and the disagreement between earlier laps), report results
by race **and** history length, and compare T1 with the exploratory T3 directly. T3 is slightly better on validation, but its
advantage is not consistent across races, so the nominated candidate stays T1 (§6). At the Demo Day the instructor asked why
the lap behind our inspected case was slow: we checked, and the driver was being **lapped by six cars** under blue flags, a
traffic loss the track-status field does not record. About half of the pooled gain comes from the 83 rows where the driver was
being lapped during lap *t*; on the other 1,725 rows the gain is +0.0095 s, positive in both races (§3, §6).

---

## 1. Evaluation validity

**Split and fitting (C1a).** Target `next_lap_s` at the end of lap *t*. Train R1–R7 (5,772 pairs), development validation
R8–R9 (1,808). Every model is `R.make_model("OLS", …)` from the course helper: imputer, scaler and one-hot encoder live
inside the pipeline and `.fit()` is called on training rows only (Hito2 notebook §4, Final notebook "Fit all versions"). The
spread's fallback value (0.258 s) is read from the fitted imputer and asserted equal to the training median. The split per race
is printed in Hito2 notebook §1. The instructor's final-evaluation data were never accessed.

**Input availability (C1b).** `recent_std_s` is built from the released lap snapshots (`data/race_2026_r0*_laps_v2.csv`,
checksums verified) on the **original timeline**, before the clean-pair filter: standard deviation (ddof = 1) of the clean laps
≤ *t* in a rolling window of 3, inside driver, race and stint, using the same clean-lap rule as the helper's `recent_mean_s`.
Four checks fixed in Hito 1 were run before reading any extension MAE (Hito2 notebook §3):

| Check | Result |
|---|---|
| Our window rebuilds the released `recent_mean_s` / `recent_count` | 7,580 rows, max diff 1.4e-14 s, 0 mismatches |
| Deleting every lap after *t* does not change the feature | 700 rows (500 train + 200 validation, seed 414): 0 changed |
| Last observation used ends at or before `end_time_s` of *t* | 0 of 7,580 rows after *t* |
| Every row has a value or the declared fallback | 5,772 / 1,808 kept; fallback (1 clean lap: `has_history2 = 0` + training median) on 456 / 148 rows |

`clean_pair` uses *t+1* conditions, so it only **defines the evaluated population** (normal green-flag laps) and is never an
input; `next_*` fields are excluded. The results therefore do not describe pit, safety-car or non-clean laps.

**Common validation IDs (C1c).** All versions are joined by `pair_id` with one-to-one validation; the 1,808 IDs are unique and
identical to the reference set (SHA-256 of the sorted IDs `f453a98d…` in both run records). The reference reproduces Hito 1 and
the instructor's file row by row (≤ 4.3e-14 s), and the Final notebook asserts that the planned predictions are identical to
Hito 2 (≤ 1.4e-14 s). Coverage of the spread: 1,660 / 1,808 validation rows (91.8%), R8 92.3%, R9 91.1%.

---

## 2. Comparison and decision

**Executed comparison (C2a).** Same family (OLS), preprocessing, training rows, validation IDs and metric. The control is the
candidate with only the spread removed, refitted from scratch.

| Version | Inputs | n | MAE (s) | Saved run |
|---|---|---|---|---|
| (a) OLS reference | 6 published inputs | 1,808 | **0.3867** | `pred_a_OLS_reference` |
| (c) Candidate = T1 | (a) + `has_history2` + `recent_std_s` | 1,808 | **0.3697** | `pred_c_candidate` |
| (b) Matched control | (a) + `has_history2` | 1,808 | **0.3877** | `pred_b_matched_control` |

Columns of `Hito2/outputs/validation_predictions.csv`; table in Hito2 notebook §5.

**Measured difference (C2b).** Control − candidate = 0.3877 − 0.3697 = **+0.0179 s** (positive = lower candidate MAE),
4.6% of the control's MAE. Reference − candidate = +0.0170 s. Reference − control = −0.0010 s: the flag alone does not help.
Paired cluster bootstrap over driver-races (43 clusters, 2,000 reps, seed 414): **[+0.009, +0.028] s**, all replicates positive
(reproduced in the Final notebook R4).

**Decision from evidence (C2c).** We keep the spread because the only difference between (b) and (c) is that input, and removing
it raises MAE overall and in each race (§3), with an interval that excludes zero. This meets the rule written in Hito 1 before
running the experiment: (c) below (a) and (b), overall and in R8 and R9. The claim is bounded to OLS, these two races and clean
pairs; §3 shows where it does not hold.

---

## 3. Contextual errors and scope

**Race comparison (C3a)** (Hito2 notebook §5):

| Race | n | Control MAE | Candidate MAE | Control − candidate |
|---|---|---|---|---|
| R8 Austria | 1,066 | 0.3716 | 0.3588 | **+0.0128** |
| R9 Britain | 742 | 0.4107 | 0.3854 | **+0.0253** |
| Pooled | 1,808 | 0.3877 | 0.3697 | **+0.0179** |

Row-weighted: (1,066 × 0.0128 + 742 × 0.0253) / 1,808 = 0.0179 s, the pooled value (asserted in code). Both races agree in
sign; the gain in Britain is about twice the gain in Austria, and Britain is where every model errs more.

**By race and history length** (new, Final notebook R3; history = clean laps in the window at *t*):

| Race | History | n | Control | Candidate | Control − candidate |
|---|---|---|---|---|---|
| R8 | 3 laps | 893 | 0.3688 | 0.3514 | **+0.0174** |
| R8 | 2 laps | 91 | 0.3890 | 0.4131 | −0.0240 |
| R8 | 1 lap (fallback) | 82 | 0.3830 | 0.3786 | +0.0045 |
| R9 | 3 laps | 594 | 0.3913 | 0.3585 | **+0.0328** |
| R9 | 2 laps | 82 | 0.4611 | 0.4508 | +0.0102 |
| R9 | 1 lap (fallback) | 66 | 0.5230 | 0.5464 | −0.0234 |
| R8+R9 | 3 laps | 1,487 | 0.3778 | 0.3543 | **+0.0235** |
| R8+R9 | 2 laps | 173 | 0.4232 | 0.4310 | −0.0078 |
| R8+R9 | 1 lap (fallback) | 148 | 0.4455 | 0.4534 | −0.0080 |

The gain comes from 3-lap windows (82% of rows), positive in both races. With 2 laps or the fallback the sign changes between
races and the groups are small, so we make no claim there. Within 3-lap windows the gain concentrates in high-spread rows
(spread > 0.403 s, training tercile: +0.085 s, n = 397; Hito2 notebook §6).

**Inspected case (C3b): `2026-R9-23-L7`** (Britain, driver 23, HARD, tyre life 6). Rule fixed before looking: the row where the
spread moved the prediction most, |pred(c) − pred(b)|. Laps 5–7 = 96.299 · 98.070 · **105.128** s (window mean 99.832,
spread 4.671). Observed next lap 97.882 s.

| | Prediction (s) | Absolute error (s) |
|---|---|---|
| (b) control | 102.110 | 4.228 |
| (c) candidate | 99.121 | **1.239** = \|97.882 − 99.121\| |

**Why lap 7 was slow** (Demo Day question; Final notebook R5). On lap 7 only driver 23 lost more than 2 s against his previous
lap (1 of 22 drivers; field median +0.11 s), with green track status for everyone. Reconstructing on-track passes from the
official timing (constant speed within each lap, no positions available), driver 23 was **lapped by six cars on lap 7** (Norris,
Antonelli, Verstappen, Hamilton, Hadjar, Russell) and by Leclerc on lap 6. This matches the public record: after a lap-1 contact
he pitted for a new front wing, rejoined almost a lap down and was caught in a blue-flag chain [1, 2]. The race was dry [3],
with neutralisations only on lap 22 and at the end [4]. So the slow lap was a transient traffic loss, and the spread corrected
it for the right reason; but blue flags are not in the track-status field, so our "clean" population includes traffic laps.

**Scope and next check (C3c).** The case shows what the spread does: after a transient loss it predicts a recovery. The same
driver shows the limits. At `2026-R9-23-L6` the slow lap is the *target* and nothing in his own history announces it (largest
candidate error, 8.25 s). At `2026-R9-23-L12` he was lapped again on laps 12 and 13, so the slow pace continued and the candidate
is 1.15 s worse than the control. A declared diagnostic slice (retrospective, not an input) shows how much of the result is
traffic:

| Rows | n | Control | Candidate | Control − candidate |
|---|---|---|---|---|
| Driver lapped during lap *t* | 83 | 1.1317 | 0.9375 | **+0.1942** |
| Not lapped during lap *t* | 1,725 | 0.3519 | 0.3424 | **+0.0095** |
| — R8 not lapped / R9 not lapped | 1,018 / 707 | | | +0.0046 / +0.0164 |

The 83 lapped rows (4.6%) supply about half of the pooled gain (83 × 0.1942 / 1,808 = 0.0089 s of 0.0179 s); the rest is a smaller
but positive gain in both races. Short-history rows add a second limit: they do not benefit, and in R9 they are worse than the
reference by 0.144 s (n = 66).
**Next check:** a traffic input available at *t*, built from the as-of timing of the other cars: whether faster cars a lap ahead are
within a short gap behind the driver at the end of lap *t* (about to lap him). Compared with T1 on the same IDs, by race, history
length and lapped/not lapped, declared as a trial before running and judged on held-out rows. A frozen OLS fallback for
`recent_count = 1` rows stays open as a second check.

---

## 4. Model interpretation

**Local SHAP (C4a).** Model: (c) candidate fitted on the 5,772 training rows. Explainer: `shap.LinearExplainer`, independent
(interventional) masker, background = all 5,772 training rows after the train-fitted preprocessing. One-hot columns are summed
back to their original input. Row `2026-R9-23-L7` (Hito2 notebook §8):

| Input | Value at *t* | Contribution (s) |
|---|---|---|
| `recent_mean_s` | 99.832 | +8.183 |
| `current_lap_s` | 105.128 | +6.668 |
| `recent_std_s` | 4.671 | **−2.467** |
| other 5 inputs | | +0.003 in total |
| **Base** (mean training prediction) | | 86.734 |
| Base + contributions | | 86.734 + 12.386 = **99.121** = model prediction |

Reconstruction error 0.0 s on this row and ≤ 1.4e-14 s on all 1,808 rows; SHAP equals coef × (x − background mean). The base is
the average training prediction; the pace inputs push this row up mainly because Silverstone laps are longer than the average
training lap, and the spread pushes it **2.47 s below** what the other inputs give. That explains the **prediction**, not a
causal effect. The **observed error is separate**: |97.882 − 99.121| = 1.239 s.

**Global SHAP (C4b).** Sample: all 1,808 validation rows. Same explainer and background. Method: per row, sum the encoded
contributions of each original input, take |·|, average over rows (Hito2 notebook §8, table and bar plot).

| Input | Mean \|SHAP\| (s) all | R8 | R9 |
|---|---|---|---|
| `recent_mean_s` | 7.405 | 8.704 | 5.539 |
| `current_lap_s` | 4.273 | 5.005 | 3.220 |
| `recent_std_s` | 0.149 | 0.150 | 0.148 |
| `current_team` | 0.064 | 0.059 | 0.072 |
| `has_history2` | 0.058 | 0.056 | 0.060 |
| others | ≤ 0.031 | | |

The pace inputs dominate mostly because each circuit's lap time is far from the 86.7 s training base. The spread is third, with a
similar size in both races, and the beeswarm shows higher spread → lower prediction. Scope: associations of this fitted OLS on these
1,808 R8–R9 rows against a training background; not a causal ranking and not evidence of lower error (that is the ablation, §2).

---

## 5. Reproducibility and decisions

**Runnable repository (C5a).** URL and commit on page 1. Data are the course release
`IIT414W_W09_Fri_Capstone_StudentPack_v2.zip` (SHA-256 `2ad128c6…2747a`), extracted at the repository root into the git-ignored
`W09_StudentPack_v2/` (or `NEXTLAP_DATA`). Environment: Python 3.10.12 with pinned versions in `Final/requirements.txt`.
Seed 414 (truncation-check sample, bootstrap; OLS has no randomness). Commands, from the repository root:

```bash
pip install -r Final/requirements.txt
jupyter nbconvert --to notebook --execute --inplace Hito2/hito2_experiment.ipynb   # planned experiment + SHAP
jupyter nbconvert --to notebook --execute --inplace Final/final_revision.ipynb     # revision R1–R5
```

Full checksums, the map from each section to its output, and the run records are in `Final/README.md` and in the last cell of
each notebook.

**Decision record (C5b).** `Final/DECISION_LOG.md` lists every trial with its setting and result, including the unsuccessful
ones (flag alone, T2, T4 signed deviation), the decisions they support, and the decision or check each member contributed.

**AI-use record (C5c).** Actual use and the checks performed on adopted suggestions are recorded in `Final/DECISION_LOG.md` §E
(Hito 2 use in `Hito2/DECISION_LOG.md` §F).

---

## 6. Revision after feedback

**Priority (C6a).** Hito 2 feedback (20 Nov): *"With three clean laps, the proposed signed deviation equals 1.5 × (current_lap_s −
recent_mean_s). Both terms are already in OLS. Also, the bootstrap interval compares T1 with the matched control. It does not
measure T1 versus T3."* Requested check: specify what the next variant adds to those two inputs; compare on the same IDs, by race
and history length; keep T3 separate; use a direct paired comparison for T1 versus T3. **Demo Day (20 Nov, oral):** the instructor
asked whether we had investigated why driver 23's lap 7 was slow in our inspected case. We had not, and argued that the cause did
not matter; rows 5–6 below are our response.

**What changed and why (C6a), with the check and its result (C6b)** — all in `Final/final_revision.ipynb`; predictions in
`Final/outputs/final_validation_predictions.csv`:

| # | Change | Check | Result |
|---|---|---|---|
| 1 | **Withdrew** the signed deviation as next check. | Identity on the data, then OLS with it added (T4) vs the control. | signed_dev = 1.5 × (t − mean) for 3 laps and 2 × (t − mean) for 2 laps (max diff 3.6e-14 s, 1,660 rows). Control − T4: **−0.0001 s** overall (R8 +0.0005, R9 −0.0009): it adds nothing. |
| 2 | **Specified** what the spread adds beyond `current_lap_s` and `recent_mean_s`. | Exact decomposition for 3 laps *a, b, t*: **spread² = ¾ (t − mean)² + ¼ (a − b)²**; then each part alone (exploratory). | Identity holds (max diff 2e-13 s², 1,487 rows). OLS is linear, so it can use the signed t − mean but not its **size**, nor the **disagreement between earlier laps** (a − b)². Each part alone vs control: T5 \|t − mean\| **+0.0153 s** (R8 +0.0113, R9 +0.0210); T6 earlier-lap spread **+0.0114 s** (R8 +0.0099, R9 +0.0135). The spread combines both (+0.0179). |
| 3 | **Added** results by race and history length on the same IDs. | Table in §3, row-weighted pooling asserted. | Gain in 3-lap windows in both races (+0.017 / +0.033); no consistent gain with 2 laps or the fallback. |
| 4 | **Replaced** the Hito 2 T1-vs-T3 argument with a direct paired comparison. | Per-row \|err T1\| − \|err T3\|, paired driver-race cluster bootstrap (seed 414). | MAE(T1) − MAE(T3) = **+0.0045 s** [+0.0009, +0.0085]. By race: R8 +0.0012 [−0.0042, +0.0060]; R9 +0.0093 [+0.0045, +0.0144]. By history: 3 laps +0.0012 [−0.0014, +0.0038]; 2 laps +0.0047 [−0.0036, +0.0126]; fallback +0.0371 [−0.0149, +0.0933]. |
| 5 | **Investigated** the cause of the slow lap in the inspected case (Demo Day). | Field check on lap 7 (all 22 drivers) and on-track passes reconstructed from official timing; public race reports. | Only driver 23 lost > 2 s (field median +0.11 s), all green. He was lapped by 6 cars on lap 7 (1 on lap 6, 2 on lap 12, 1 on lap 13), consistent with the team report of a blue-flag chain [1, 2]; dry race [3], neutralisations far from lap 7 [4]. |
| 6 | **Bounded** the claim by traffic and **corrected** the L12 explanation; **replaced** the next check. | Diagnostic slice: lapped during *t* vs not, by race. | Lapped rows (n = 83): +0.194 s; others (n = 1,725): +0.0095 s (R8 +0.0046, R9 +0.0164). Half the gain comes from traffic laps; the rest is positive in both races. Next check: a traffic input available at *t* (§3). |

**Correction of a Hito 2 statement.** Hito 2 argued that the T3 − T1 gap was "small compared with" the control-vs-T1 interval. That
interval measures a different contrast. The direct paired comparison (row 4) shows T3 is better on validation overall.

**Decision retained, with reason.** We keep **T1** as the planned result and the nominated candidate:
- T3 differs from T1 only by removing `has_history2`. The flag's effect changes sign between races on short-history rows (vs the
  reference: R8 0.383 vs 0.391, helps; R9 0.523 vs 0.402, hurts), so T3's advantage rests on one race and few rows.
- On 3-lap windows (82% of rows) and in R8, the paired interval includes zero.
- T3 was chosen after seeing validation; adopting it would be selection on two races. We report it separately as exploratory
  evidence, and the short-history problem stays open as a secondary check (§3).
- The traffic finding (rows 5–6) does not change the decision: the gain remains positive in both races on rows without lapping,
  and the candidate still meets the planned rule. It changes what we claim the spread does: it mostly corrects transient
  losses, of which being lapped is the clearest example.

---

## Improvement bonus nomination

Candidate: (c) = T1, OLS with the six reference inputs + `has_history2` + `recent_std_s`, trained on R1–R7 only, preprocessing
fitted on train, seed 414. Validation (selection evidence, R8–R9, n = 1,808): reference 0.3867 s, candidate 0.3697 s, gain
0.0170 s (4.4%). Feature generation needs only the as-of lap history (driver/race/stint keys, lap time, end time and status
fields ≤ *t*), which matches the published input contract. Fallback for rows without two clean laps: training median + flag, 148
validation rows (8.2%). No external source, no test access. Run command and commit as above; trial log in `Final/DECISION_LOG.md`.

---

## Sources (race context, §3 and §6)

1. Williams Racing, *Report: An unfortunate weekend at Silverstone comes to a close*, 5 Jul 2026. https://www.williamsf1.com/articles/5377a1c7-dd17-42b3-9b56-db4bdccbfe94/report-britain-sunday-2026
2. Alex Albon, *Report: Silverstone struggles*. https://www.alexalbon.com/articles/8b52e81d-e3ac-4b6a-be36-26736b90279c/report-silverstone-struggles
3. Sofascore, *British GP 2026: Leclerc leads Ferrari to Silverstone win* (52 dry laps). https://www.sofascore.com/th/news/charles-leclerc-wins-the-british-gp
4. The Race, *British GP: Leclerc wins after safety car finish* (VSC on lap 22, safety-car finish). https://www.the-race.com/formula-1/british-gp-leclerc-wins-safety-car-finish-f1/
