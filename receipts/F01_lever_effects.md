# F01 — Lever effects (36 within-model cells) — GOLD receipt

**Recompute script:** `C:/Research/Train_LLM/Final_Validation/scripts/F01_lever_effects.py`
(idempotent, offline, no API; `36/36 checks PASS`, exit 0). Re-derives every number below
from raw `pairwise_grounded.jsonl` with independent code; compares against — but does not trust —
`cross_corpus_stats.json`.

## Method (2-3 lines)
For each of 3 corpora × 3 models × 4 levers: load raw grounded both-orders judgments, restrict to
answerable qids (exclude `out_of_corpus` per the eval set; COVID has none), **strict both-orders
consolidation** (a qid is decisive for a config only if BOTH presentation-order rows name that one
config; any tie / disagreement / missing order ⇒ tie). Win-rate = (w + t/2)/n. McNemar = exact
two-sided binomial on the w+l discordant pairs. 2000-resample percentile bootstrap 95% CI
(seed 12345). Schuirmann TOST (±0.05) via two one-sided t-tests on the {0,0.5,1} score vector.
Global Holm + BH applied to the 36 McNemar p-values.

## Raw inputs (exact paths)
- Grounded judgments (corpus/model → file):
  - base: `experiments/v5_raft_cot/artifacts/runs/20260531-010503` (llama8b),
    `…/runs_minicpm1b/20260531-111249` (minicpm1b), `…/runs_phi4mini/20260601-122450` (phi4mini),
    each `…/judgments/grounded/pairwise_grounded.jsonl`.
  - covidqa: `…/artifacts/covidqa/{llama8b/20260605-230052, minicpm1b/20260606-120325, phi4mini/20260606-105705}/judgments/grounded/pairwise_grounded.jsonl`.
  - qasper: `…/artifacts/qasper/{llama8b/20260603-090219, minicpm1b/20260603-153951, phi4mini/20260603-135816}/judgments/grounded/pairwise_grounded.jsonl`.
- Eval sets (answerable filter): base `experiments/v4_raft_cot/data/eval_set.jsonl`;
  covid `experiments/v5_raft_cot/corpora/covidqa/eval_set.jsonl`;
  qasper `experiments/v5_raft_cot/corpora/qasper/eval_set.jsonl`.
- Comparison only (not trusted): `FinalRunPack/artifacts/cross_corpus_stats.json`.

## Lever → config-pair mapping (matches published `cross_corpus_stats.py` EFFECT_LABEL)
| lever | control | treatment | meaning |
|---|---|---|---|
| RAG-over-base | base | rag | retrieval vs no-retrieval |
| RAG-over-LoRA | lora | **lora_rag** | retrieval added on top of LoRA (NOT rag-vs-lora) |
| LoRA-over-base | base | lora | fine-tuning vs no-tuning |
| RAFT-over-RAG | rag | lora_rag | RAFT synergy (LoRA on top of RAG) |

> NOTE: "RAG-over-LoRA" is the retrieval effect measured on the LoRA arm (`lora_rag` vs `lora`).
> A naive reading as `rag` vs `lora` gives different counts and does NOT reproduce the paper.

## Verdicts

