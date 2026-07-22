# F09 — Arena (Bradley-Terry) + ROUGE-L receipt

**Family:** F09 arena-rouge
**Recompute script:** `C:/Research/Train_LLM/Final_Validation/scripts/F09_arena_rouge.py` (runs offline, no API, idempotent, exit 0; 21/21 checks pass)
**Scratch dump:** `C:/Research/Train_LLM/Final_Validation/scratch/F09_recompute.json`

All numbers below are recomputed from RAW with independent code: my own panel-majority +
strict both-orders consolidation, my own MM Bradley-Terry, my own nominal Krippendorff alpha,
and my own LCS-based ROUGE-L. I import no project analysis code. `collapse_repetition` and the
`normalize_answer` content-keeping step are re-implemented from scratch (faithful to
`src/pipeline/lib/text_norm.py` and `scripts/smoke_eval_grounded.py`) so the published ROUGE
numbers reproduce exactly.

## Raw inputs
- Arena rows (panel judgments, 3 judges x 2 orders):
  - base: `C:/Research/Train_LLM/experiments/v5_raft_cot/artifacts/arena/arena_pairwise.jsonl` (24,831 rows)
  - covid: `C:/Research/Train_LLM/experiments/v5_raft_cot/artifacts/covidqa/arena/arena_pairwise.jsonl` (10,340 rows)
  - qasper: `C:/Research/Train_LLM/experiments/v5_raft_cot/artifacts/qasper/arena/arena_pairwise.jsonl` (10,799 rows)
- Eval sets (gold + answerable flag = `source_chunk_ids`):
  - base: `C:/Research/Train_LLM/experiments/v4_raft_cot/data/eval_set.jsonl`
  - covid: `C:/Research/Train_LLM/experiments/v5_raft_cot/corpora/covidqa/eval_set.jsonl`
  - qasper: `C:/Research/Train_LLM/experiments/v5_raft_cot/corpora/qasper/eval_set.jsonl`
- Model answers: `C:/Research/Train_LLM/experiments/v5_raft_cot/artifacts/{runs/20260531-010503, runs_phi4mini/20260601-122450, runs_minicpm1b/20260531-111249, covidqa/<model>/<ts>, qasper/<model>/<ts>}/answers/{base,rag,lora,lora_rag}.jsonl`
- Comparison targets: `FinalRunPack/artifacts/arena/{base,covidqa,qasper}_arena_report.json`, `FinalRunPack/artifacts/rouge_triangulation.json`

## Method (2–3 lines each)
- **Arena:** per (qid, pair, order) take strict-majority of the 3 judges' winner labels (else tie); consolidate the two presentation orders strictly (decisive only if both orders name the same model, else tie); win-rate counts ties as half. MM Bradley-Terry over decisive pairs, strengths normalized to geometric mean 1. Nominal Krippendorff alpha over the raw 3-judge winner labels.
- **ROUGE-L:** for answerable questions (those with `source_chunk_ids`), normalize each model answer (collapse degenerate repetition, keep text after last `Answer:`, strip `[n]`), tokenize `[a-z0-9]+`, compute token-level LCS via rolling DP, report ROUGE-L F1 vs `gold_answer`, average over questions present in both answer file and eval set.

---

## (a) Bradley-Terry + Krippendorff alpha

### Recomputed BT scores (n_iter=400, geometric-mean-1 normalized), ranking, and panel alpha

| corpus | config | ranking (BT desc) | GOLD BT | best non-GOLD (BT) | alpha | GOLD losses |
|---|---|---|---|---|---|---|
| base | rag | GOLD > phi4mini > llama8b > minicpm1b | 56.5316 | phi4mini (0.6145) | 0.6308 | 2 (convergent) |
| base | lora_rag | GOLD > llama8b > phi4mini > minicpm1b | 192.8415 | llama8b (0.3635) | 0.6251 | 0 (DIVERGES) |
| covid | rag | GOLD > llama8b > phi4mini > minicpm1b | 31.7852 | llama8b (0.7885) | 0.6409 | 2 (convergent) |
| covid | lora_rag | GOLD > llama8b > phi4mini > minicpm1b | 49.1136 | llama8b (0.7162) | 0.6422 | 1 (convergent) |
| qasper | rag | GOLD > phi4mini > llama8b > minicpm1b | 275.0497 | phi4mini (0.2599) | 0.5659 | 0 (DIVERGES) |
| qasper | lora_rag | GOLD > phi4mini > llama8b > minicpm1b | 301.1060 | phi4mini (0.3123) | 0.5750 | 0 (DIVERGES) |

