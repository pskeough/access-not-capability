# F08 — deepseek-v2 frontier rejudge (GOLD receipt)

**Family:** F08 deepseek-v2
**Recompute script:** `C:/Research/Train_LLM/Final_Validation/scripts/F08_deepseek_v2.py` (offline, idempotent, exit 0; 28/28 PASS)
**Published source:** `C:/Research/Train_LLM/FinalRunPack/frontier_pilot/summary_deepseek_v2.json` and `C:/Research/Train_LLM/FinalRunPack/frontier_pilot/INTERPRETATION.md`; claim→artifact map `C:/Research/Train_LLM/FinalRunPack/_ARTIFACT_INDEX.md`.

## Method (shared)
All cells recomputed from RAW judgment rows with independent code; no intermediate summary trusted (compared only). **Consolidation:** strict both-orders — a pair is decisive only if BOTH presentation orders (order 0 and order 1) name the same side; otherwise tie. **Win-rate (fwr):** `(F + 0.5*T)/n` (ties count half). **Panel:** per-judge strict both-orders consolidation, then majority vote across judges present, with a ≥2-judge quorum. **3-judge-only** variant restricts to cells where all 3 judges produced a complete (both-orders) verdict.

## Raw inputs (all under `C:/Research/Train_LLM/FinalRunPack/frontier_pilot/`)
- `judgments_deepseek_v2.jsonl` — v2 single-judge (gemini), generation @4096 tokens
- `judgments_deepseek_v2_panel.jsonl` — v2 3-judge panel (gemini/mistral/kimi)
- `judgments.jsonl` — v1 single-judge (contains `deepseek:rag__vs__*`, @512 cap)
- `judgments_panel.jsonl` — v1 panel (contains `deepseek:rag__vs__*`, @512 cap)
- `answers_deepseek_rag.jsonl` — v1 deepseek answer texts + `output_tokens` (@512 cap)
- `answers_deepseek_rag_v2.jsonl` — v2 deepseek answer texts + `output_tokens` (@4096)

---

## Claims

| # | Claim (published) | Published | Recomputed | Verdict |
|---|---|---|---|---|
| 1 | Single-judge v2 vs 8B: F/L/T, fwr | 51/9/285, .5609 | 51/9/285 (n=345), .5609 | MATCH |
| 2 | Single-judge v2 vs 1B: F/L/T, fwr | 133/12/200, .6754 | 133/12/200 (n=345), .6754 | MATCH |
| 3 | Panel v2 vs 8B: F/L/T, fwr | 69/32/244, .5536 | 69/32/244 (n=345), .5536 | MATCH |
| 4 | Panel v2 vs 1B: F/L/T, fwr | 164/22/159, .7058 | 164/22/159 (n=345), .7058 | MATCH |
| 5 | Δfwr single 8B (v2 − v1 strict) | +.016 | +.0160 (v1 .5449→v2 .5609) | MATCH |
| 6 | Δfwr single 1B | +.011 | +.0107 (v1 .6647→v2 .6754) | MATCH |
| 7 | Δfwr panel 8B | +.018 | +.0175 (v1 .5361→v2 .5536) | MATCH |
| 8 | Δfwr panel 1B | +.036 | +.0357 (v1 .6701→v2 .7058) | MATCH |
| 9 | v1 decisive local wins vs 8B | 24 | 24 | MATCH |
| 10 | of those, dissolved in v2 | 16 | 16 | MATCH |
| 11 | dissolved breakdown | 15 tie, 1 frontier | 15 tie, 1 frontier | MATCH |
| 12 | exact-512-token v1 answers (global) | 41 | 41 | MATCH |
| 13 | dissolved that are provable 512-trunc / CoT-leak artifacts | ≥14 | 14 (all exact-512 truncations) | MATCH |
| 14 | v2 cap hits @4096 | 0 | 0 (max output = 1984 tok) | MATCH |
| 15 | v2 CoT leaks flagged @4096 | 0 | 0 | MATCH |
| 16 | Panel attrition (3-judge / 2-judge / dropped) | 664 / 26 / 0 | 664 / 26 / 0 | MATCH |
| 17 | 3-judge-only fwr shift | ≤0.003 | 8B .0019, 1B .0027 (both ≤.003) | MATCH |

