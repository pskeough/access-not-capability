# F20 — Phi-4-mini (3.8B) frontier arm: the middle rung of the size ladder

**Date:** 2026-07-09
**Purpose:** the paper's frontier comparison had two local sizes (1B, 8B) and §6.5
described "a gap between the 1 billion and 8 billion configurations, not a graded
size response; the intermediate arm remains unrun." This runs it.

## Design
Judge-only (no generation): the cached frontier answers (gpt-5.1, claude-sonnet-4.6,
deepseek-v4-pro; base and rag) paired against the cached Phi-4-mini-3.8B private-corpus
RAG answers (canonical run 20260601-122450). Both orders, per-judge strict both-orders,
panel majority quorum-of-two (`_common.panel_majority_verdicts`, the canonical paper
consolidation). Single judge (gemini-3-flash) and 3-judge panel (gemini/mistral/kimi)
both run.

## PROTOCOL CAVEAT (load-bearing for comparison)
This arm used the **capped-evidence panel** (gold + 1600-char evidence block), the SAME
protocol as the paper's capped-panel robustness row, NOT the clean protocol (full context
+ de-verbosity line). The correct 8B comparison is therefore the **capped-panel 8B =
0.573**, not the clean 0.600. The capped panel carries a mild verbosity-reward artifact
(F15), which slightly favors the local model, so the 3.8B point may be marginally
optimistic for the local side; a clean-protocol re-judge would settle it.

## Raw inputs
- Script: `scripts/remediation/phi_frontier_arm.py --panel --max-usd 16`
- Analysis + clustered CIs: `scripts/analyze_phi_frontier_arm.py`
- Judgments: `FinalRunPack/frontier_pilot/phi_arm/judgments_phi_arm_panel.jsonl` (single: `judgments_phi_arm.jsonl`)
- Summaries: `.../summary_phi_arm_panel.json`, `.../analysis_phi_arm.json`
- Local answers: `experiments/v5_raft_cot/artifacts/runs_phi4mini/20260601-122450/answers/rag.jsonl`
- Cost: single-judge $2.42 + panel $8.37 = $10.79

## Numbers (panel, canonical consolidation; verified by analyze_phi_frontier_arm.py)

**Closed-book frontier vs 3.8B+RAG** (n = 1{,}025 pairs / 345 questions):
- F/L/T = 25 / 783 / 217; frontier net win-rate **0.130** [0.109, 0.153]
- **frontier fails to beat: 97.6%**

**Same-retrieval frontier vs 3.8B+RAG** (n = 1{,}022 / 345):
- F/L/T = 197 / 98 / 727; frontier net win-rate **0.548** [0.527, 0.572]
- tie share 71.1%; local not-lose 80.7%
- panel Krippendorff α (nominal) = 0.627

## Size-ladder context (net frontier win-rate under same retrieval; fails-to-beat closed-book)

| local size | closed-book: frontier fails to beat | same-retrieval: net frontier win-rate |
|---|---|---|
| 1B  | 91.4% (panel, 200-q subsample) | 0.720 (clean protocol) |
| **3.8B** | **97.6% (panel, full 345)** | **0.548 [0.527, 0.572] (capped panel)** |
| 8B  | 95.8% (panel, 200-q subsample) | 0.573 (capped panel) / 0.600 (clean) |

## What this licenses (and does NOT)
- **Licensed:** (1) the closed-book access gap is essentially size-independent — the
  frontier fails to beat the local+RAG model at every size, 91–98%. (2) Under the
  MATCHED capped-panel protocol, the 3.8B (0.548 [0.527, 0.572]) is statistically
  indistinguishable from the 8B (0.573, sits at the top of the 3.8B CI): the frontier's
  residual same-retrieval edge is roughly flat from 3.8B to 8B, and both are far below
  the 1B's 0.720. The intermediate rung behaves like the 8B, not like a 1B→8B midpoint.
- **NOT licensed:** (a) a clean-protocol 3-point ladder — the 3.8B is capped-panel only;
  the 1B (0.720) and clean 8B (0.600) are clean protocol. Matched comparisons are 3.8B vs
  8B capped (0.548 vs 0.573) and the closed-book 3-point. (b) "3.8B beats 8B" — the ~2.5
  point gap is within the verbosity-artifact direction of the capped panel. (c) revising
  the "roughly 8B" recommendation downward without a clean-protocol confirm.
- **Sample caveat:** the 1B/8B closed-book panel numbers are on the preregistered 200-q
  subsample (n≈595 pooled); the 3.8B arm is on the full 345 (n≈1025). The metric is
  comparable; the n is not identical.

## Clean-protocol re-judge — DONE 2026-07-09 (removes the protocol caveat)
Ran `scripts/remediation/phi_clean_rejudge.py` (single gemini, full retrieved context +
de-verbosity line, same 345-q private set, both orders, strict both-orders). Cost $6.15.
CI: `scripts/clean_ladder_ci.py`. Result — pooled same-retrieval **net frontier win-rate
0.5589 [0.5411, 0.5773]** (n=1035, local not-lose 0.856, tie 0.83); per-matchup fwr
gpt 0.555 / claude 0.568 / deepseek 0.554. Artifact:
`phi_arm/clean_rejudge/summary_clean_single.json`.

**Clean single-protocol ladder (matched judge AND matched full-345 sample):**
1B 0.720 / **3.8B 0.559 [0.541, 0.577]** / 8B 0.600. The 8B's 0.600 sits OUTSIDE the 3.8B
CI, so under matched conditions the frontier's edge over the 3.8B is at or marginally below
its edge over the 8B, and far below the 1B. Reading: the same-retrieval edge is a 1B-vs-rest
gap, not a graded size response; it reaches its floor by ~4B, and beyond that the specific
model (strong recent Phi-4-mini vs older Llama-3.1-8B) matters more than raw size. NOT a
"smaller model is better" size claim — it is a model-quality difference at similar capability.
