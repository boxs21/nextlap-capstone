# Decision log · NextLap · Hito 2

Team: Lab 8 · Members: Andy Ignacio Jesús Villarroel González, Agustín Eduardo Reyes Pereira
Hito 1 log (unchanged): [`../Hito1/DECISION_LOG.md`](../Hito1/DECISION_LOG.md)

---

## D1 · Checks defined before implementing, and their results

**Original entry (Hito 1, `HITO1.md` §4, commit `453863d`), kept as written:**

| Check | What would make it fail |
|---|---|
| Rebuilding `recent_mean_s` from the raw timeline with our code gives exactly the released column | Any difference → our window is not the reference window |
| Removing every lap after *t* does not change `recent_std_s` (random sample of training rows) | Any change → the feature uses future laps (leakage) |
| The last observation used by the feature ends at or before `end_time_s` of *t* | Any later timestamp |
| Every train/validation row gets a feature value or the declared fallback | Row count different from 5,772 / 1,808, or a changed `pair_id` set |

**Revision (19 Nov), before any extension MAE was computed:** check 2 now uses 500 training rows **plus 200 validation
rows** (seed 414). Reason: validation rows are the ones we score, so a leak there would bias the comparison directly. Check 1
was also extended to compare `recent_count`, not only `recent_mean_s`.

**Observed results** ([notebook §3](hito2_experiment.ipynb)): (1) 7,580 rows, max diff 1.4e-14 s, 0 count
mismatches · (2) 700 rows, 0 changed · (3) 0 rows use a lap after *t* · (4) 5,772 / 1,808 rows, 456 / 148 fallbacks. All pass.

**Pre-registered decision rule (Hito 1):** supports the hypothesis if (c) has lower MAE than (a) and (b), overall and in
R8 and R9. **Observed:** met in all three populations (notebook §10 (`run → decision_rule`)).

---

## D2 · Trials (settings and results)

All: OLS from the course helper, preprocessing fitted on 5,772 training rows, scored on the same 1,808 validation `pair_id`s,
MAE in seconds, seed 414. Source: [notebook §5](hito2_experiment.ipynb), [notebook §9](hito2_experiment.ipynb).

| Trial | Setting | MAE all | Control − candidate (all / R8 / R9) | Outcome |
|---|---|---|---|---|
| (a) | Reference, 6 inputs | 0.3867 | — | Baseline, reproduces Hito 1 and instructor (≤ 4.3e-14 s) |
| (b) | (a) + `has_history2` | 0.3877 | reference − control = −0.0010 / +0.0019 / −0.0051 | **Unsuccessful on its own**: the flag does not lower MAE overall and is worse in R9 |
| T1 = (c) | (b) + `recent_std_s`, window 3, min 2 clean laps (planned) | 0.3697 | +0.0179 / +0.0128 / +0.0253 | **Retained** |
| T2 | spread and flag with min **3** clean laps | 0.3711 | +0.0156 / +0.0131 / +0.0192 | Not adopted: lower gain than T1 overall and in R9 |
| T3 | spread **without** the flag, control = (a) | 0.3652 | +0.0215 / +0.0160 / +0.0294 | Not adopted (chosen after seeing validation); kept as next check |

T2 and T3 were run after the planned comparison and were not used to pick the reported model.

---

## D3 · Review after checking (change or justified retention)

**Finding.** After the planned run we checked the result by race, by declared slices and with a cluster bootstrap
(`HITO2.md` §2–3). Three findings: the flag alone does not help (b vs a); short-history rows get worse with (c) in R9
(−0.023 s, n=66); and T3 (no flag) is lower than T1.

**Decision: retain the planned design (c), justified.** The planned version meets the pre-registered rule in both races,
and the bootstrap interval of its gain excludes zero ([+0.009, +0.028] s). Switching to T3 after seeing its validation MAE
would be selecting on validation, and the T3 − T1 gap (0.0045 s) is small compared with that interval. We record T3 and the
short-history loss as the reason for the next check instead of changing the reported model.

