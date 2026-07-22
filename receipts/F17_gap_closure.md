# F17 — gap closure (the 5 claims F16 found uncovered)

Script: `scripts/F17_gap_closure.py` (offline, idempotent, exit 0, all checks PASS).
Raw inputs: `FinalRunPack/frontier_pilot/judgments_panel.jsonl`, `clean_rejudge/judgments_single.jsonl`,
`rag/data/chunks.jsonl`, `experiments/v5_raft_cot/_analysis/master_analysis/scripts/frontier_pilot.py`.

| # | claim (06_PAPER_SKELETON) | recomputed | verdict |
|---|---|---|---|
| 1a | gpt+claude-only capped-panel pool vs 8B: not-lose 75.2% / net 0.591 | F/L/T=99/26/275, n=400 → **0.7525 / 0.5913** | **MATCH** |
| 1b | same pool under the CLEAN re-judge (now the relevant cut) | F/L/T=153/8/525, n=686 → not-lose **0.7770**, net **0.6057** | **NEW** — gpt+claude clean pool ≈ full 3-model pool (0.600); deepseek-v2 inclusion no longer moves it; the interim "quote clean-cells only" guidance is obsolete |
| 2 | capped-panel question-clustered sign test vs 8B: 60:26, p=3.2e-4 | qF/qL/qT = **60/26/114**, exact two-sided p = **3.17e-4** | **MATCH** |
| 3 | base corpus ≈420K tokens ("fits a 1M context window") | 593 chunks, 28 papers, 1,688,029 chars → **416,698 tokens** (tiktoken o200k_base) | **MATCH** |
| 4 | base corpus = 28 *published* papers (provenance reframe) | 28 paper_ids; **13 carry explicit OpenAlex W-ids**; the other 15 are recognizable published-paper slugs (Sadler & Tai, Schwartz depth-vs-breadth, Dabney 2011 IJSE, Andriole 2016, Tai 2017…) | **MATCH** (reframe supported; "contamination-free by construction" remains indefensible) |
| 5 | decoding disclosure: silent temperature-drop-on-400 fallback; max_tokens=512 | fallback present in frontier_pilot.py; max_tokens=512 hardcoded; no temperature literal found in current source (drop path active) | **MATCH** (disclosure item confirmed; full per-arm decoding table remains a writing task, not a data gap) |

**NEW (flag):** *"Planning Early for Careers in Science"* appears under **two** paper_ids
(`2006_Planning_Early_for_Careers_in_Science_W1557291404` and `Planning_Early_for_Careers_In_Science`)
— a possible duplicate document in the base corpus. Impact bound: ≤2× weight for one paper in
retrieval and homogeneity; does not touch any judged comparison directly (retrieval was identical
across arms). Disclose in the corpus table; optionally diff the two texts before camera-ready.