| # | Claim (published) | Published value | Recomputed | Verdict |
|---|---|---|---|---|
| 1 | GOLD dominates every ranking | (base_arena_report.json etc., `ranking[0]`="GOLD" all cells) | GOLD top + strictly highest BT in all 6 cells | **MATCH** |
| 2 | base-rag GOLD BT 56.5 vs best model 0.61 | base_arena_report.json → configs.rag.bradley_terry (GOLD 56.5316, phi4mini 0.6145) | GOLD 56.5316, phi4mini 0.6145 (best non-GOLD) | **MATCH** |
| 3 | base-lora_rag GOLD BT 192.8 vs 0.36 | base_arena_report.json → configs.lora_rag (GOLD 192.8415, llama8b 0.3635) | GOLD 192.8415, llama8b 0.3635 (best non-GOLD) — reproduces ONLY at n_iter=400 (see NEW finding) | **MATCH** (at published settings) |
| 4 | COVID best model → Llama, both configs | covidqa_arena_report.json `ranking[1]`="llama8b" both | llama8b best non-GOLD in rag and lora_rag | **MATCH** |
| 5 | QASPER best model → Phi, both configs | qasper_arena_report.json `ranking[1]`="phi4mini" both | phi4mini best non-GOLD in rag and lora_rag | **MATCH** |
| 6 | base flips: rag→Phi, lora_rag→Llama | base_arena_report.json rankings | rag best=phi4mini, lora_rag best=llama8b | **MATCH** |
| 7 | Panel alpha base 0.63 | base_arena_report.json (rag 0.6308) | 0.6308 | **MATCH** |
| 8 | Panel alpha COVID 0.64 | covidqa_arena_report.json (rag 0.6409) | 0.6409 | **MATCH** |
| 9 | Panel alpha QASPER 0.57 | qasper_arena_report.json (rag 0.5659) | 0.5659 | **MATCH** |

All four published COVID/QASPER GOLD anchors (covid-rag 31.7852, covid-lora_rag 49.1136,
qasper-rag 275.0497, qasper-lora_rag 301.1060) also reproduce to |diff|<0.1 at n_iter=400.

### NEW finding — three GOLD anchors are non-convergent (n_iter artifacts)

The MM Bradley-Terry MLE does not exist for a player that is **never beaten** — its strength
diverges to infinity as iterations increase. In exactly three cells GOLD has 0 decisive losses,
so the reported GOLD BT magnitude is purely a function of the iteration budget (the production
`analyze_arena.py` point estimate stops at `n_iter=400`):

| cell | GOLD losses | GOLD BT @400 (published) | GOLD BT @4000 |
|---|---|---|---|
| base-lora_rag | 0 | 192.84 | 1085.40 |
| qasper-rag | 0 | 275.05 | 1546.81 |
| qasper-lora_rag | 0 | 301.11 | 1693.85 |

Convergent cells (GOLD has ≥1 loss: base-rag 2, covid-rag 2, covid-lora_rag 1) are stable and
iteration-independent. **Impact: none on any scientific claim** — GOLD's *dominance*, the model
*rankings*, the corpus-dependent best model, and the non-GOLD BT scores are all robust; only the
raw GOLD anchor magnitude in the three never-beaten cells is non-identifiable. Recommend the paper
either cap the GOLD anchor, report it as a lower bound, or note it is convergence-budget-dependent.

---

## (b) ROUGE-L F1 vs gold (answerable-only), own LCS

Recomputed full grid (matches `rouge_triangulation.json` exactly, max abs diff = 0.000 over all 36 cells):

| corpus | model | base | rag | lora | lora_rag | n |
|---|---|---|---|---|---|---|
| base | llama8b | 0.223 | 0.480 | 0.172 | 0.559 | 293 |
| base | phi4mini | 0.388 | 0.562 | 0.466 | 0.550 | 293 |
| base | minicpm1b | 0.178 | 0.402 | 0.238 | 0.488 | 293 |
| covidqa | llama8b | 0.090 | 0.340 | 0.102 | 0.381 | 400 |
| covidqa | phi4mini | 0.087 | 0.320 | 0.125 | 0.357 | 400 |
| covidqa | minicpm1b | 0.060 | 0.227 | 0.085 | 0.269 | 400 |
| qasper | llama8b | 0.067 | 0.091 | 0.087 | 0.125 | 302 |
| qasper | phi4mini | 0.051 | 0.098 | 0.067 | 0.116 | 302 |
| qasper | minicpm1b | 0.039 | 0.087 | 0.049 | 0.076 | 302 |

| # | Claim (published) | Published value | Recomputed | Verdict |
|---|---|---|---|---|
| 10 | base-llama quadruple 0.223/0.480/0.172/0.559 | rouge_triangulation.json row[0]; _ARTIFACT_INDEX.md L41 | 0.223 / 0.480 / 0.172 / 0.559 | **MATCH** |
| 11 | rag, lora_rag > base, lora on all 9 corpus-model rows | rouge_triangulation.py "expected" ordering | min(rag,lora_rag) > max(base,lora) holds on all 9 rows | **MATCH** |
| 12 | Answerable n per corpus | base 293 / covid 400 / qasper 302 | 293 / 400 / 302 | **MATCH** |
| 13 | ROUGE grid vs rouge_triangulation.json | full 9x4 grid | max abs diff = 0.000 (all 36 cells exact) | **MATCH** |

**ROUGE method note:** own token-level LCS via rolling DP (no `rouge_score` library — it is not
installed in the project .venv). Normalization (`collapse_repetition` + Answer:-keeping + `[n]`
stripping) re-implemented independently. The repetition-collapse step is a genuine no-op for
Llama/Phi but DOES fire for minicpm1b's degenerate loops; omitting it left 5 minicpm cells off by
up to 0.018 (e.g. qasper-minicpm-lora_rag 0.058 vs 0.076), confirming it is load-bearing for the
small-model rows. With it included, all cells reproduce exactly.

## Issues / notes
- The three GOLD BT anchors in never-beaten cells (base-lora_rag, qasper-rag, qasper-lora_rag) are convergence-budget artifacts; reproducible only at n_iter=400. Flagged as a NEW finding; no claim is affected.
- `FinalRunPack/artifacts/rouge_triangulation.json` is a JSON array (not JSONL) — handled.
