# NextLap capstone · Hito 1

IIT414W Artificial Intelligence Workshop · Team: Lab 8 · Members: Andy Ignacio Jesús Villarroel González, Agustín Eduardo Reyes Pereira

Goal: at the end of lap *t*, predict the same driver's next lap time (`next_lap_s`, seconds).

## Contents

| File | What it is |
|---|---|
| [`HITO1.md`](HITO1.md) | Problem framing, reproduced reference, hypothesis and experiment plan |
| [`DECISION_LOG.md`](DECISION_LOG.md) | Decisions, feedback, AI use and who did what |
| [`reproduce_reference.ipynb`](reproduce_reference.ipynb) | Notebook that reproduces median, persistence and OLS on the 1,808 validation rows |
| `outputs/` | Generated metric table, validation predictions and run record |
| `requirements.txt` | Pinned package versions |

## Data

The data are not copied here. They come from the course release
`IIT414W_W09_Fri_Capstone_StudentPack_v2.zip` → folder `base_NextLap/`.
SHA-256 of the files used are recorded in `outputs/reference_run.json`.

Extract the ZIP at the repository root, so that the folder is `W09_StudentPack_v2/base_NextLap`
(it is git-ignored). Otherwise set:

```bash
export NEXTLAP_DATA=/path/to/base_NextLap
```

## How to run

From this `Hito1/` folder:

```bash
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
jupyter nbconvert --to notebook --execute --inplace reproduce_reference.ipynb
```

Or open `reproduce_reference.ipynb` in Jupyter or VS Code and use **Run All**.

The notebook writes `outputs/reference_metrics.csv` (model, population, n, MAE in seconds),
`outputs/validation_reference_predictions.csv` (per `pair_id`) and `outputs/reference_run.json`.
No internet, no test data, `RANDOM_SEED = 414`, runs in a few seconds.

The notebook is adapted from the course script `probar_referencia_v1.py` and uses the course
helper `nextlap_reference_v1.py`, both with attribution.