### Specific T1 win-rates (01_FINDINGS.md §1, STATISTICAL_SUPPLEMENT.md S1)
| claim | published | recomputed | verdict |
|---|---|---|---|
| base llama8b RAG-over-base | 84.1% | 84.1% (w/l/t 205/5/83, n=293) | MATCH |
| base phi4mini RAG-over-base | 84.8% | 84.8% (210/6/77) | MATCH |
| base minicpm1b RAG-over-base | 81.2% | 81.2% (184/1/108) | MATCH |
| COVID llama8b RAG-over-base | 86.4% | 86.4% (300/9/91, n=400) | MATCH |
| COVID phi4mini RAG-over-base | 83.6% | 83.6% (282/13/105) | MATCH |
| COVID minicpm1b RAG-over-base | 79.2% | 79.2% (237/6/152, n=395*) | MATCH |
| QASPER llama8b RAG-over-base | 59.6% | 59.6% (77/19/206, n=302) | MATCH |
| QASPER phi4mini RAG-over-base | 60.9% | 60.9% (73/7/222) | MATCH |
| QASPER minicpm1b RAG-over-base | 62.3% | 62.3% (74/0/228) | MATCH |
| base llama8b LoRA-over-base (ns) | 48.8%, ns | 48.8% (5/12/276); McNemar p=0.143; Holm non-survivor; TOST-equivalent to 0.5 | MATCH |
| QASPER phi4mini LoRA-over-base (ns) | 51.5%, ns | 51.5% (10/1/291); Holm non-survivor; TOST-equivalent | MATCH |
| LoRA-over-base remaining 7 cells | base-phi 55.8, base-mc 53.2, COVID 53.0/54.4/54.9, QASPER-llama 53.5, QASPER-mc 52.9 | all reproduced exactly | MATCH |
| RAFT COVID-MiniCPM | 58.0%, p<1e-4 | 58.0% (123/60/211); McNemar p=3.7e-6 | MATCH |
| RAFT QASPER-Llama | 54.6%, p=8e-4 | 54.6% (47/19/236); McNemar p=7.6e-4 | MATCH |
| RAFT COVID-Phi near-null | 50.9%, p=4e-4 (TOST) | 50.9% (51/44/305); TOST-equivalent to 0.5 | MATCH |

*COVID-minicpm1b raw file has 4724/4800 rows (76 missing) → n=393–395 in its cells; the published
cell is identically n=395 with the same 237/6/152 split. Disclosed below.

### Global multiple-comparison control (family = 36)
| claim (01_FINDINGS §1 / S1) | published | recomputed | verdict |
|---|---|---|---|
| Holm survivors total | 27 | 27 | MATCH |
| Holm retrieval (RAG-over-base + RAG-over-LoRA) | 18/18 | 18/18 | MATCH |
| Holm LoRA-over-base | 7/9 | 7/9 | MATCH |
| Holm RAFT-over-RAG | 2/9 | 2/9 | MATCH |
| BH survivors total | 29 | 29 | MATCH |
| BH LoRA-over-base | 8/9 | 8/9 | MATCH |
| BH RAFT-over-RAG | 3/9 | 3/9 | MATCH |
| BH retrieval | 18/18 | 18/18 | MATCH |

### Full-table reproduction vs cross_corpus_stats.json
All **36/36** cells match published w/l/t and win-rate (tolerance 0.0011); **no divergence**.
RAG-over-LoRA cells match once the correct `lora_rag`-vs-`lora` mapping is used.

### McNemar / TOST spot-checks
- base llama8b RAG-over-base McNemar p ≈ 4.0e-54 (≪1e-4, "p<1e-4" ✓).
- TOST-equivalent-to-0.5 cells reproduced: base-llama LoRA, qasper-phi LoRA, base-minicpm LoRA,
  qasper-minicpm LoRA, COVID-llama LoRA, COVID-llama RAFT, COVID-phi RAFT, qasper-phi RAFT,
  qasper-minicpm RAFT, base-phi RAFT — i.e. the near-null LoRA/RAFT cells the docs flag as genuine.

## Issues / divergences
1. **No numeric divergence** from the published within-model cells: all 36 reproduce exactly.
2. **COVID-MiniCPM incompleteness (disclosed, not an error):** its raw grounded file has 4724/4800
   rows (≈76 missing, including one single-order qid in the `(base,rag)` pair). Strict both-orders
   correctly demotes the single-order qid to a tie, giving n=395 and the published 237/6/152 — so the
   published number already accounts for this. base & QASPER are complete; COVID llama8b/phi4mini are
   complete (n=400).
3. **Naming caveat (resolved):** "RAG-over-LoRA" denotes `lora_rag` vs `lora`, not `rag` vs `lora`;
   the wrong reading fails to reproduce. Documented in the script and mapping table above.

**Overall: every F01 number in 01_FINDINGS.md §1 and STATISTICAL_SUPPLEMENT.md S1 is MATCH.**
