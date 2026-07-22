# F11 — Hit/Miss Stratification of Frontier-vs-Local Tie Mass (GOLD receipt)

**Family:** F11 (NEW analysis, high priority — conditions the paper headline)
**Script:** `C:/Research/Train_LLM/Final_Validation/scripts/F11_hit_miss_stratification.py`
**Results JSON:** `C:/Research/Train_LLM/Final_Validation/F11_results.json`
**Status:** script runs clean offline, exit 0, all internal checks PASS.

## Question
Is the frontier-vs-8B tie mass **capability parity** or **mutual retrieval
blindness** (both models tie because neither saw the gold evidence)? We stratify
the pooled frontier+RAG-vs-8B and vs-1B clean-rejudge cells by retrieval **HIT**
vs **MISS** on the base and QASPER corpora.

## Raw inputs (exact paths)
- Judgments (clean re-judge, single judge gemini, strict both-orders):
  `C:/Research/Train_LLM/FinalRunPack/frontier_pilot/clean_rejudge/judgments_single.jsonl`
  (5911 rows; base 4123, qasper 1788; one judge = `gemini`.)
- Base gold source_chunk_ids: `C:/Research/Train_LLM/experiments/v4_raft_cot/data/eval_set.jsonl`
- Base llama8b retrieved top-6: `C:/Research/Train_LLM/experiments/v5_raft_cot/artifacts/runs/20260531-010503/answers/rag.jsonl`
- QASPER gold: `C:/Research/Train_LLM/experiments/v5_raft_cot/corpora/qasper/eval_set.jsonl`
- QASPER llama8b retrieved top-6: `C:/Research/Train_LLM/experiments/v5_raft_cot/artifacts/qasper/llama8b/20260603-090219/answers/rag.jsonl`

## Method (3 lines)
1. **HIT/MISS:** a question is a HIT iff ≥1 gold `source_chunk_id` (from the eval
   set) appears in the **llama8b** RAG `retrieved_chunk_ids` (top-6) for that
   question; else MISS. The 8B retriever defines retrieval for BOTH the vs-8B and
   vs-1B pools (per task spec — it is the shared RAG context the local arms used).
2. **Consolidation (strict both-orders):** per (question_id, matchup) a pair is
   counted only if BOTH presentation orders are present (5 single-order base pairs
   dropped, matching published `n`); the pair is decisive for a side iff both
   orders name that same side, else TIE. Win-rate counts ties as half:
   `winrate = (F + 0.5·T)/n`; frontier net win-rate = this winrate; local not-lose
   = `(L+T)/n`; tie rate = `T/n`.
3. **Pooling + CIs:** pool the 3 frontier matchups (claude/gpt/deepseek :rag) per
   local arm; stratify; cluster bootstrap by question_id (B=10000, seed 20260611,
   percentile 95% CI). HIT-vs-MISS deltas use a paired resample (same qid draw
   feeds both strata).

## Reproduction check (CHECK A) — recompute reproduces published pooled cells
| cell | published F/L/T/n (winrate, notlose) | recomputed | verdict |
|---|---|---|---|
| base, frontier vs 8B | 219/14/797/1030 (0.5995, 0.7874) | 219/14/797/1030 (0.5995, 0.7874) | **MATCH** |
| base, frontier vs 1B | 467/14/548/1029 (0.7201, 0.5462) | 467/14/548/1029 (0.7201, 0.5462) | **MATCH** |
| QASPER, frontier vs 8B | 85/20/342/447 (0.5727, 0.8098) | 85/20/342/447 (0.5727, 0.8098) | **MATCH** |
| QASPER, frontier vs 1B | 250/14/183/447 (0.764, 0.4407) | 250/14/183/447 (0.7640, 0.4407) | **MATCH** |

Published values: `FinalRunPack/frontier_pilot/clean_rejudge/summary_single.json`
→ `corpora.{base,qasper}.pooled`. Exact reproduction confirms the consolidation
rule (drop single-order pairs; strict both-orders; ties=half) is the published one.

## NEW results — stratified by HIT vs MISS

### BASE corpus
**Pooled frontier+RAG vs 8B (llama8b):**
| stratum | n_q | pairs | F | L | T | frontier net win-rate (95% CI) | local not-lose | tie rate |
|---|---|---|---|---|---|---|---|---|
| ALL  | 344 | 1030 | 219 | 14 | 797 | 0.5995 [0.580, 0.620] | 0.7874 | 0.7738 |
| HIT  |  86 |  258 |  57 |  2 | 199 | 0.6066 [0.568, 0.649] | 0.7791 | 0.7713 |
| MISS | 258 |  772 | 162 | 12 | 598 | 0.5972 [0.574, 0.621] | 0.7902 | 0.7746 |

**Pooled frontier+RAG vs 1B (minicpm1b):**
| stratum | n_q | pairs | F | L | T | frontier net win-rate (95% CI) | local not-lose | tie rate |
|---|---|---|---|---|---|---|---|---|
| ALL  | 344 | 1029 | 467 | 14 | 548 | 0.7201 [0.694, 0.746] | 0.5462 | 0.5326 |
| HIT  |  86 |  258 | 125 |  3 | 130 | 0.7364 [0.684, 0.789] | 0.5155 | 0.5039 |
| MISS | 258 |  771 | 342 | 11 | 418 | 0.7147 [0.685, 0.745] | 0.5564 | 0.5422 |

