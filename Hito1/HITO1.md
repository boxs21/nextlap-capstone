# Hito 1 · NextLap · Problem framing and experiment plan

Team: Lab 8 · Members: Andy Ignacio Jesús Villarroel González, Agustín Eduardo Reyes Pereira · Commit: `453863d` (notebook and outputs) · Date: 12 Nov 2026

---

## 1. Problem and decision value

| Item | Our definition |
|---|---|
| **What we predict** | The duration of a driver's **next lap** (lap *t+1*) |
| **Target and unit** | `next_lap_s`, in seconds |
| **When we predict** | At the moment lap *t* ends (`end_time_s` of lap *t*). Only information available at that moment can be used |
| **Metric** | Mean absolute error (MAE) in seconds |

**Who could use it and for what.** A team's race engineer could use it to see whether the
driver's pace on the next lap is likely to stay the same or change. For example, it could
help check whether a rival behind is likely to close the gap on the next lap, or whether the
driver's pace is starting to drop. It is one input to a decision, not the decision itself.

**What it cannot tell us.** A lap-time prediction alone does not say *when to pit* or *how
many positions a strategy will gain*.

**Limit of the clean-pair setting.** Our rows are only "clean pairs": both lap *t* and lap
*t+1* are normal green-flag laps without pit stops. That filter uses information about
lap *t+1*, which is not known in advance. So our MAE describes **normal racing laps only**. It
does not describe pit laps, safety-car laps, or a live system that cannot know beforehand
which laps will be clean.

---

## 2. Reproduced reference

| Item | Value |
|---|---|
| Data | Course release v1 (inside student pack v2), real 2026 timing, checksums verified |
| Train | Rounds 1–7, **5,772** clean pairs |
| Validation | Rounds 8–9, **1,808** clean pairs (round 8: 1,066 · round 9: 742) |
| Inputs | `current_lap_s`, `recent_mean_s`, `current_lap_number`, `current_tyre_life`, `current_compound`, `current_team` |
| Preprocessing | Median imputation + scaling + one-hot, **fitted on training rows only** (sklearn Pipeline) |
| Seed | 414 |
| Notebook | `reproduce_reference.ipynb` (Run All) |

**Our own run (validation MAE, seconds):**

| Model | All (n=1808) | Round 8 (n=1066) | Round 9 (n=742) |
|---|---|---|---|
| Training median | 11.376 | 11.716 | 10.888 |
| Persistence (next lap = current lap) | 0.4093 | 0.4005 | 0.4219 |
| **OLS reference** | **0.3867** | **0.3736** | **0.4056** |

All three models are scored on the **same 1,808 rows** (same `pair_id`s; their fingerprint is
in `outputs/reference_run.json`). Our numbers match the instructor's supplied outputs, taken
from the course pack (`StudentPack_v2` → `salidas_referencia_INSTRUCTOR/reference_metrics.csv`
and `validation_reference_predictions.csv`; not copied here, checksums recorded). The notebook
(section 5) merges both by `pair_id` and finds the same 1,808 rows and counts, a max
prediction difference of 1.4e-14 s and a max MAE difference of 4.4e-16 s (stored under
`instructor_check` in `outputs/reference_run.json`). The instructor files are only used for
this check, never to produce our numbers.

**One comparison explained.** A single median for every lap is very poor (11.4 s) because
lap times differ a lot between circuits. Simply repeating the current lap time already
brings the error down to 0.41 s, so most of the information is in the current pace. OLS
improves on that by only **0.023 s**. Any new feature therefore has to find a small but real
signal on top of "the driver will lap like they just did".

---

## 3. Primary hypothesis: pace consistency

**Idea in one sentence.** When a driver's last few clean laps were very similar, the next lap
is easier to predict. When they varied a lot, it is harder. The recent **average** used by
the reference cannot see this difference, but the **spread** can.

| Item | Definition |
|---|---|
| **New feature** | `recent_std_s`: standard deviation (ddof=1) of the driver's **clean** lap times in laps *t−2*, *t−1*, *t* |
| **Clean lap** | Same rule the reference uses: not lap 1, no pit in/out, not deleted, accurate timing, green flag. Other laps are left out of the window |
| **Window** | Exactly the same 3-lap window as `recent_mean_s`, inside the same driver, race and tyre stint |
| **Source** | The released timing files (`data/race_2026_r0*_laps_v2.csv`). No external source is needed, so no fallback source is required |
| **Available at *t*?** | Yes. It only uses laps that have already finished. The last observation used is lap *t* itself |
| **How it is built** | On the **original lap timeline**, before selecting clean pairs, so that the future-based filter cannot change which past laps are used |
| **Short history** | With fewer than 2 clean laps the spread cannot be computed. This affects **456 of 5,772** training rows and **148 of 1,808** validation rows (about 8%). Rule: add a flag `has_history2` (1 = at least 2 clean laps) and fill the missing value with the **training median**. **No rows are removed** |

