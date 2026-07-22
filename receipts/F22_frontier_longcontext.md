# F22 — Frontier full-context (long-context) arm: the procurement question

**Date:** 2026-07-09
**Purpose:** answer the one arm Part 1 left unrun and a reviewer will demand — the frontier
handed the ENTIRE corpus in-context (the "buy at full strength" procurement option), vs a
local 8B with retrieval, and vs the frontier's own RAG (the Li et al. long-context-vs-RAG
contrast). Pilot N.

## Design
Generation: the full 593-chunk private corpus (~417K tokens, 434K by chunk token_count)
placed in-context each query, same system_qa.txt + question wording as the Part 1 frontier
RAG arm (only the context delivered differs: whole corpus vs top-6). Prompt-cached corpus
prefix (Anthropic cache_control; OpenAI/DeepSeek auto). Judge: single gemini, clean protocol
(gold + full retrieved context + de-verbosity line), both orders, strict both-orders,
question-clustered bootstrap CI. n=90 questions/model (seed 20260709 subset of the 293
answerable), pooled n=180.

## Models
- **GPT-5.1: NOT RUN — context overflow, VERIFIED.** OpenRouter error body: "maximum context
  length is 400000 tokens. However, you requested about 467126 tokens." GPT-5.1's window is
  **400K**; the 416,698-token corpus (467K with the full-context formatting) exceeds it. Control
  test (`gpt_window_probe.py`) confirmed cache_control is NOT the cause (tiny prompt + cache_control
  -> HTTP 200); a 234K-token prefix fits. So full-context is genuinely unavailable for this model
  (a finding: window size gates the procurement option, and a modest 28-paper corpus already
  exceeds a 2026 frontier window).
- **Claude-Sonnet-4.6** (1M window) and **DeepSeek-v4-pro** (1.05M): ran, 90 each.

## Scripts / artifacts
- Gen: `scripts/remediation/frontier_longcontext.py --run --n 90` (probe mode first)
- Judge: `scripts/remediation/frontier_longcontext_judge.py`
- `FinalRunPack/frontier_pilot/longcontext/{lc_claude,lc_deepseek}.jsonl`, `summary_lc.json`,
  `judgments_lc.jsonl`
- Cost: gen ~$14 (Claude cache-write dominates) + judge $2.7 ≈ $17.

## Numbers (LC = frontier full-context; net = LC win-rate, ties=0.5)
| comparison | Claude | DeepSeek | pooled n=180 | reference |
|---|---|---|---|---|
| **LC vs local-8B-RAG** (procurement) | 0.622 | 0.500 | **0.561 [0.508, 0.614]** | frontier-RAG vs 8B = 0.600 [F06] |
| **LC vs the model's own RAG** (Li et al.) | 0.478 | 0.389 | **0.433 [0.392, 0.475]** | 0.5 = parity |

- Procurement: 8B+RAG not-lose 0.889 (loses to a whole-corpus frontier only 11% of the time).
- Own-RAG: full-context not-lose 0.811, but net **0.433 with upper CI 0.475 < 0.5** → retrieval
  significantly beats full-context for the same frontier model.

## What this licenses
- **The procurement "buy at full strength" option does not beat local+RAG.** Giving the
  frontier the entire corpus scores 0.561 [0.508, 0.614] against the local 8B+RAG, no better
  than the same frontier given only retrieval (0.600), DeepSeek at exact parity (0.500). The
  access gap does not widen when access is maximized.
- **Retrieval beats full-context for the frontier itself**, on a 417K-token corpus: pooled
  0.433 [0.392, 0.475], significant. Directly measures the lost-in-the-middle degradation at
  a corpus 3x beyond where long-context models use context well; the opposite of Li et al.'s
  smaller-setup finding, and it reinforces retrieval-first.
- **Window size gates the option**: GPT-5.1 cannot hold the corpus at all.

## Caveats (pilot scope)
- n=90/model, pooled n=180; CI ~±0.04. A pilot, not the 293-question full set. Extends
  resume-safe (generations banked; only add questions).
- Two models (GPT excluded by overflow, itself reported).
- Single gemini judge (clean protocol; matches the paper's clean ladder numbers).
- Corpus assembled by concatenating parent_text in file order — the realistic naive
  "paste the whole corpus" baseline the objection proposes, not an order-optimized variant.
