# F19 — Cross-judge bridge: within-model judge vs frontier panel

**Date:** 2026-07-09 (receipt added; underlying run 2026-06-01)
**Purpose:** put the paper's two result families on one measurement scale. The within-model
grid (Results 1, 2, 4) is scored by a single Qwen-family judge; the frontier arm by a
family-firewalled Gemini/Mistral/Kimi panel. This bridge quantifies their agreement, and
doubles as the structural mitigation for the judge-gold family affinity disclosed in
Threats (Qwen3.6-Flash judge scoring against Qwen-2.5-14B-written gold).

## Design
Pre-registered per the script docstring (EVAL_DESIGN_REPORT §9): 75 qwen-judged
within-model pairs (llama8b, canonical run) re-judged by all three panel judges under the
identical grounded prompt, both orders, strict consolidation; qwen verdict compared to the
panel majority.

## Raw inputs (exact paths)
- Script (recompute): `scripts/cross_judge_bridge.py`
- Artifact: `experiments/v5_raft_cot/artifacts/runs/20260531-010503/judgments/grounded/cross_judge_bridge_result.json`
- Row-level: `.../judgments/grounded/cross_judge_bridge.jsonl`

## Numbers (verified against the artifact 2026-07-09)
| quantity | value |
|---|---|
| units compared | 75 |
| Krippendorff α, qwen vs panel majority | **0.8379** |
| exact agreement, qwen vs panel majority | **0.8933** (67/75) |
| per-judge agreement vs qwen | gemini 0.893, kimi 0.840, mistral 0.707 |
| α all four raters | 0.687 |
| α panel-only | 0.657 |
| by question type (qwen vs majority) | single-hop 0.881, multi-hop 0.813, definitional / OOC / paraphrase 1.00 |

## Caveats
- Scope: llama8b pairs only (the canonical within-model run); the bridge was not run on
  the phi4mini or minicpm grids.
- n = 75 units; α at this n carries meaningful sampling error and is quoted to two
  decimals in the paper (0.84) without a CI.
- The mistral judge's lower agreement (0.707) is consistent with its position bias
  documented in F15; the panel *majority* is the comparison target, which contains it.

## Paper sentences this receipt carries
- §3.3: "...the single judge agrees with the panel majority at Krippendorff's α = 0.84
  (89 percent exact agreement)..."
- §8 Gold provenance: "...the cross-judge bridge of Section 3.3 shows the Qwen judge
  agreeing with the non-Qwen panel majority at α = 0.84."
