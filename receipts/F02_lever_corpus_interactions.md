# F02 — Lever × corpus-pair interaction G-tests (GOLD-STANDARD receipt)

**Family:** F02 (interactions). **Status:** all published claims MATCH; two NEW analyses added.
**Recompute script:** `C:/Research/Train_LLM/Final_Validation/scripts/F02_lever_corpus_interactions.py` (offline, idempotent, exit 0, 21/21 checks PASS).

## Method (2–3 lines)
Re-read the 9 RAW `pairwise_grounded.jsonl` files (3 models × 3 corpora). Re-consolidated each lever with the **strict both-orders** rule (decisive only if both presentation orders name the same winner; else tie), **answerable-only** (drop `question_type=="out_of_corpus"`), then **pooled decisive W/L across the 3 models** per (lever, corpus). For each lever × corpus-pair I recomputed the likelihood-ratio G-test on the 2×2 decisive table (G = 2·Σ O·ln(O/E), df=1, p = χ²₁ survival). I did **not** trust the intermediate JSON; I compared against it as a final cross-check (all 12 G match to <1e-2).

## Raw inputs (exact paths)
- `experiments/v5_raft_cot/artifacts/runs/20260531-010503/judgments/grounded/pairwise_grounded.jsonl` (base/llama8b, 345 qids)
- `experiments/v5_raft_cot/artifacts/runs_minicpm1b/20260531-111249/.../pairwise_grounded.jsonl` (base/minicpm1b, 345)
- `experiments/v5_raft_cot/artifacts/runs_phi4mini/20260601-122450/.../pairwise_grounded.jsonl` (base/phi4mini, 345)
- `experiments/v5_raft_cot/artifacts/covidqa/{llama8b/20260605-230052, phi4mini/20260606-105705, minicpm1b/20260606-120325}/.../pairwise_grounded.jsonl` (400/400/395 qids)
- `experiments/v5_raft_cot/artifacts/qasper/{llama8b/20260603-090219, phi4mini/20260603-135816, minicpm1b/20260603-153951}/.../pairwise_grounded.jsonl` (400/400/400 qids)

Published values cited from `FinalRunPack/_ARTIFACT_INDEX.md` (§ "Key claims", lines 26–27) and `FinalRunPack/artifacts/cross_corpus_stats.json` → `interaction`.

## Claim-by-claim verdicts

| # | Claim (published) | Published | Recomputed (raw) | Verdict |
|---|---|---|---|---|
| 1 | RAG-over-base base-vs-QASPER interaction | G=26.5, p=2.7e-7 | G=26.477, p=2.67e-07 (base dec [599,12], qasper [224,26]) | **MATCH** |
| 2 | RAG-over-base COVID-vs-QASPER interaction | G=17.6, p=2.7e-5 | G=17.635, p=2.68e-05 (covid [819,28]) | **MATCH** |
| 3 | RAG-over-base base-vs-COVID not significant | ns | G=2.485, p=0.115 | **MATCH** |
| 4 | RAG-over-LoRA base-vs-QASPER interaction | G=39.8 | G=39.796, p=2.82e-10 (base [614,26], qasper [237,48]) | **MATCH** |
| 5 | RAG-over-LoRA COVID-vs-QASPER interaction | G=38.7 | G=38.729, p=4.87e-10 (covid [821,40]) | **MATCH** |
| 6 | RAG-over-LoRA base-vs-COVID not significant | ns | G=0.299, p=0.584 | **MATCH** |
| 7 | LoRA-over-base — all corpus-pairs ns | p>0.09 | min p=0.091 (base-vs-QASPER G=2.850); others 0.613, 0.152 | **MATCH** |
| 8 | RAFT-over-RAG — all corpus-pairs ns | p>0.09 | min p=0.100 (base-vs-COVID G=2.706); others 0.163, 0.979 | **MATCH** |

All 12 pooled G statistics also cross-check against `cross_corpus_stats.json` to <1e-2 (mine vs published identical to 4 dp).

## NEW analyses

### (B) Per-model G-tests — base-vs-QASPER retrieval interactions
Does each of the 3 models *individually* show the base-vs-QASPER retrieval interaction? Pooled decisive W/L split by model, same G-test.

| Lever | model | base dec [W,L] | qasper dec [W,L] | G | p | sig |
|---|---|---|---|---|---|---|
| RAG-over-base | llama8b | [205,5] | [77,19] | 25.475 | 4.48e-07 | **SIG** |
| RAG-over-base | minicpm1b | [184,1] | [74,0] | 0.674 | 0.411 | ns |
| RAG-over-base | phi4mini | [210,6] | [73,7] | 4.373 | 0.0365 | **SIG** |
| RAG-over-LoRA | llama8b | [221,4] | [80,24] | 38.993 | 4.25e-10 | **SIG** |
| RAG-over-LoRA | minicpm1b | [192,7] | [82,12] | 8.284 | 0.0040 | **SIG** |
| RAG-over-LoRA | phi4mini | [201,15] | [75,12] | 3.328 | 0.0681 | ns |

**Verdict: NEW.** The pooled base-vs-QASPER RAG interaction is **not** uniform across models. It is dominated by **Llama8b** (G≈25–39) and partly **Phi4mini**. **MiniCPM1b shows no significant RAG-over-base interaction** (G=0.67) because it is saturated — it loses to RAG essentially never in either corpus (1 and 0 losses), so there is no room for a corpus difference. The interaction is significant per-model in 2/3 models for both retrieval levers, but in *different* models (RAG-over-base: llama+phi; RAG-over-LoRA: llama+minicpm). This is a real qualifier on the pooled claim and worth stating in the paper.

### (C) Question-level block permutation — pooled base-vs-QASPER RAG-over-base
Addresses the shared/pooled-question critique. Unit of permutation = a (corpus, question_id) **block** carrying that question's per-model decisive outcomes; the corpus label is permuted across the 380 question-blocks (base=250, qasper=130) while each question's across-model outcomes stay together, preserving within-question across-model correlation under the null. Statistic = pooled-table G. Seed 20260611, 2000 perms, add-one (Phipson–Smyth) p.

- G_obs = 26.477; #(G_perm ≥ G_obs) = **0/2000**; **perm_p = 0.00050**.

**Verdict: NEW.** The asymptotic G p=2.7e-7 is corroborated by a question-level permutation p≈5e-4 (floor of the 2000-perm resolution). The interaction is **not** an artifact of treating correlated per-question/per-model judgments as independent. (perm_p is bounded below by 1/(n_perm+1); the asymptotic test is the more precise estimate, the permutation only certifies robustness to the pooling assumption.)

## Notes / caveats
- covidqa/minicpm1b has 395/400 qids (5 short) — this matches the canonical pipeline, which flags <95% as incomplete; 395 is ≥95% so it is **included**, consistent with the published pooled counts (covid RAG-over-base [819,28] reproduces exactly).
- "base" run dir used is `runs/20260531-010503` (per the task header and the canonical `RUN_DIR`), matching published base decisive totals.
- Consolidation semantics verified against raw: `winner ∈ {config_name, "tie"}`; a lever is decisive only on unanimous both-orders agreement.
