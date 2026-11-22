# Decision log · NextLap · Final report

Team: Lab 8 (G08) · Members: Andy Ignacio Jesús Villarroel González, Agustín Eduardo Reyes Pereira
Earlier logs (unchanged): [`../Hito1/DECISION_LOG.md`](../Hito1/DECISION_LOG.md) · [`../Hito2/DECISION_LOG.md`](../Hito2/DECISION_LOG.md)

---

## A · Decisions carried from Hito 1 and Hito 2

| When | Decision | Evidence |
|---|---|---|
| Hito 1 | Hypothesis `recent_std_s` (same clean-lap window as `recent_mean_s`), fallback `has_history2` + training median, ablation (a)/(b)/(c), pre-registered rule: (c) below (a) and (b) overall and in R8 and R9 | `Hito1/HITO1.md` §3–4 |
| Hito 2 · D1 | Truncation check extended to 200 validation rows | 700 rows, 0 changed |
| Hito 2 · D3–D4 | Keep (c) = T1; do not switch to T3 (chosen after seeing validation) | control − T1 = +0.0179 s, CI [+0.009, +0.028] |
| Hito 2 · D5 | Next check: signed deviation | **Withdrawn in C1 below; next check now C7** |

---

## B · All trials (settings and results)

All: OLS from the course helper, preprocessing fitted on the 5,772 training rows, scored on the same 1,808 validation `pair_id`s,
MAE in seconds, seed 414. Sources: `Hito2/hito2_experiment.ipynb` §5 and §9; `Final/final_revision.ipynb` ("Fit all versions").
"Status" says whether the setting was fixed before seeing validation results.

| Trial | Setting | Status | MAE all | R8 | R9 | vs its control | Outcome |
|---|---|---|---|---|---|---|---|
| (a) | Reference, 6 inputs | planned | 0.3867 | 0.3736 | 0.4056 | — | baseline, reproduces Hito 1 and instructor |
| (b) | (a) + `has_history2` | planned | 0.3877 | 0.3716 | 0.4107 | (a) − (b) = −0.0010 | **unsuccessful alone** |
| **T1 = (c)** | (b) + `recent_std_s`, window 3, min 2 laps | planned | **0.3697** | 0.3588 | 0.3854 | (b) − T1 = **+0.0179** | **retained, nominated** |
| T2 | spread and flag with min 3 laps | exploratory (Hito 2) | 0.3711 | 0.3580 | 0.3900 | +0.0156 | not adopted: lower than T1 |
| T3 | spread without the flag | exploratory (Hito 2) | 0.3652 | 0.3576 | 0.3762 | (a) − T3 = +0.0215 | not adopted (C4) |
| T4 | (b) + signed deviation `current_lap_s − mean(previous clean laps)` | exploratory (Final) | 0.3878 | 0.3711 | 0.4116 | (b) − T4 = −0.0001 | **unsuccessful**: redundant with OLS inputs (C1) |
| T5 | (b) + \|`current_lap_s − recent_mean_s`\| | exploratory (Final) | 0.3724 | 0.3603 | 0.3897 | (b) − T5 = +0.0153 | diagnostic only (C2) |
| T6 | (b) + spread of the earlier laps in the window (3-lap windows only) | exploratory (Final) | 0.3763 | 0.3617 | 0.3972 | (b) − T6 = +0.0114 | diagnostic only (C2) |

T2–T6 were run after the planned comparison and were not used to choose the nominated model.

---

## C · Revision after the Hito 2 feedback (20 Nov)

**Priority received:** the signed deviation equals 1.5 × (`current_lap_s` − `recent_mean_s`) with three clean laps, and both
terms are already in OLS; the Hito 2 bootstrap compared T1 with the control, not T1 with T3. **Requested check:** specify what the
next variant adds, compare on the same IDs by race and history length, keep T3 separate, use a direct paired comparison for T1
vs T3. No written Demo Day feedback was available in Canvas.

**C1 · Signed deviation: change (withdrawn).** Check: identity on the data and an executed OLS (T4). Result: signed_dev = 1.5 ×
(t − mean) on 1,487 three-lap rows and 2 × (t − mean) on 173 two-lap rows (max diff 3.6e-14 s); control − T4 = −0.0001 s. It adds
no information to OLS. Removed as next check.

**C2 · What the spread adds: specified.** Check: exact identity for a 3-lap window *a, b, t*: spread² = ¾ (t − mean)² + ¼ (a − b)²
(max diff 2e-13 s² on 1,487 rows). OLS can form t − mean with its sign, but not its **size** nor the **disagreement between the
earlier laps**. Each part alone improves on the control (T5 +0.0153 s, T6 +0.0114 s, positive in both races), and the spread,
which carries both, gives +0.0179 s. Decision: no change to the candidate; the explanation in the report is corrected.

