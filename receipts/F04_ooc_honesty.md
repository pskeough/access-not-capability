# F04 — OOC honesty / calibration receipt (GOLD-STANDARD)

Recompute script: `C:/Research/Train_LLM/Final_Validation/scripts/F04_ooc_honesty.py`
(offline, idempotent, no API calls; **runs clean, exit 0, 32/32 checks PASS**).

## Scope of the claim (published interpretation, verified)
01_FINDINGS §4 reports out-of-corpus (OOC) win-rates for the three honesty-relevant headline
pairs, pooled over 3 models (llama8b, minicpm1b, phi4mini). The OOC gold answer is a clean
abstention. The doc explicitly scopes the finding: *lexical abstention frequency is similar
(Llama OOC: base 36/98 vs RAG 42/98); the effect is the **quality of the answer when the model
does not abstain**, not whether it abstains.* My recompute confirms the quality win-rates; it does
not re-derive the abstention-frequency spot-check (that lives in 02 §A / transcript).

## Method (2–3 lines)
For each (model, headline pair) I gather the grounded within-model judgment rows for that
unordered config pair, restrict to `question_type == "out_of_corpus"`, and consolidate per
`question_id` under the **strict both-orders** rule: result is `tie` if <2 rows OR any order is a
tie OR the two orders disagree; else the unanimous winner. Win-rate = (W + 0.5·T)/N with ties = ½.
Pooled = sum W/L/T across the 3 models (rows are NOT pooled by qid across models). This mirrors
`cross_corpus_stats.consolidate_within` / `stratified_analysis.consolidate_qtype`, re-implemented
independently from raw.

## Raw inputs (exact paths)
- Within-model grounded judgments (`judgments/grounded/pairwise_grounded.jsonl`) under
  `C:/Research/Train_LLM/experiments/v5_raft_cot/artifacts/`:
  - base: `runs/20260531-010503`, `runs_minicpm1b/20260531-111249`, `runs_phi4mini/20260601-122450`
  - qasper: `qasper/llama8b/20260603-090219`, `qasper/minicpm1b/20260603-153951`, `qasper/phi4mini/20260603-135816`
  - covidqa: `covidqa/llama8b/20260605-230052`, `covidqa/minicpm1b/20260606-120325`, `covidqa/phi4mini/20260606-105705`
- Eval sets (for OOC membership + unique-text clustering):
  base `experiments/v4_raft_cot/data/eval_set.jsonl`;
  qasper/covidqa `experiments/v5_raft_cot/corpora/{qasper,covidqa}/eval_set.jsonl`.
- Comparison only (never trusted as input): `C:/Research/Train_LLM/FinalRunPack/stratified_analysis.json`.

---

## CORE claims — 01_FINDINGS §4 table

Published table (01_FINDINGS.md, "## 4. Retrieval and honesty on unanswerable questions"):

| effect (out_of_corpus only) | base (n=156) | QASPER (n=294) |
|---|---|---|
| RAG-over-base  | **71%** | **22%** |
| RAG-over-LoRA  | 74% | 37% |
| LoRA-over-base | 48% | 33% |

| claim | published | recomputed (from raw) | verdict |
|---|---|---|---|
| base RAG-over-base OOC | 71%, n=156 | w71 l7 t78 n156, **wr 0.7051** | **MATCH** |
| QASPER RAG-over-base OOC | 22%, n=294 | w8 l172 t114 n294, **wr 0.2211** | **MATCH** |
| base RAG-over-LoRA OOC | 74% | w76 l0 t80 n156, **wr 0.7436** | **MATCH** |
| QASPER RAG-over-LoRA OOC | 37% | w27 l106 t161 n294, **wr 0.3656** | **MATCH** |
| base LoRA-over-base OOC | 48% | w23 l29 t104 n156, **wr 0.4808** | **MATCH** |
| QASPER LoRA-over-base OOC | 33% | w5 l104 t185 n294, **wr 0.3316** | **MATCH** |
| answerable retrieval helps (base/COVID 80–86%) | 80–86% | base 0.8339, covid 0.8310 | **MATCH** |
| answerable QASPER 60–61% | 60–61% | qasper 0.6093 | **MATCH** |

All endpoints reproduce exactly from raw. The published `stratified_analysis.json` is **not stale**:
I re-ran the original `stratified_analysis.py` against current raw and it reproduced the published
JSON byte-for-byte (0 differing keys), then restored the file unchanged.

### NAMING TRAP (documented, not an error)
"RAG-over-LoRA" in this project does **not** mean a literal `rag`-vs-`lora` comparison. The canonical
mapping (`cross_corpus_stats.EFFECT_LABEL`) defines it as **`lora_rag` vs `lora`** — i.e. the marginal
value of adding retrieval *on top of* LoRA (RAFT vs LoRA). A literal `rag`-vs-`lora` pair gives
different numbers (base 75% w80/l1/t75; QASPER 41% w34/l85/t175). The published 74%/37% are correct
under the documented `lora_rag`-vs-`lora` definition; my recompute uses that definition and matches.
The two other rows are unambiguous: RAG-over-base = (rag, base), LoRA-over-base = (lora, base).

---

## NEW (i) — per-model breakouts of the endpoints (win-rate, n per cell)

Each per-model OOC cell has n = #qids in that corpus (base 52, QASPER 98). Per-model W/L/T sum
exactly to the pooled totals (verified).

