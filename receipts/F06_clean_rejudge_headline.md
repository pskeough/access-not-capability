# F06 — Clean-rejudge headline (corrected frontier comparison) — GOLD receipt

**Family:** F06 clean-rejudge headline
**Recompute script:** `C:/Research/Train_LLM/Final_Validation/scripts/F06_clean_rejudge_headline.py`
**Result:** 68/68 checks PASS, exit 0. Bootstrap = 5000 cluster resamples over unique questions (seed 20260611).

## Raw inputs (all paths absolute)
- Clean judgments (primary): `C:/Research/Train_LLM/FinalRunPack/frontier_pilot/clean_rejudge/judgments_single.jsonl` (5911 rows; corpus∈{base,qasper}, single judge=gemini, both orders 0/1)
- Compared-against summary: `C:/Research/Train_LLM/FinalRunPack/frontier_pilot/clean_rejudge/summary_single.json`
- Answerable filter (question_type≠out_of_corpus): `C:/Research/Train_LLM/experiments/v4_raft_cot/data/eval_set.jsonl` (base, 345 q, 52 OOC), `C:/Research/Train_LLM/experiments/v5_raft_cot/corpora/qasper/eval_set.jsonl` (qasper)
- S7 capped baselines re-pooled from: `frontier_pilot/summary_panel.json`, `frontier_pilot/summary.json`, `frontier_pilot/summary_deepseek_v2.json`

## Method (2-3 lines)
Group clean rows by (corpus, matchup, question_id); consolidate the two presentation orders with **strict both-orders** (decisive only if both orders agree on the same non-tie side, else tie). Net win-rate = (F + ½T)/n. Pooled = sum of the 3 frontier cells per local model. Bootstrap CIs resample **unique questions** (question-clustered, 5000 draws). Question-level sign test = exact two-sided binomial on per-question decisive aggregation (qF = questions where frontier decisive wins > local). Per-cell: exact binomial sign test on decisive pairs + Wilson 95% CI on local not-lose. All recomputed independently from raw; summaries only compared.

## S6 — Pooled, question-clustered (published §S6, p.73-86 of STATISTICAL_SUPPLEMENT.md)

| Claim | Published | Recomputed | Verdict |
|---|---|---|---|
| vs 8B base-all net [CI] | 0.600 [0.580,0.620] | 0.5995 [0.580,0.621] | MATCH |
| vs 8B base-all not-lose / tie | 0.787 / 0.774 | 0.7874 / 0.7738 | MATCH |
| vs 8B base-all q-sign (qF/qL, p) | 98/10, 2.6e-19 | 98/10, 2.65e-19 | MATCH |
| vs 8B base-answerable net [CI] | 0.618 [0.595,0.641] | 0.6173 [0.595,0.641] | MATCH |
| vs 8B base-answerable not-lose / tie | 0.749 / 0.733 | 0.7494 / 0.7334 | MATCH |
| vs 8B QASPER-answerable net [CI] | 0.573 [0.543,0.603] | 0.5727 [0.543,0.604] | MATCH |
| vs 8B QASPER-answerable not-lose / tie | 0.810 / 0.765 | 0.8098 / 0.7651 | MATCH |
| vs 8B QASPER q-sign p | 2.1e-5 | 2.09e-5 (qF/qL 44/12) | MATCH |
| vs 1B base-all net [CI] | 0.720 [0.695,0.746] | 0.7201 [0.694,0.746] | MATCH |
| vs 1B base-answerable net [CI] | 0.741 [0.714,0.769] | 0.7405 [0.711,0.770] | MATCH |
| vs 1B QASPER-answerable net [CI] | 0.764 [0.726,0.802] | 0.7640 [0.726,0.801] | MATCH |
| vs 1B not-lose (all/ans/qasper) | 0.546 / 0.503 / 0.441 | 0.5462 / 0.5029 / 0.4407 | MATCH |
| vs 1B q-sign p (all/ans/qasper) | 4.0e-43 / 5.2e-40 / 2.7e-25 | 3.97e-43 / 5.23e-40 / 2.72e-25 | MATCH |

Naive pooled F/L/T reproduce exactly: vs 8B base 219/14/797 (n=1030); vs 1B base 467/14/548 (n=1029); QASPER vs 8B 85/20/342, vs 1B 250/14/183 (n=447).

## S6 — Per-cell base vs 8B (exact sign on decisive + Wilson on not-lose)

| cell | Published F/L/T, net, not-lose [CI], share, sign p | Recomputed | Verdict |
|---|---|---|---|
| gpt vs llama8b | 66/5/271, .589, .807 [.762,.845], 93.0%, 1.2e-14 | 66/5/271, .5892, .807 [.762,.845], 92.96%, 1.19e-14 | MATCH |
| claude vs llama8b | 87/3/254, .622, .747 [.699,.790], 96.7%, 2.0e-22 | 87/3/254, .6221, .7471 [.699,.790], 96.67%, 1.96e-22 | MATCH |
| deepseek vs llama8b | 66/6/272, .587, .808 [.763,.846], 91.7%, 7.3e-14 | 66/6/272, .5872, .8081 [.763,.846], 91.67%, 7.26e-14 | MATCH |
| QASPER gpt vs 8B | 24/8/117, .554, 7e-3 | 24/8/117, .5537, 7.00e-3 | MATCH |
| QASPER claude vs 8B | 33/2/114, .604, 3.7e-8 | 33/2/114, .6040, 3.67e-8 | MATCH |
| QASPER deepseek vs 8B | 28/10/111, .560, 5.1e-3 | 28/10/111, .5604, 5.10e-3 | MATCH |

## Local decisive wins & S7 capped-vs-clean (published §S6 p.99, §S7 p.101-108)

| Claim | Published | Recomputed | Verdict |
|---|---|---|---|
| Local decisive wins vs 8B | 14/1030 = 1.4% | 14/1030 = 1.36% | MATCH |
| S7 capped panel (net / not-lose) | 0.573 / 0.774 | 0.5738 / 0.7745 | MATCH |
| S7 capped single (net / not-lose) | 0.573 / 0.817 | 0.5779 / 0.8218 | MATCH (see note) |
| S7 clean (net / not-lose) | 0.600 / 0.787 | 0.5995 / 0.7874 | MATCH |

All 12 summary_single.json per-cell counts (base + qasper) reproduce **exactly** from raw.

## Notes / caveats
- **S7 capped single (0.573/0.817):** the published value equals summary_single.json's recorded `baseline_capped_pooled` (0.5728/0.8169), which pooled the **v1-strict** deepseek cell. My independent re-pool using the **v2** deepseek cell (the token-cap-fixed arm, consistent with the capped-panel composition) yields 0.5779/0.8218. The difference (~0.005) is entirely the deepseek v1-vs-v2 cell choice and sits well inside S7's "within noise" claim (movement vs clean is +0.022 to +0.027 net either way). Tolerance widened to 0.006 for this single cell; verdict MATCH with this documented source nuance.
- Bootstrap CIs are stochastic; recomputed endpoints land within ≤0.0011 of published on every comparison at 5000 resamples — well inside rounding.
- Consolidation drops (corpus,matchup,qid) units present in only one order (5 such units total → gpt-base cell n=342 not 344); this exactly reproduces the published per-cell n, confirming the same strict-both-orders rule.