**C3 · Race × history length: added.** Check: table on the same 1,808 IDs with row-weighted pooling asserted. Result: control −
T1 = +0.0174 (R8, n 893) and +0.0328 (R9, n 594) in 3-lap windows; with 2 laps −0.0240 (R8, n 91) / +0.0102 (R9, n 82); fallback
+0.0045 (R8, n 82) / −0.0234 (R9, n 66). Decision: the claim is limited to 3-lap windows.

**C4 · T1 vs T3: direct paired comparison.** Check: per-row |err T1| − |err T3|, paired driver-race cluster bootstrap (43 clusters,
2,000 reps, seed 414). Result: +0.0045 s [+0.0009, +0.0085]; R8 +0.0012 [−0.0042, +0.0060]; R9 +0.0093 [+0.0045, +0.0144];
3 laps +0.0012 [−0.0014, +0.0038]; 2 laps +0.0047 [−0.0036, +0.0126]; fallback +0.0371 [−0.0149, +0.0933]. The Hito 2 sentence
comparing the T3 − T1 gap with the control-vs-T1 interval is corrected in the report.

**C5 · Decision: retain T1 (justified).** T3 removes only the flag. The flag's effect on short-history rows changes sign between
races (vs reference: R8 0.383 vs 0.391; R9 0.523 vs 0.402), T3's advantage is significant only in R9, and on 3-lap windows the
interval includes zero. T3 was found after seeing validation. T1 stays the planned result and the nominated candidate; T3 is
reported as exploratory.

**C6 · New next check (secondary after C7).** A frozen fallback that predicts `recent_count = 1` rows with the reference OLS and the candidate elsewhere,
compared with T1 on the same IDs by race and history length, evaluated on held-out rows rather than selected on R8–R9.


**C7 · Demo Day question: why was lap 7 of `2026-R9-23-L7` slow? (checked)**

**Feedback (Demo Day, 20 Nov, oral).** The instructor asked whether we investigated why driver 23 lost ~7 s on lap 7, the lap
behind our inspected case. We had not: we argued the cause did not matter because the model only reacts to the uneven window.

**Why we changed our view.** The cause decides whether the case is representative. If the slow lap was a track-wide event, a data
error or a lasting pace change, the case would not show "a transient loss the spread corrects".

**Checks and results** (`Final/final_revision.ipynb` R5, R9 lap snapshot):

| Check | Result |
|---|---|
| Track status, laps 4–10, all 22 drivers | Green (`1`) for everyone |
| Drivers more than 2 s slower than their previous lap on lap 7 | 1 of 22 (driver 23, +7.06 s); field median +0.11 s |
| Race conditions (public reports) | Dry, hot; VSC on lap 22 and late SC, both far from lap 7 |
| Driver 23's race (team and driver reports) | Lap-1 contact, pit on lap 1, rejoined almost a lap down, caught in a blue-flag chain |
| On-track passes reconstructed from official timing (constant speed within each lap) | Lap 6: lapped by 1 car; **lap 7: by 6 cars**; lap 12: 2; lap 13: 1; laps 8–11: none |
| Diagnostic slice, all 1,808 validation rows: driver lapped during lap *t* | Lapped (n = 83): control − candidate +0.1942 s; not lapped (n = 1,725): +0.0095 s (R8 +0.0046, R9 +0.0164) |

**Conclusion.** Lap 7 was slow because driver 23 was being lapped by six cars under blue flags. It was transient and specific to
him, so the spread's correction was right for the right reason. Blue flags are not in the track-status field, so our "clean pair"
population includes traffic laps, and about half of the pooled gain comes from the 4.6% of rows where the driver was being lapped.

**Decisions.**
- **Kept:** the candidate (T1) and the inspected case. The gain stays positive in both races on rows without lapping.
- **Corrected:** the explanation of `2026-R9-23-L12` (the row where the spread hurt most). It was not "the driver stayed slow": he
  was lapped again on laps 12 and 13.
- **Added limit:** a large part of the spread's gain comes from traffic episodes that the clean-pair filter does not exclude.
- **Next check (replaces C6 as primary):** a traffic input available at *t*, computed from the as-of timing of the other cars
  (faster cars a lap ahead within a short gap behind the driver), compared with T1 on the same IDs by race, history length and
  lapped/not lapped; declared as a trial before running and judged on held-out rows.