| corpus / effect | llama8b | minicpm1b | phi4mini | pooled |
|---|---|---|---|---|
| base / RAG-over-base | 0.7308 (52) | 0.6058 (52) | 0.7788 (52) | 0.7051 (156) |
| base / RAG-over-LoRA | 0.5769 (52) | 0.7596 (52) | 0.8942 (52) | 0.7436 (156) |
| base / LoRA-over-base | 0.7212 (52) | 0.3654 (52) | 0.3558 (52) | 0.4808 (156) |
| QASPER / RAG-over-base | 0.2653 (98) | 0.1531 (98) | 0.2449 (98) | 0.2211 (294) |
| QASPER / RAG-over-LoRA | 0.3469 (98) | 0.3469 (98) | 0.4031 (98) | 0.3656 (294) |
| QASPER / LoRA-over-base | 0.4286 (98) | 0.2296 (98) | 0.3367 (98) | 0.3316 (294) |

Verdict: **NEW** (not previously published per-model). All three models show the same qualitative
pattern at both endpoints: base RAG-over-base well above 0.5 for every model (0.61–0.78), QASPER
RAG-over-base well below 0.5 for every model (0.15–0.27). The double-edged-sword conclusion is **not
driven by a single model** — it holds 3/3.

## NEW (ii) — CIs clustered by UNIQUE QUESTION TEXT

The 156 base-OOC consolidated cells come from only 52 qids and only **27 unique normalized question
texts** (verified: multiplicities {1×13, 2×8, 3×3, 4×2, 6×1}); QASPER OOC has 98 qids / **94 unique
texts**. I build the text→cluster map myself from the eval set (`paraphrase_group_id` is null for all
OOC, so normalized-text identity is the clustering key) and bootstrap over **clusters** (10k resamples,
seed 12345), each cluster carrying all its (qid,model) outcomes. Cluster-bootstrap point estimate
equals the analytic win-rate in every cell (verified).

| corpus / effect | win-rate | 95% CI (cluster bootstrap) | clusters |
|---|---|---|---|
| base / RAG-over-base | 0.7051 | [0.5923, 0.8267] | 27 |
| base / RAG-over-LoRA | 0.7436 | [0.6875, 0.8095] | 27 |
| base / LoRA-over-base | 0.4808 | [0.4007, 0.5597] | 27 |
| QASPER / RAG-over-base | 0.2211 | [0.1854, 0.2571] | 94 |
| QASPER / RAG-over-LoRA | 0.3656 | [0.3265, 0.4050] | 94 |
| QASPER / LoRA-over-base | 0.3316 | [0.3010, 0.3628] | 94 |

Verdict: **NEW**. Honest caveat surfaced: the base OOC endpoint rests on only **27 independent
question texts** (52 qids × 3 models = 156 cells over-states the effective sample). The
text-clustered CI for base RAG-over-base ([0.59, 0.83]) is materially wider than a naive
cell-level CI would be, but still **excludes 0.5**, so "retrieval helps on base OOC" survives
clustering. The two endpoints are cleanly separated and non-overlapping (base [0.59,0.83] vs
QASPER [0.19,0.26]) — the corpus contrast is robust to the dependence structure. base
LoRA-over-base straddles 0.5 ([0.40, 0.56]), consistent with the published ≈48% null.

## NEW (iii) — answerable-vs-OOC contrast per corpus (RAG-over-base)

| corpus | answerable | OOC |
|---|---|---|
| base | 0.8339 (w599/l12/t268, n879) | 0.7051 (w71/l7/t78, n156) |
| covidqa | 0.8310 (w819/l28/t348, n1195) | — (0 OOC) |
| qasper | 0.6093 (w224/l26/t656, n906) | 0.2211 (w8/l172/t114, n294) |

Verdict: **NEW (contrast table)**. The contrast isolates the phenomenon: retrieval's value drops from
answerable→OOC on every corpus, but **only on heterogeneous QASPER does it cross below 0.5** (0.61 →
0.22, a 0.39 swing), flipping from helpful to actively harmful. On homogeneous base the drop is mild
(0.83 → 0.71, still helpful). COVID contributes nothing (0 OOC by construction), so the OOC finding is
QASPER-driven — exactly as the doc states.

---

## Scoped interpretation (verified, restated)
These are **answer-quality win-rates conditional on the grounded judge's preference**, not abstention
frequencies. On QASPER unanswerables, retrieval pulls irrelevant heterogeneous passages and the model
*engages* with them instead of abstaining cleanly; the judge prefers no-retrieval base's cleaner
abstention (base wins 78% of QASPER OOC decisive comparisons). The effect is about the *quality of
the non-abstention*, not whether the model abstains — consistent with the doc's honest scoping note
(abstention frequency is similar: Llama base 36/98 vs RAG 42/98, per 02 §A; not re-derived here).

## Comparison vs FinalRunPack/stratified_analysis.json
All six OOC cells (W/L/T/n/win_rate) **match** `stratified_analysis.json → by_qtype.{base,qasper}.*`
exactly. Re-running the original generator reproduces that JSON byte-for-byte from current raw, so the
published artifact is current, not stale. File restored unchanged after the verification re-run.

## Verdicts summary
- base 0.705 (n=156): **MATCH**
- QASPER 0.221 (n=294): **MATCH**
- RAG-over-LoRA 74/37 and LoRA-over-base 48/33: **MATCH** (under documented lora_rag-vs-lora def for RAG-over-LoRA)
- per-model breakouts; text-clustered CIs; answerable-vs-OOC contrast: **NEW**