**Why this direction.** It uses only the released timing data, so it needs no external source
and no fallback source. It is also simple to check for leakage, because it uses the same
window as a column the instructor already provides (`recent_mean_s`). The short-history
counts above come from the released `recent_count` column (rows with `recent_count = 1`).

---

## 4. Experiment plan (to be executed in Hito 2)

**What stays fixed in every comparison:** target `next_lap_s` · the same 1,808 validation
rows (same `pair_id`s) · train rounds 1–7 / validation rounds 8–9 · OLS · preprocessing
fitted on training rows only · MAE in seconds · seed 414.

**Three versions of the same model:**

| Version | Inputs | What it tells us |
|---|---|---|
| (a) Reference | 6 reference inputs | The baseline to beat |
| (b) + flag | (a) + `has_history2` | Effect of only knowing "short history or not" |
| (c) + flag + spread | (b) + `recent_std_s` | **(c) vs (b) = what the spread adds** (ablation) |

We need (b) because the flag by itself is also new information (it says how many clean laps
there were). Without (b) we could not tell whether a gain came from the spread or from the flag.

**Supports the hypothesis if** (c) has a lower MAE than both (a) and (b), overall **and** in
each validation race separately (round 8 and round 9).

**Contradicts the hypothesis if** (c) is not lower than (b) overall or in either race, or if
any feature value turns out to depend on lap *t+1*. A negative result will be reported as
such.

**How validation and test are used.**
- Validation (rounds 8–9) is used **once** for the comparison above, with the variant fixed now. If we try anything else later, every attempt goes into the log.
- The instructor's final test data are **never** used by us to choose features or settings. We have had no prior access to them.

**Checks written before building** (run before looking at any validation MAE of the extension):

| Check | What would make it fail |
|---|---|
| Rebuilding `recent_mean_s` from the raw timeline with our code gives exactly the released column | Any difference → our window is not the reference window |
| Removing every lap after *t* does not change `recent_std_s` (random sample of training rows) | Any change → the feature uses future laps (leakage) |
| The last observation used by the feature ends at or before `end_time_s` of *t* | Any later timestamp |
| Every train/validation row gets a feature value or the declared fallback | Row count different from 5,772 / 1,808, or a changed `pair_id` set |

**Robustness checks planned for Hito 2:** MAE by validation race with counts · MAE for
short-history vs normal rows · errors by context (compound, tyre age, stint phase) · SHAP for
(c) · OLS coefficients of the new inputs.

---

## 5. Traceability and next steps

| Member | Main contribution | Decision they can explain |
|---|---|---|
| Andy Ignacio Jesús Villarroel González | Reproduced reference notebook (`reproduce_reference.ipynb`), generated `outputs/`, `requirements.txt` and run record with checksums; made the Hito 1 commit | Why all three models are scored on the same 1,808 `pair_id`s and why preprocessing is fitted on training rows only |
| Agustín Eduardo Reyes Pereira | Problem framing (target, timing, clean-pair limit), pace-consistency hypothesis and the (a)/(b)/(c) ablation plan with its leakage checks | Why `recent_std_s` uses the same clean-lap window as `recent_mean_s`, and why the `has_history2` flag needs its own version (b) |

- **AI use:** see `DECISION_LOG.md`.
- **Revision after feedback:** the in-class version said "variability of the last three observed laps" and "MAE lower than the reference". After feedback we (1) defined the window as the same clean-lap window used by `recent_mean_s`, (2) added version (b) so the flag and the spread are not confused, and (3) required the gain in each validation race, not only overall.
- **Before Hito 2 (19 Nov):** Andy: run comparison (a)/(b)/(c) and per-race table · Agustín: contextual errors and SHAP · both: limits and report.
- **Known blocker:** none so far. To confirm with Ángel (12–13 Nov): whether the 2-lap spread (only two values) should also count as short history.
