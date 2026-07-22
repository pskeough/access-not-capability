# F18 — Training-seed replicates (LoRA-over-base, private corpus)

**Date:** 2026-07-09
**Purpose:** bound training-seed variance on two cells of Table 2 — one TOST-null cell
(Llama-8B) and one Holm-surviving positive cell (Phi-4-mini) — answering the single-seed
reviewer objection ("LoRA effects of 2–6 points could sit inside seed noise").

## Two cells (added Phi-4-mini 2026-07-09)
| cell | paper (seed 42) | seed 43 | seed 44 | mean | SD | vs effect |
|---|---|---|---|---|---|---|
| Llama-8B (TOST-null, 0.488) | 0.4881 | 0.4915 | 0.4966 | 0.4921 | **0.0043** | null stable across seeds |
| Phi-4-mini (Holm-survivor, +5.8pt, 0.558) | 0.5580 | 0.5495 | 0.5666 | 0.5580 | **0.0086** | effect ≈ 6× the seed SD |

Both seed-42 recomputes reproduce the published values (Llama 0.488 Δ0.0001; Phi 0.558 Δ0.0000).
The positive-cell result is the load-bearing one: a genuine +5.8-point LoRA effect with a
0.9-point seed SD is not seed noise. Phi artifacts:
`experiments/v5_raft_cot/artifacts/seed_replicates_phi4mini/summary.json`.

---
## Llama-8B detail (original)

## Design
End-to-end retrain of the headline cell at seeds 43 and 44 (paper = 42), everything
else held fixed: same frozen `train_qa.jsonl` (sha-checked), same frozen 345-question
eval set (hash gate in train_lora), same base answers (copied from the canonical run —
base does not depend on training seed), same grounded judge (qwen/qwen3.6-flash,
temp 0, judging seed 42), both presentation orders, F01-identical strict both-orders
consolidation, answerable-only primary cut (n = 293).

## Raw inputs (exact paths)
- seed 42 (canonical): `experiments/v5_raft_cot/artifacts/runs/20260531-010503/judgments/grounded/pairwise_grounded.jsonl`
- seed 43: `experiments/v5_raft_cot/artifacts/runs/seedrep-43/judgments/grounded/pairwise_grounded.jsonl`
- seed 44: `experiments/v5_raft_cot/artifacts/runs/seedrep-44/judgments/grounded/pairwise_grounded.jsonl`
- configs: `config/pipeline.v5.seed43.yaml`, `config/pipeline.v5.seed44.yaml`
  (diff vs `pipeline.v5.yaml`: `seeds.training` and `run.name` only)
- driver: `scripts/seed_replicates.ps1` | recompute: `scripts/analyze_seed_replicates.py`
- summary artifact: `experiments/v5_raft_cot/artifacts/seed_replicates/summary.json`

## Numbers (answerable, n = 293, ties as half)
| seed | W | L | T | win-rate |
|---|---|---|---|---|
| 42 | 5 | 12 | 276 | 0.4881 |
| 43 | 6 | 11 | 276 | 0.4915 |
| 44 | 8 | 10 | 275 | 0.4966 |

Mean **0.4921**, SD **0.0043**, range [0.4881, 0.4966].
All-questions variant (n = 345): 0.5232 / 0.5261 / 0.5000.

**Consolidation validity check:** the seed-42 recompute reproduces the published
F01 value 0.488 at delta 0.0001 — the replicate seeds are scored by a provably
identical instrument.

## Caveats
- One cell, one model, one corpus; SD from n = 3 seeds (df = 2) is itself an
  estimate. The claim licensed is "the lever ranking sits well clear of seed noise
  on the cell we replicated," not a global seed-variance bound.
- This is the TOST-null cell (0.488 ≡); the replicate shows the *null is stable*
  across seeds, complementing the positive LoRA cells elsewhere in Table 2.
- Judge is API-served (OpenRouter); judged 2026-07-08/09; total judge cost < $1.
