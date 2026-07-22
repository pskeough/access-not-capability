# F21 — Abstention discrimination (Youden's J) across all three local sizes

**Date:** 2026-07-09
**Purpose:** Result 4's beneficial direction (retrieval sharpens answerable/unanswerable
discrimination on the homogeneous private corpus) is stated in the paper for Llama-8B only.
This records that it holds across all three local sizes, generalizing the benefit half of
the abstention finding.

## Source
- Recompute: `experiments/v5_raft_cot/_analysis/master_analysis/scripts/05_answer_chars.py`
- Output: `.../stats/05_answer_chars.json` -> `calibration` block
- Feeds: STATISTICAL_REPORT.md §9, fig5_calibration_youdenJ

## Metric
Youden's J = correct-abstention-rate(OOC) − false-refusal-rate(answerable), private corpus,
ooc_n = 52, answerable_n = 293. Classifier: the single-sentence refusal regex of
05_answer_chars.py (SIMPLER than the F12 3-class rule set; the direction is robust to this
choice, magnitudes may shift under F12's classifier).

## Numbers (verified read 2026-07-09)
| model | base | rag | lora | lora_rag |
|---|---|---|---|---|
| Llama-8B   | −0.303 | **+0.887** | −0.425 | **+0.878** |
| Phi-4-mini | −0.194 | **+0.885** | −0.014 | **+0.907** |
| MiniCPM-1B | −0.347 | **+0.683** | −0.375 | **+0.701** |

Every retrieval config (rag, lora_rag): J ∈ [+0.68, +0.91], strongly positive.
Every non-retrieval config (base, lora): J ∈ [−0.43, −0.01], negative or ~zero.
The negative→positive transition on adding retrieval holds 3/3 models.

## What this licenses (and does NOT)
- **Licensed:** the beneficial direction of Result 4 — on the homogeneous corpus, retrieval
  converts a model that cannot separate answerable from unanswerable (negative J) into one
  that can (strongly positive J) — generalizes across the 1B, 3.8B, and 8B local sizes, not
  just the 8B reported in the body.
- **NOT licensed:** generalizing the SIGN FLIP (the QASPER collapse) across sizes; no
  phi/minicpm QASPER or COVID abstention data exists, so the harmful direction remains
  demonstrated on Llama-8B only. The magnitudes here use a simpler classifier than F12.