**Verification of the numbers we report.** The notebook asserts: pooled MAE from race values weighted by rows equals the
overall MAE; the reference equals Hito 1 and the instructor file row by row; SHAP base + contributions reconstructs every
prediction (max 1.4e-14 s) and SHAP equals coef × (x − background mean). The case `2026-R9-23-L7` was re-read from the raw lap
file ([notebook §7](hito2_experiment.ipynb)) to confirm laps 5–7 and the target lap 8.

---

## D4 · Decision: keep or discard the change

**Keep `recent_std_s`** (with the planned `has_history2` fallback). Tested scope: OLS, window of up to 3 clean laps in the same
driver/race/stint, R8 Austria and R9 Britain clean pairs. Effect: −0.018 s MAE vs the matched control, concentrated in rows with
spread > 0.403 s (+0.085 s, n=397); no gain for short-history rows. We do not claim it for pit, safety-car or non-clean laps, for
other circuits or for other model families.

---

## D5 · Next check

Replace the symmetric spread by a signed deviation (`current_lap_s` − mean of the previous clean laps in the window), and
compare it with (c) as a matched ablation on the same 1,808 IDs, per race and in the high-spread and short-history slices.
Criterion stated now: lower MAE than (c) in both races and no loss in the short-history slice. Motivation: case
`2026-R9-23-L12`, where the spread over-corrects a pace drop that persisted (c is 1.15 s worse than b), and the short-history
loss in R9.

---

## E · Contributions

| Member | Contribution | Decision they can explain |
|---|---|---|
| Andy Ignacio Jesús Villarroel González | `history_features()` on the original timeline; the four Hito 1 checks (leakage, availability at *t*, coverage), run before reading any extension MAE; reproducibility (README, data hashes, Run All, run record) | Extend check 2 to 200 validation rows (D1): those are the scored rows. Result: 700 rows, 0 changed |
| Agustín Eduardo Reyes Pereira | Ablation (a)/(b)/(c) and trials T2/T3; errors by race and by slice; inspected case `2026-R9-23-L7`; cluster bootstrap; local and global SHAP | Keep (c), do not switch to T3 (D3): T3 was chosen after seeing validation. (c) meets the rule in R8 and R9, 95% CI [+0.009, +0.028] s |

Both members reviewed the final report and the decision in D4.

---

## F · AI use and verification

We used Claude (Anthropic) as a coding assistant in this hito. It did not choose the hypothesis, the decision rule or the
reported model; those were fixed in Hito 1 or decided by us (D3).

| Use | Representative prompt (paraphrased) | Accepted / changed | How we verified it |
|---|---|---|---|
| `history_features()` and leakage assertions (Andy) | "Write a function that computes the std (ddof=1) of the last up to 3 clean laps up to and including *t*, inside driver, race and stint, on the original lap timeline, plus asserts for the Hito 1 checks" | Accepted the code after review. The window rule (same as the course helper) and the extension of check 2 to validation rows (D1) were our decisions, not the assistant's | Rebuilt `recent_mean_s` / `recent_count` match the released columns (7,580 rows, max diff 1.4e-14 s, 0 mismatches); truncation check 0 changed in 700 rows; reference reproduces Hito 1 and the instructor file (≤ 4.3e-14 s) |
| Cluster bootstrap and SHAP code (Agustín) | "Paired bootstrap of the MAE difference resampling whole driver-races; then SHAP LinearExplainer for the OLS pipeline with one-hot columns summed back to the original input" | Accepted after review. Background (all 5,772 training rows), 43 clusters and seed 414 are the settings we checked in the run record | Base + contributions rebuilds every prediction (max 1.4e-14 s); SHAP equals coef × (x − background mean); case `2026-R9-23-L7` re-read from the raw lap file |
| Drafting and editing the report text | Summaries of notebook outputs into `HITO2.md` sections | Rewritten and shortened by us | Every number was checked by us against the executed notebook's cell outputs and the run record (§10) |