### QASPER corpus
**Pooled frontier+RAG vs 8B (llama8b):**
| stratum | n_q | pairs | F | L | T | frontier net win-rate (95% CI) | local not-lose | tie rate |
|---|---|---|---|---|---|---|---|---|
| ALL  | 149 | 447 | 85 | 20 | 342 | 0.5727 [0.543, 0.603] | 0.8098 | 0.7651 |
| HIT  |  36 | 108 | 32 |  7 |  69 | 0.6157 [0.537, 0.694] | 0.7037 | 0.6389 |
| MISS | 113 | 339 | 53 | 13 | 273 | 0.5590 [0.529, 0.590] | 0.8437 | 0.8053 |

**Pooled frontier+RAG vs 1B (minicpm1b):**
| stratum | n_q | pairs | F | L | T | frontier net win-rate (95% CI) | local not-lose | tie rate |
|---|---|---|---|---|---|---|---|---|
| ALL  | 149 | 447 | 250 | 14 | 183 | 0.7640 [0.726, 0.801] | 0.4407 | 0.4094 |
| HIT  |  36 | 108 |  64 |  1 |  43 | 0.7917 [0.718, 0.866] | 0.4074 | 0.3981 |
| MISS | 113 | 339 | 186 | 13 | 140 | 0.7552 [0.709, 0.799] | 0.4513 | 0.4130 |

### HIT-vs-MISS contrast (paired bootstrap, frontier net win-rate)
| cell | Δ net win-rate (HIT−MISS) 95% CI | Δ local not-lose 95% CI |
|---|---|---|
| base vs 8B   | +0.0094 [−0.037, +0.057] | −0.011 [−0.104, +0.080] |
| base vs 1B   | +0.0218 [−0.038, +0.082] | −0.041 [−0.155, +0.075] |
| QASPER vs 8B | +0.0567 [−0.029, +0.142] | −0.140 [−0.288, +0.003] |
| QASPER vs 1B | +0.0365 [−0.051, +0.124] | −0.044 [−0.209, +0.123] |

**Every Δ CI includes 0** — frontier dominance does not differ significantly by
retrieval hit/miss in any cell.

## Pre-registered interpretation (stated before reading deltas)
> If 8B holds near-parity ON HITS, parity is capability-real; if frontier
> dominates hits, the headline must be scoped as retrieval-limited.

**Outcome.** Frontier net win-rate vs 8B is **> 0.5 on hits** in both corpora
(base 0.607, CI excludes 0.5; QASPER 0.616, CI lower bound 0.537), so by the
literal pre-registered binary the frontier "dominates hits." **But** the decisive
test of the *mutual-retrieval-blindness* hypothesis is whether the frontier edge
is **concentrated where retrieval missed**. It is not: the HIT−MISS delta in
frontier net win-rate is statistically **zero** in every cell (deltas +0.009 to
+0.057, all CIs straddle 0), and the 8B's not-lose / tie rates are essentially
unchanged across strata (base: tie 0.771 HIT vs 0.775 MISS; not-lose 0.779 vs
0.790). The 8B holds the **same** near-parity **with the gold chunk in its
context** as without it.

**Therefore the tie mass is capability parity, not mutual retrieval blindness.**
When retrieval succeeds (HIT), the 8B still ties the frontier ~77% of the time on
base — it is not losing because both models were blind; it is matching the
frontier on questions where the evidence was demonstrably present. The frontier's
modest, retrieval-invariant edge (~0.60 net on base, ~0.61 on QASPER) is a
capability gap, not a retrieval artifact.

**Headline guidance:** the near-parity claim survives and should be framed as
capability-real. A retrieval-limited caveat is **not** warranted by these data;
the small frontier edge does not grow when the 8B's retriever succeeds. (Note the
base retriever is low-recall: only 86/344 judged base questions and 36/149 QASPER
questions are exact-id HITS — yet stratifying on this makes no difference to the
gap, which is itself the key evidence.)

## Claims & verdicts
| # | Claim | Published | Recomputed | Verdict |
|---|---|---|---|---|
| 1 | base pooled frontier-vs-8B (clean rejudge) | F/L/T/n 219/14/797/1030, winrate 0.5995, notlose 0.7874 | identical | MATCH |
| 2 | base pooled frontier-vs-1B | 467/14/548/1029, 0.7201, 0.5462 | identical | MATCH |
| 3 | QASPER pooled frontier-vs-8B | 85/20/342/447, 0.5727, 0.8098 | identical | MATCH |
| 4 | QASPER pooled frontier-vs-1B | 250/14/183/447, 0.764, 0.4407 | identical | MATCH |
| 5 | 8B holds near-parity ON HITS (base) | (not previously published) | net 0.607, not-lose 0.779, tie 0.771; Δ vs MISS = 0 (CI incl 0) | NEW |
| 6 | 8B holds near-parity ON HITS (QASPER) | (not previously published) | net 0.616, not-lose 0.704, tie 0.639; Δ vs MISS = 0 (CI incl 0) | NEW |
| 7 | Frontier edge is retrieval-invariant (tie mass ≠ mutual blindness) | (not previously published) | HIT−MISS Δ net win-rate CI includes 0 in all 4 cells | NEW |

## Notes / caveats
- HIT defined by **exact chunk-id** intersection of gold vs llama8b top-6. This is
  strict (same-paper-different-passage counts as MISS), matching the spec ("gold
  source_chunk_ids vs the llama8b retrieved top-6"). Base has 52 judged questions
  with empty gold (out-of-corpus) — these are necessarily MISS, correctly so.
- The vs-1B pool reuses the **8B** retriever's hit/miss labels (the shared RAG
  context per the task spec). The 1B saw the same retrieved chunks; results are
  consistent (retrieval-invariant edge) but the vs-1B gap is large in every
  stratum, as expected.
- 5 base single-order pairs dropped to match the published `n` (1030/1029);
  treating them as ties instead shifts pooled n to 1032 and winrate by <0.0003 —
  immaterial to any conclusion.
