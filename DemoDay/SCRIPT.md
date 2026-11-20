# NextLap · Demo Day · Script (Group 8)

**Friday 20 Nov · 13:05–13:12 talk · 13:12–13:17 questions · Lab 033 RESB**
Andy Villarroel + Agustín Reyes · 7-min talk slot (we use **~5 min**), 5 min of questions. Both answer.

Split: **Andy** opens and closes (slides 1, 2, 6). **Agustín** presents the results (slides 3, 4, 5).
Pace: ~130 words per minute, ~5 min total. If there is time left, stretch the case on slide 4 (limit 7:00).

> Spanish reference translation: `GUION.md`.

---

## How to use the deck

**Trick: the slide is the script.** Text on each slide is in the same order you say it. Read top to bottom, left to right, pointing at what you say. If you get lost: `CARDS.pdf` (one page per person, in hand or on your phone).

- Open `NextLap_DemoDay.html` in **Chrome** → `F` fullscreen.
- `→` / space / click = next · `←` = back · `1`–`8` = jump to slide.
- `N` = shows the script at the bottom (practice only, **not** with the projector mirrored).
- `T` = timer (`mm:ss / 05:00`, turns red if you are behind on that slide). Starts by itself on slide 2.
- Slides 7 and 8 = **backup** for questions (not presented).
- If the HTML fails: `NextLap_DemoDay_backup.pdf` (same content, static).
- Works **offline** (fonts and Three.js are inside the file).

---

## 01 · Title — ANDY · 0:00–0:35

> Hi, we're Andy Villarroel and Agustín Reyes, group 8, and this is NextLap. The question: when a driver finishes a lap, **how long will the next one take?** We predict that time in seconds at the end of lap *t*, using only what is already known. It could help a race engineer anticipate whether pace is dropping or recovering. Everything here is validation on past races, not a live test.

---

## 02 · What we changed — ANDY · 0:35–1:20

> The base model uses the average of the last laps, but an average cannot tell a **steady** driver from an **erratic** one. So we added the **spread**: how much their last up-to-three clean laps vary, computed up to lap *t*, never after. With fewer than two, we use a flag and the training median. We train on races 1 to 7 and validate on Austria and Great Britain: **the same 1,808 laps for every model**. Before looking at results we passed four leakage checks. The control is the same model **without** the spread. Agustín.

---

## 03 · Result — AGUSTÍN · 1:20–2:25

> We measure **MAE**: on average, how many seconds we miss by. Lower is better. The course reference: **0.3867**. Our candidate, with the spread: **0.3697**. The control, same but without the spread: **0.3877**; the flag alone does not help. Control minus candidate: **plus 0.018 seconds** in favor of the candidate, about 4.6 percent less error. A bootstrap by driver-race gives an interval from **0.009 to 0.028**: it excludes zero. And it meets the rule we set in Milestone 1 before running anything. To be honest: it is a real gain, but **small** next to a typical error of 0.37.

---

## 04 · Where it struggles — AGUSTÍN · 2:25–3:20

> It improves in both races: Austria **0.013**, Great Britain **0.025**. But almost all the gain comes from **unstable windows**: there it drops **0.085**. With short history, it gets slightly worse. Real case: driver 23 at Silverstone was lapping around 96 seconds and then does a **105**. The control believes it and predicts 102.1: off by **4.2**. The candidate sees the unstable window and predicts **99.1**; the real lap was 97.9, an error of **1.24**. The limit: spread **does not know the direction**; if the slow pace continues, it over-corrects.

---

## 05 · SHAP — AGUSTÍN · 3:20–4:20

> What moved that prediction? With **SHAP** on the trained candidate: we start at **86.73**, the training average. The recent average adds **8.2** and the current lap **6.7**, because Silverstone is a long lap. The spread **subtracts 2.47**: with an unstable window, the model expects a faster lap. Base plus contributions gives **99.121**, exactly the prediction. This explains the prediction, not the error, which is separate: 1.24. Globally, across the 1,808 laps, spread is the **third input**. But the ablation proves the error gain, not SHAP. Andy.

---

## 06 · Decision — ANDY · 4:20–5:00

> Decision: **we keep the spread.** Same model, same laps, same control: it improves in both races and the interval excludes zero. It applies to linear regression, clean laps, Austria and Great Britain; not to short history, pits or other circuits. A no-flag variant scored slightly better, but we picked it after seeing validation, so we did not adopt it. Next check: a **signed deviation**, which knows whether the lap was slower or faster. **Thank you, questions?**

---

## Likely questions (5 min · both answer)

