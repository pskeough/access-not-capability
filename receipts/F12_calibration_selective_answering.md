# F12 — Calibration / Selective-Answering (NEW family)

Recompute script: `C:/Research/Train_LLM/Final_Validation/scripts/F12_calibration_selective_answering.py`
(idempotent, offline, no API calls; **exit 0**, all hard claims PASS).
Companion JSON: `C:/Research/Train_LLM/Final_Validation/scratch/F12_selective_answering.json`.
Run log: `C:/Research/Train_LLM/Final_Validation/scratch/F12_run.log`.

Scope: within-model **llama8b** answers, configs `base / rag / lora / lora_rag`, on the
**BASE** corpus and **QASPER** corpus. COVID is excluded from the honesty matrix (0 OOC questions).

Raw inputs (verbatim paths):
- Answers: `experiments/v5_raft_cot/artifacts/runs/20260531-010503/answers/{base,rag,lora,lora_rag}.jsonl` (base corpus, llama8b),
  `experiments/v5_raft_cot/artifacts/qasper/llama8b/20260603-090219/answers/{base,rag,lora,lora_rag}.jsonl`,
  `experiments/v5_raft_cot/artifacts/covidqa/llama8b/20260605-230052/answers/*.jsonl` (sanity only).
- Eval sets (answerable vs OOC = empty `source_chunk_ids` ⇔ `question_type=="out_of_corpus"`, verified equal):
  base `experiments/v4_raft_cot/data/eval_set.jsonl`; qasper `experiments/v5_raft_cot/corpora/qasper/eval_set.jsonl`;
  covid `experiments/v5_raft_cot/corpora/covidqa/eval_set.jsonl`.
