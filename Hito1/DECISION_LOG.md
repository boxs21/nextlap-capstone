# Decision log · NextLap · Hito 1

Team: Lab 8 · Members: Andy Ignacio Jesús Villarroel González, Agustín Eduardo Reyes Pereira

AI tool used: Claude

---

## A. Andy: reference reproduction and reproducibility

### A1 · Reproducing the reference
- **Prompt:** "Adapt the course script `probar_referencia_v1.py` into a notebook that trains the training median, persistence and OLS references on train rounds 1–7 and scores them on validation rounds 8–9, overall and per round."
- **Output:** Notebook with loading, three models, MAE table by population.
- **Decision:** Accepted with changes.
- **What changed:** Added asserts (5,772 / 1,808 rows, rounds, unique and disjoint `pair_id`s, no `next_*` inputs). Kept the course helper `nextlap_reference_v1.py` unmodified, with attribution.

### A2 · Same rows for every model
- **Prompt:** "How do I make sure all models are compared on exactly the same validation rows, and prove it?"
- **Output:** Suggested comparing row counts.
- **Decision:** Modified.
- **What changed:** Row counts alone were not enough for us. We store a SHA-256 of the sorted validation `pair_id`s in `outputs/reference_run.json` and save per-`pair_id` predictions.

### A3 · Saved outputs and run record
- **Prompt:** "Write the outputs of the notebook (metrics table, predictions, run record) so the result can be checked without rerunning."
- **Output:** CSV and JSON writing code.
- **Decision:** Accepted with changes.
- **What changed:** Added `test_used: false`, the seed (414), package versions and data-file checksums to the JSON.

### A4 · README and pinned requirements
- **Prompt:** "Write a README with how to run and a requirements file for the notebook. Data must not be copied into the repo."
- **Output:** README draft and package list.
- **Decision:** Modified.
- **What changed:** Data path set through `NEXTLAP_DATA` or the git-ignored `W09_StudentPack_v2/`. Versions pinned to those we actually ran; comment added that results are identical to the course-verified versions.

---

## B. Agustín: framing, hypothesis and experiment plan

### B1 · Problem framing
- **Prompt:** "Help me define the prediction task for the next lap: target, unit, when we predict, metric, and what the model cannot tell a race engineer."
- **Output:** Table of target/time/metric and use cases, including pit-strategy claims.
- **Decision:** Modified.
- **What changed:** Removed claims that the model decides when to pit. Added the clean-pair limit: the filter uses lap *t+1*, so MAE only describes normal green-flag laps.

### B2 · Hypothesis definition (after feedback)
- **Prompt:** "Our hypothesis is 'variability of the last three laps improves MAE'. Make it testable and leak-free."
- **Output:** Standard deviation of last 3 lap times as new feature `recent_std_s`.
- **Decision:** Modified.
- **What changed:** Window defined as the same clean-lap window as `recent_mean_s` (ddof=1), built on the original lap timeline before choosing clean pairs.

### B3 · Short-history handling
- **Prompt:** "The spread needs at least 2 laps. What do we do with rows that have fewer?"
- **Output:** Options: drop rows, impute, or add an indicator.
- **Decision:** Modified.
- **What changed:** Dropping rejected (would change the 1,808 validation rows). We fill with the training median and add `has_history2`. Counts (456 train, 148 validation) taken from the released `recent_count` column, not from AI.

### B4 · Ablation and success criteria
- **Prompt:** "Design the comparison so a gain can be attributed to the spread and not to something else."
- **Output:** Reference vs reference + spread.
- **Decision:** Modified.
- **What changed:** Added version (b) = reference + flag, so (c) vs (b) isolates the spread. Success now requires a lower MAE in each validation race, not only overall. Added four leakage and coverage checks to run before looking at validation MAE.

---

## C. Final check against the rubric (both)

- **Prompt:** "Check the repository (HITO1.md, README.md, DECISION_LOG.md, notebook, outputs) against this rubric: problem and decision value 20, reproduced reference 25, feature hypothesis 20, experiment plan 25, reproducibility and responsibility 10. List what is missing."
- **Output:** Criteria 1–4 judged complete on the document evidence. Gaps listed: (1) HITO1.md claims our numbers match the instructor's outputs (max difference 4e-16 s) but no file or notebook cell in the repo shows that comparison; (2) `<commit hash>` placeholder in HITO1.md.
- **Decision:** Accepted. Both gaps are ours to fix before submitting.
- **What changed:** (1) Added section 5 to the notebook. It reads the instructor's outputs from the course pack (`salidas_referencia_INSTRUCTOR/reference_metrics.csv` and `validation_reference_predictions.csv`), merges them with ours by `pair_id` and asserts the same 1,808 rows, the same counts and differences below 1e-9 s. Result: max prediction difference 1.4e-14 s, max MAE difference 4.4e-16 s; stored with the files' checksums under `instructor_check` in `outputs/reference_run.json`. HITO1.md now cites these two numbers instead of a single unsupported "4e-16 s". (2) Filled the commit hash in HITO1.md with the commit that contains the final notebook and outputs.
- **How we checked:** Re-ran the whole notebook (`nbconvert --execute`); `reference_metrics.csv` did not change.