Rule: short answer → number → limit. If you don't know: "We didn't test that; what we did measure is…".

| Question | Who | Short answer |
|---|---|---|
| How do you know you don't use future information? | **Andy** | Four checks fixed in Milestone 1, run **before** seeing MAE. We deleted every lap after *t* in 700 rows (500 train + 200 validation, seed 414): 0 changes. None of the 7,580 rows uses a lap ending after *t*. Slide 7 (backup A). |
| Why did you extend the check to validation? | **Andy** | Validation rows are the ones we score; a leak there would bias the comparison directly. Limit: it's a 700-row sample; running it on all 7,580 is cheap and the next step. |
| What if there is not enough history? | **Andy** | With < 2 clean laps: `has_history2 = 0` and the train median (0.258 s), fit on train only. That's 148 validation rows (8 %). No gain there: −0.008 s, and in R9 −0.023 (n = 66). |
| What is the *clean pair* filter? Isn't it cheating? | **Andy** | It selects pairs where *t* and *t+1* are clean laps. It uses *t+1* info, so it **only defines the evaluated population** and is never an input. That's also why the result doesn't apply to pits or safety car. |
| Why not keep T3 if it was better (0.3652)? | **Agustín** | We tried it **after** seeing validation; choosing it would be tuning on validation. The T3−T1 gap is 0.0045 s, small next to the interval [0.009; 0.028]. It stays as a next check. Slide 8. |
| Does 0.018 s matter at all? | **Agustín** | Small on average (≈ 4.6 %), but 0.085 s in unstable windows, and the L7 case drops from 4.2 to 1.2 s. We don't promise operational performance: it's retrospective validation on 2 races. |
| Why a driver-race bootstrap? | **Agustín** | Laps of the same driver in a race are correlated; resampling single rows would give a too-narrow interval. 43 clusters, 2,000 resamples, seed 414. |
| How did you pick the case you show? | **Agustín** | Rule fixed before looking: the validation row where spread moved the prediction most, max \|pred(c) − pred(b)\|. We didn't pick it because it looks best. |
| What was the worst error? | **Agustín** | `2026-R9-23-L6`, 8.25 s: the slow lap is the **target** and nothing visible at L6 signals it (σ = 0.98). No history feature can fix that. |
| What is the SHAP "base value"? | **Agustín** | The mean prediction over the background of 5,772 training rows: 86.734 s. LinearExplainer, independent masker. Base + contributions = prediction (max error 1.4e-14 s over 1,808 rows). |
| Does SHAP show spread improves the error? | **Agustín** | No. SHAP explains the model output; the error gain is shown by the ablation (control vs candidate, same rows). |
| Why is Great Britain worse? | both | Every model misses more there (control 0.41 vs 0.37). It's also where spread helps most (+0.025). With 2 races we can't separate circuit from conditions. |
| Would it work with another model (trees, boosting)? | both | We didn't test it. The claim is for OLS only; a non-linear model might capture instability differently. |
| Why a 3-lap window? | **Andy** | Same window the reference uses for `recent_mean_s`, so they're comparable. With a 3-lap minimum (T2) the gain was smaller (+0.0156). |
| How did you use AI? | both | Claude as a coding assistant: Andy for `history_features()` and the asserts; Agustín for bootstrap and SHAP. The hypothesis, decision rule and reported model were ours. We verified with the checks: rebuilding `recent_mean_s`, row-by-row reproduction of the reference, SHAP reconstruction. |
| What did each of you do? | each | **Andy**: feature on the original timeline, the 4 leakage checks, reproducibility (README, hashes, Run All). **Agustín**: a/b/c ablation, T2/T3, per-race errors and slices, inspected case, bootstrap, SHAP. |

**The decision each of you defends (they will ask):**
- **Andy** → extending the truncation check to 200 validation rows. Result: 700 rows, 0 changes. Limit: it's a sample.
- **Agustín** → keeping (c) and not switching to T3. Result: +0.018 s, CI [+0.009; +0.028]. Limit: the gain is concentrated in high spread and gets worse with short history in R9.

---

## Checklist for tomorrow

- [ ] Rehearse once with the timer (`T`). Goal: finish slide 6 near 5:00 (max 7:00).
- [ ] Copy `NextLap_DemoDay.html` **and** `NextLap_DemoDay_backup.pdf` to a USB stick + send them to Agustín.
- [ ] Open the HTML on the lab computer early (13:00). If slow: the PDF.
- [ ] Have the repo open at notebook §5 / §8 in case they ask to see code.
- [ ] Write down the jury's feedback: you need it for section 4 of the final report (Sun 22, 23:59).