**Limit of the check.** Passes are reconstructed by interpolating each car's progress within a lap (no positions or GPS). With six
cars passing in one lap the result is clear; single passes near the start or end of a lap could be assigned to the adjacent lap.
The lapped/not-lapped slice uses timing of other cars around lap *t*, so it is a retrospective diagnostic, not a model input.

---

## D · Contributions

| Member | Contribution (final revision) | Decision or check they can explain |
|---|---|---|
| Andy Ignacio Jesús Villarroel González | **R1** signed deviation as T4 and its identity with `current_lap_s − recent_mean_s` (C1); **R2** decomposition spread² = ¾ (t − mean)² + ¼ (a − b)² and the part-by-part trials T5/T6 (C2); **R3** table by race × history length (C3); independent re-run of `Final/final_revision.ipynb` before submission | Withdraw the signed deviation (C1): it is an exact multiple of t − mean, already in OLS; control − T4 = −0.0001 s. Limit the claim to 3-lap windows (C3): the only history length with a positive gain in both races (+0.0174 / +0.0328 s) |
| Agustín Eduardo Reyes Pereira | **R4** paired T1-vs-T3 comparison with driver-race cluster bootstrap (C4–C5); **R5** Demo Day question: field check on lap 7, reconstruction of on-track passes, race reports, lapped / not-lapped slice (C7); `FINAL_REPORT.md`, README and PDF | Keep T1 although T3 is +0.0045 s better (C5): the advantage is not consistent across races (R8 interval includes zero) and T3 was chosen after seeing validation. Lap 7 was slow because driver 23 was lapped by 6 cars (C7); half of the gain comes from traffic laps, so the next check is a traffic input |

Both members reviewed the final report, the T1 decision (C5) and the traffic finding (C7).

---

## E · AI use and verification (final revision)

We used Claude (Anthropic) as a coding and writing assistant, as in Hito 2. It did not choose what to check, the decision to
keep T1 or the next check; those followed the Hito 2 feedback, the Demo Day question and the rule fixed in Hito 1.

| Use (member) | Representative prompt (paraphrased) | Accepted / changed | How we verified it |
|---|---|---|---|
| R1 identity and T4 (Andy) | "Show that the signed deviation is a multiple of current_lap_s − recent_mean_s for 2- and 3-lap windows and add it to the OLS as a trial" | Accepted the code after review. The factor per window (1.5 for 3 laps, 2 for 2 laps) was derived by hand and asserted, not taken from the assistant | Notebook assert: max diff 3.6e-14 s (1,487 rows) and 2.8e-14 s (173 rows); control − T4 = −0.0001 s |
| R2 decomposition and T5/T6 (Andy) | "Decompose the variance of three laps a, b, t into a term in t − mean and a term in a − b, and fit each part alone" | Accepted the identity after checking it on paper; T5/T6 labelled exploratory (our decision) | Assert: identity holds to 2e-13 s² on 1,487 rows; T5/T6 MAEs re-read from the notebook table (+0.0153 / +0.0114 s) |
| R3 race × history table (Andy) | "Group control and candidate MAE by race and recent_count on the same pair_ids" | Accepted; pooling check added by us | Assert: the planned predictions are identical to Hito 2 (≤ 1.4e-14 s); the row-weighted sum of groups equals the pooled control − candidate (0.0179 s) |
| R4 paired bootstrap (Agustín) | "Paired cluster bootstrap of \|err T1\| − \|err T3\| resampling driver-races, overall, by race and by history length" | Accepted after review; seed 414 and 43 clusters kept from Hito 2 | Assert: bootstrap mean equals MAE(T1) − MAE(T3); the Hito 2 interval for control − T1 is reproduced exactly ([+0.0089, +0.0281]) |
| R5 pass reconstruction and race reports (Agustín) | "Using lap end times of all cars, find which cars lapped driver 23 on each lap"; "find race reports explaining Albon's slow lap 7 at Silverstone 2026" | Accepted the code; interpolation assumption and its limit written by us. Sources kept only after reading them | Checked in the R9 lap CSV that lap 7 of #23 is 105.128 s (laps 5–6: 96.299, 98.070; lap 8: 97.882) with green status; assert 6 lapping cars on lap 7; opened the Williams and Albon reports and confirmed they mention the lap-1 contact, rejoining almost a lap down and the "blue flag chain" |
| Drafting and editing the report (both) | Summaries of notebook outputs into `FINAL_REPORT.md` and this log | Rewritten and shortened by us | Every number checked against the executed notebook outputs; the notebook was re-run from scratch before submission and all asserts passed with identical MAEs and intervals |