- `normalize_answer` imported from `scripts/smoke_eval_grounded.py` (the judge pipeline's CoT-scaffold stripper).
- Compared-against (not trusted): `FinalRunPack/01_FINDINGS.md` §4, `FinalRunPack/05_VERIFICATION_AUDIT.md` item 8,
  `FinalRunPack/artifacts/rouge_triangulation.json`.

---

## 0. Honest feasibility: ECE is NOT derivable

Classic confidence calibration (ECE / reliability diagrams / Brier) requires per-token logprobs or an
explicit scalar confidence. **No artifact records any such field.** The answer rows carry only
`{question_id, config, answer, retrieved_chunk_ids, latency_ms, input_tokens, output_tokens, answered_at}`.
**Verdict: UNVERIFIABLE / not-applicable — ECE cannot be computed and is not fabricated.** What IS
derivable without logprobs is **selective answering** using *abstention as the rejection signal*: the
honesty matrix, a discrete risk-coverage curve, and an engagement-quality proxy. Those are below.

## Abstention classifier (rejection signal) — documented verbatim

Three-way scheme `{pure_abstain, hedged, substantive}` from F05
(`Final_Validation/scripts/F05_frontier_closed_book.py`), with `ABSTAIN_SENT` **extended** for the
within-model (local 8B) phrasings the frontier-tuned F05 regex missed. Additions over F05:
`"the context/passage/… (provided) does not provide|contain|mention|address…"`,
`"there is no information|mention|data…"`, `"I couldn't/could not/didn't find (any) information…"`,
`"unfortunately, I (don't|couldn't)…"`, `"(not enough|insufficient|does not contain enough) information…"`.
`OFFER_SENT` and `HEDGE_BRIDGE` are unchanged from F05; thresholds `BRIDGE_MIN_WORDS=12`,
`PLAIN_MIN_WORDS=30`. A sentence abstains iff `ABSTAIN_SENT` matches; classes:
`substantive` = no abstention sentence; `hedged` = abstention + meaningful residual general-knowledge;
`pure_abstain` = abstention with no substantive residual. The exact regex is in the script.

**Primary "abstains" signal = `pure_abstain OR hedged`** (any decline / missing-context flag — the lexical
signal the published spot-check used). A **strict `pure_abstain`-only** variant is reported as sensitivity.
Manual audit of 18 answers spanning 2 corpora × 3 configs: all 18 labels correct by inspection
(e.g. RAG/QASPER OOC "BEA stands for Benchmarking Error-Analysis" → `substantive`/engage = a fabrication;
"The context does not provide … However it mentions …" → `hedged`/abstain).

---

## 1. Honesty matrix (2×2 per corpus × config) — NEW

Rows = abstain / engage; columns = OOC / answerable. Rates over llama8b answers (abstain = pa|hedged).

| corpus | config | correct-abstain (OOC) | false-engage (OOC) | false-abstain (answerable) |
|---|---|---|---|---|
| base | base | 32/52 = **61.5%** | 38.5% | 251/293 = 85.7% |
| base | rag | 52/52 = **100.0%** | 0.0% | 33/293 = 11.3% |
| base | lora | 48/52 = 92.3% | 7.7% | 272/293 = 92.8% |
| base | lora_rag | 52/52 = **100.0%** | 0.0% | 15/293 = 5.1% |
| qasper | base | 62/98 = **63.3%** | 36.7% | 173/302 = 57.3% |
| qasper | rag | 17/98 = **17.3%** | **82.7%** | 53/302 = 17.5% |
| qasper | lora | 45/98 = 45.9% | 54.1% | 97/302 = 32.1% |
| qasper | lora_rag | 15/98 = **15.3%** | **84.7%** | 14/302 = 4.6% |

Sensitivity (strict `pure_abstain` only): base-corpus OOC base 24/52, rag 49/52; QASPER OOC base 58/98,
rag 6/98 — same direction, larger gap. **Verdict: NEW.** Method: classify each normalized answer, count
abstain on OOC vs answerable per (corpus, config). Inputs: the answer + eval paths above.

## 2. Risk-coverage curve (OOC subset) — NEW

Coverage = fraction engaged; on OOC every engaged answer is an error (it should have abstained).
Sweep the rejection threshold over abstention bands. (point = coverage, i.e. n_engaged / n_OOC):

| corpus | config | reject_none | reject pure_abstain | reject pure_abstain+hedged |
|---|---|---|---|---|
| base | base | cov 1.000 (52) | cov 0.538 (28) | cov 0.385 (20) |
| base | rag | cov 1.000 (52) | cov 0.058 (3) | cov 0.000 (0) |
| base | lora_rag | cov 1.000 (52) | cov 0.000 (0) | cov 0.000 (0) |
| qasper | base | cov 1.000 (98) | cov 0.408 (40) | cov 0.367 (36) |
| qasper | rag | cov 1.000 (98) | cov **0.939** (92) | cov **0.827** (81) |
| qasper | lora_rag | cov 1.000 (98) | cov **0.867** (85) | cov **0.847** (83) |

Reading: at the same rejection threshold, base-corpus RAG drives OOC coverage (= error exposure) to ~0,
whereas QASPER RAG keeps coverage near 0.85–0.94 — i.e. it almost never self-rejects on unanswerable
QASPER questions. **Verdict: NEW.** Method: per OOC question, band the answer; coverage = kept fraction.

## 3. Engagement quality when engaging (ROUGE-L on non-abstained answerable) — NEW + cross-check

| corpus | config | n engaged | ROUGE-L (engaged) | ROUGE-L (all answerable) |
|---|---|---|---|---|
| base | base | 42 | 0.358 | **0.223** |
| base | rag | 260 | 0.494 | **0.480** |
| base | lora | 21 | 0.317 | **0.172** |
| base | lora_rag | 278 | 0.567 | **0.559** |
| qasper | base | 129 | 0.091 | 0.067 |
| qasper | rag | 249 | 0.098 | 0.091 |
| qasper | lora | 205 | 0.099 | 0.087 |
| qasper | lora_rag | 288 | 0.126 | 0.125 |

Cross-check: the **all-answerable** ROUGE-L column reproduces `rouge_triangulation.json` for base-llama8b
(`base 0.223 / rag 0.480 / lora 0.172 / lora_rag 0.559`) to ±0.0015 — **MATCH**, which validates the ROUGE
implementation and tokenization end-to-end. Published doc: `FinalRunPack/_ARTIFACT_INDEX.md` /
`FinalRunPack/artifacts/rouge_triangulation.json`. **Verdict (cross-check): MATCH; engaged-only ROUGE: NEW.**

---

## 4. Reconciliation of the published abstention spot-check

**Published (01_FINDINGS.md §4 footnote, line 117):** "lexical abstention *frequency* is similar
(Llama OOC: **base 36/98 vs RAG 42/98**)" → RAG abstains *slightly more*; effect framed as
"answer *quality* when engaging, not whether it abstains."
**Audit (05_VERIFICATION_AUDIT.md item 8):** calls the spot-check **unreproducible**; its rederivation
gave **11/98 vs 25/98** (same "RAG more" direction, 3× different magnitude).

**Recomputed (this receipt, QASPER OOC n=98, llama8b):**
- abstains (pa|hedged): **base 62/98 vs RAG 17/98**
- strict pure_abstain: **base 58/98 vs RAG 6/98**

| claim | published | recomputed | verdict |
|---|---|---|---|
| Llama QASPER-OOC abstention base-vs-RAG | 36/98 vs 42/98 ("RAG ≥ base; frequency similar") | 62/98 vs 17/98 (pa\|hedged); 58/98 vs 6/98 (pure) | **CORRECTED** |

**Both the published footnote AND the audit rederivation have the direction backwards.** Under every
defined classifier I tried, **RAG abstains far LESS than base on QASPER OOC** (it engages retrieved
heterogeneous passages — exactly the mechanism the paper invokes verbally). The "abstention frequency is
similar" framing is **not** supported: the frequency *drops sharply* with RAG on heterogeneous OOC.
The 36/42 and 11/25 magnitudes are both unreproducible (different classifier definitions); the safe,
reproducible statement is the **sign**. Note: the paper's *conclusion* (RAG harms OOC honesty on QASPER)
is correct and in fact *stronger* than stated — only the "frequency similar" hedge is wrong.

## 5. Verdict: does RAG shift abstention BEHAVIOR or only ENGAGEMENT QUALITY?

RAG-vs-base deltas (llama8b):
- **base corpus (homogeneous):** Δ correct-abstain(OOC) **+38.5pp**, Δ false-abstain(ANS) **−74.4pp**,
  Δ ROUGE(engaged) **+0.136**. RAG improves *both* honesty and quality.
- **QASPER (heterogeneous):** Δ correct-abstain(OOC) **−45.9pp**, Δ false-abstain(ANS) **−39.7pp**,
  Δ ROUGE(engaged) **+0.007**. RAG slashes abstention (engages everything) with ~zero engaged-quality gain.

**RAG shifts abstention BEHAVIOR, and the sign is corpus-homogeneity dependent** — it is *not* a
quality-only effect as 01_FINDINGS §4 scopes it. On the homogeneous base corpus, retrieved passages are
on-topic and let the model confirm absence → cleaner abstention (correct-abstain 61.5%→100%). On
heterogeneous QASPER, retrieved off-topic passages *induce engagement* → abstention collapses (63.3%→17.3%,
false-engage 82.7%) with no engaged-quality improvement (ROUGE +0.007). This both **corrects** the
"frequency similar" footnote and **reinforces** the §4 retrieval-honesty hazard, while adding the new,
load-bearing observation that the *behavioral* effect flips sign with corpus homogeneity.

---

## Per-claim verdict summary

| # | claim | published | recomputed | verdict |
|---|---|---|---|---|
| 0 | ECE / token-logprob calibration | (implied derivable in 03_PAPER_DIRECTIONS as future work) | no logprobs in any artifact | **UNVERIFIABLE** (not-applicable; documented) |
| 1 | Honesty matrix per corpus×config | not in paper | full 2×2 table (§1) | **NEW** |
| 2 | Risk-coverage curve (OOC) | not in paper | coverage points (§2) | **NEW** |
| 3a | ROUGE-L all-answerable (base-llama8b) | 0.223/0.480/0.172/0.559 (rouge_triangulation.json) | identical ±0.0015 | **MATCH** |
| 3b | ROUGE-L engaged-only | not in paper | §3 table | **NEW** |
| 4 | Llama QASPER-OOC abstention base-vs-RAG | 36/98 vs 42/98 ("similar / RAG more") | 62/98 vs 17/98 (RAG far less) | **CORRECTED** |
| 5 | RAG shifts only engagement quality, not abstention frequency | "frequency similar; quality only" (§4) | behavior shifts; sign flips by corpus homogeneity | **CORRECTED** |

All hard claims in the script PASS (exit 0): ECE-not-derivable, structural OOC/answerable counts
(base 52/293, qasper 98/302, covid 0 OOC), ROUGE cross-check ×4, QASPER-OOC "RAG abstains less"
(both signals), base-corpus "RAG abstains ≥ base".