**Overall verdict: all 17 claims MATCH.** Recompute script reports 28/28 sub-checks PASS, exit 0.

---

## Per-claim detail

**Claims 1–4 (v2 cells).** Grouped raw v2 judgment rows by (matchup, qid), applied strict both-orders consolidation per qid, tallied. Single-judge cells use `judgments_deepseek_v2.jsonl`; panel cells use `judgments_deepseek_v2_panel.jsonl` with per-judge strict→majority (≥2-judge quorum). All four cells reproduce the published F/L/T and fwr exactly (each n=345; no v2 attrition).

**Claims 5–8 (deltas vs v1).** v1 baseline recomputed independently from `judgments.jsonl` (`deepseek:rag__vs__llama8b:rag` 54/24/256, n=334, fwr .5449; `deepseek:rag__vs__minicpm1b:rag` 127/17/190, n=334, fwr .6647) and `judgments_panel.jsonl` (8B 35/21/138, n=194, fwr .5361; 1B 82/16/96, n=194, fwr .6701). Deltas to 3 decimals: +.0160 / +.0107 / +.0175 / +.0357, rounding to the published +.016 / +.011 / +.018 / +.036.

**Claims 9–11 (answer-level fix).** Consolidated v1 `deepseek:rag__vs__llama8b:rag` → 24 decisive local wins (qids listed below). Looked up each in the v2 consolidated `deepseek:rag_v2__vs__llama8b:rag` map: 8 remain local, 15 become tie, 1 becomes frontier → 16 dissolved, breakdown 15 tie + 1 frontier. The single frontier flip is q0250; q0019/q0042/q0098/q0178/q0180/q0219/q0221/q0242 remain local.

**Claim 12 (512-token cap).** v1 `answers_deepseek_rag.jsonl` `output_tokens` field: exactly 41 answers report 512 completion tokens (the v1 generation cap). v2 `answers_deepseek_rag_v2.jsonl`: 0 at the 4096 cap, max 1984.

**Claim 13 (provable artifacts).** Of the 16 dissolved qids, 14 had v1 answers truncated at exactly 512 tokens (mid-reasoning cut-off; tails such as "Thus, the answer:", "I'll craft", "So perhaps", "roughly half of the…" with no terminal sentence). These are provable truncation/CoT-leak artifacts: the v1 model's response was severed mid-chain-of-thought by the 512 cap, which v2's 4096 cap removes. The remaining 2 dissolved qids (q0001 @45 tok, q0080 @347 tok) are non-truncated genuine verdict shifts. Count of provable artifacts = 14, satisfying ≥14.
- Exact-512 dissolved qids: q0004, q0031, q0063, q0096, q0196, q0215, q0232, q0235, q0240, q0241, q0250, q0253, q0262, q0266.

**Claims 14–15 (v2 generation health).** `answers_deepseek_rag_v2.jsonl`: no answer reaches `max_tokens` (4096); `cot_leak_stripped == True` count = 0. Matches the published "0 cap hits and 0 CoT leaks at 4096."

**Claim 16 (panel attrition).** Over the 690 v2 panel cells (345 qids × 2 matchups), per-cell complete-judge counts: 664 cells have all 3 judges, 26 have exactly 2, 0 fall below quorum (dropped). Attrition is immaterial.

**Claim 17 (3-judge-only sensitivity).** Restricting to cells with all 3 judges complete: vs 8B fwr .5556 (n=333) vs .5536 full → shift .0019; vs 1B fwr .7085 (n=331) vs .7058 full → shift .0027. Both ≤0.003.

---

## Notes / issues
- "Provable artifacts ≥14" is met exactly at 14 (all 14 are exact-512-token truncations). The receipt's CoT-leak heuristic finds 0 explicit `<think>` tags in v1 texts; the truncated tails are CoT-style mid-reasoning leaks cut by the cap, so the truncation count alone carries the claim. No headroom above 14 — if one of the 14 were reclassified the claim would fail, but the 512-token cap evidence is unambiguous.
- All published numbers in `summary_deepseek_v2.json` reconcile with the independent recompute (cross-checked in the script). No CORRECTED, UNVERIFIABLE, or NEW findings.
