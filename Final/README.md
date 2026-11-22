# NextLap capstone · Final report and repository

IIT414W · Team Lab 8 (G08) · Andy Ignacio Jesús Villarroel González, Agustín Eduardo Reyes Pereira

| | |
|---|---|
| **Repository** | https://github.com/boxs21/nextlap-capstone |
| **Report** | [`FINAL_REPORT.md`](FINAL_REPORT.md) (submitted as PDF) |
| **Decision log** | [`DECISION_LOG.md`](DECISION_LOG.md) |
| **Planned experiment + SHAP** | [`../Hito2/hito2_experiment.ipynb`](../Hito2/hito2_experiment.ipynb) → [`../Hito2/outputs/validation_predictions.csv`](../Hito2/outputs/validation_predictions.csv) |
| **Revision after feedback** | [`final_revision.ipynb`](final_revision.ipynb) → [`outputs/final_validation_predictions.csv`](outputs/final_validation_predictions.csv) |
| **Earlier milestones** | [`../Hito1/`](../Hito1/), [`../Hito2/`](../Hito2/), [`../DemoDay/`](../DemoDay/) |

## Data (inputs)

Course release `IIT414W_W09_Fri_Capstone_StudentPack_v2.zip` (Canvas), SHA-256
`2ad128c6b3af4cf44b80fb05a750b53431f4c717e7f9a5b53d867bf29db2747a`. Not copied into the repository. Files used from `base_NextLap/`:

| File | SHA-256 |
|---|---|
| `train_clean_v1.csv` (5,772 pairs, R1–R7) | `43544f38cc9ef32b44886e6751e03522601265160086fe3a4956f04b87bd5c13` |
| `validation_clean_v1.csv` (1,808 pairs, R8–R9) | `b19efb6a908af12a5d2c6bdde29aa40984295e62ce0f187a201b579278db7ee8` |
| `nextlap_reference_v1.py` (course helper, unmodified) | `f6c58925cd4a477bf3e1a5a2b3968e67df3a1cffabecee9e89701c3ac2d5f994` |
| `data/race_2026_r01…r09_laps_v2.csv` | verified against `data/source_manifest_v2.json` by the helper |
| `data/source_manifest_v2.json` | `04bfe34c4d9dff707f842dd4b96049525b66dcd5179cb120d9c050de27c1c0e8` |

```bash
unzip IIT414W_W09_Fri_Capstone_StudentPack_v2.zip -d W09_StudentPack_v2   # at the repository root (git-ignored)
# or: export NEXTLAP_DATA=/path/to/base_NextLap
```

## Environment and commands

Python 3.10.12, versions in [`requirements.txt`](requirements.txt). From the **repository root**:

```bash
python -m venv .venv && source .venv/bin/activate      # Windows: .venv\Scripts\activate
pip install -r Final/requirements.txt
jupyter nbconvert --to notebook --execute --inplace Hito2/hito2_experiment.ipynb
jupyter nbconvert --to notebook --execute --inplace Final/final_revision.ipynb
```

Each notebook stops with an assertion if a check fails (row counts, split, unique and common `pair_id`s, leakage checks,
reproduction of earlier predictions, pooled arithmetic, SHAP reconstruction, algebraic identities). Seed 414. No test data. The
Final notebook reads `Hito2/outputs/validation_predictions.csv` to assert that its planned predictions are identical.

## Where each report section comes from

| Report section | Notebook | Output |
|---|---|---|
| §1 split, fitting, availability checks, IDs | Hito2 §1–§4 | run record (Hito2 §10) |
| §2 comparison, difference, bootstrap | Hito2 §4–§5; Final R4 | `Hito2/outputs/validation_predictions.csv` |
| §3 race table, slices, case | Hito2 §5–§7 | same file |
| §3 race × history length | Final R3 | `Final/outputs/final_validation_predictions.csv` |
| §4 local and global SHAP | Hito2 §8 | cell outputs (tables, bar plot, beeswarm) |
| §6 revision R1–R4 | Final R1–R4 | `Final/outputs/final_validation_predictions.csv`, run record (last cell) |
| §3 and §6 Demo Day check (lap 7, traffic slice) | Final R5 | `Final/outputs/lapped_during_t.csv`, run record (last cell) |
