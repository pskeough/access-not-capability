# F10 — Confounds (STATISTICAL_SUPPLEMENT.md §S10) — GOLD-STANDARD receipt

**Validator:** adversarial recompute from RAW judgment/answer/eval artifacts with the validator's own
code (`Final_Validation/scripts/F10_confounds.py`). `confounds_audit.json` is compared against, never
trusted. Script runs offline, no API calls, **exit 0**, 30 checks: 27 MATCH / 3 CORRECTED / 0 FAIL.

**Published source for every row:** `FinalRunPack/STATISTICAL_SUPPLEMENT.md` §S10 (the table at lines
147–162), cross-referenced with `_ARTIFACT_INDEX.md` (claim→artifact map) and `05_VERIFICATION_AUDIT.md`
(frontier token method, §C item 9 / §E).

**Raw inputs (all corpus×model run dirs):**
- Grounded judgments: `<run>/judgments/grounded/pairwise_grounded.jsonl`
- Answer files (lengths, finish_reason, retrieved_chunk_ids, empties): `<run>/answers/{base,rag,lora,lora_rag}.jsonl`
- Eval sets (provenance + SHA-256): base `experiments/v4_raft_cot/data/eval_set.jsonl`; COVID/QASPER `experiments/v5_raft_cot/corpora/{covidqa,qasper}/eval_set.jsonl`
- Config snapshots (hash-lock): `<llama8b run>/config_snapshot.json` → `extras/eval_set/sha256`
- Chunks (frontier reconstruction): `rag/data/chunks.jsonl`
- Frontier GPT answers (API input_tokens): `FinalRunPack/frontier_pilot/answers_gpt_rag.jsonl`
- Prompt builders re-derived: `config/prompts/system_qa.txt`; user-prompt + RAG-context formatting reimplemented from `src/pipeline/run_answers.py` and `src/pipeline/rag_adapter.py`

Run dirs: base llama8b `runs/20260531-010503`, minicpm1b `runs_minicpm1b/20260531-111249`, phi4mini
`runs_phi4mini/20260601-122450`; covidqa `covidqa/{llama8b/20260605-230052, phi4mini/20260606-105705,
minicpm1b/20260606-120325}`; qasper `qasper/{llama8b/20260603-090219, phi4mini/20260603-135816,
minicpm1b/20260603-153951}` (all under `experiments/v5_raft_cot/artifacts/`).

**Consolidation rule (reimplemented):** strict both-orders — a (qid,pair) is decisive only if both
presentation orders name the same winner; otherwise tie; a single-order pair (e.g. q0367 rag/lora_rag in
qasper-minicpm) → tie. Win-rate counts ties as ½.

---

## Per-claim verdicts

### 1. Position bias — P(first-presented candidate wins | decisive), ALL grounded rows
Method: for each non-tie row, the first-presented config is `a_config` if `order==0` else `b_config`;
count first-position vs second-position wins, pooled over 3 models per corpus.

| corpus | published S10 | recomputed (current raw) | verdict |
|---|---|---|---|
| base | 0.490 | 0.490 (3436 / 3576) | MATCH |
| COVID | 0.497 | 0.4965 (4171 / 4229) | MATCH |
| QASPER | 0.496 | **0.4953 (2359 / 2404)** | CORRECTED |

QASPER: published/audit counts are 2329/2371 (4700 decisive); current raw has 2359/2404 (4763
decisive) — the audit was computed against an earlier QASPER snapshot with ~63 fewer decisive judgment
rows. Verdict ("≈0.50, clean, both-orders cancels") unchanged.

### 2. Verbosity bias — P(longer normalized answer wins | decisive headline)
Method: consolidate the 4 headline pairs answerable-only; for each decisive pair compare
`len(normalize_answer(winner))` vs `len(normalize_answer(loser))`. `normalize_answer` reimplemented in
the validator script (strip leaked Reasoning:/Answer: scaffold + `[n]` citations); the only imported
transform is `collapse_repetition` (raw library transform, not an analysis script). Robustness: with
`collapse_repetition` removed entirely, COVID P(longer)=0.4279 — still <0.5, so the conclusion does not
hinge on the import.

| corpus | published S10 | recomputed | verdict |
|---|---|---|---|
| base | 0.501 | 0.5009 (long 830 / short 827) | MATCH |
| COVID | 0.428 | 0.4286 (long 966 / short 1289) | MATCH |
| QASPER | 0.487 | **0.4842 (long 382 / short 407)** | CORRECTED |

COVID <0.5 confirms "judge favors shorter on COVID". QASPER CORRECTED for the same snapshot-drift reason.

### 3. Corpus-routing prefix check = 1.0 in all 18 rag/lora_rag cells; 0 empty retrievals
Method: across 9 model×corpus runs × {rag, lora_rag} = 18 cells, take every `retrieved_chunk_ids` id;
for COVID/QASPER require the `covidqa_`/`qasper_` prefix on all ids (frac=1.0); for base require NO id
carries a cross-corpus prefix; count cells with empty retrieval lists.
- **18/18 cells frac==1.0** → MATCH. **0 empty retrievals total** → MATCH.

### 4. Answer completeness — 400/config (base 345) × 0 empty, all 36 cells
Method: 9 runs × 4 configs = 36 cells; assert `len(answers)==eval_n` and count
`raw_content_empty` / blank answers.
- **36/36 cells n==eval_n** (base 345, COVID/QASPER 400) → MATCH. **0 empty answers** → MATCH.

### 5. MiniCPM truncation — finish_reason=='length', INCLUDING under-disclosed cells
Method: per cell count `finish_reason=='length'`.

| cell | published | recomputed | verdict |
|---|---|---|---|
| qasper minicpm lora | 78 | 78 | MATCH |
| qasper minicpm rag | 23 | 23 | MATCH |
| qasper minicpm lora_rag | 136 | 136 | MATCH |
| covid minicpm lora | 22 | 22 | MATCH |
| covid minicpm rag | 12 | 12 | MATCH |
| covid phi (lora+rag) | 2 | 2 (1+1) | MATCH |
| qasper llama (all configs) | 0 | 0 | MATCH |

All seven truncation counts the supplement lists (including the under-disclosed qasper-lora 78 /
rag 23 and covid-lora 22 / rag 12) reproduce exactly. LIMITATION disclosure is warranted as the
supplement states.

### 6. Tie-rate (answerable headline consolidated decisions)
Method: tie_n / total over the 4 headline pairs, answerable-only, pooled 3 models.

| corpus | published S10 | recomputed | verdict |
|---|---|---|---|
| base | 0.527 | 0.5273 | MATCH |
| COVID | 0.526 | 0.5255 | MATCH |
| QASPER | 0.782 | **0.7809** | CORRECTED |

QASPER low-power conclusion (≈0.78 ties → n=400 + pooling needed) unchanged.

### 7. Gold provenance
Method: tally `generated_by` over each eval set.
- base → `qwen2.5-14b-instruct-q4_k_m.gguf` (345) = LLM gold → MATCH
- COVID → `covidqa_human_annotators` (400) = human → MATCH
- QASPER → `qasper_human_annotators` (400) = human → MATCH

LIMITATION (base gold is LLM-generated; Rob's human gold pending) correctly disclosed.

### 8. Eval-set SHA-256 hash-locks vs config snapshot
Method: SHA-256 each eval_set.jsonl; compare to that run's `config_snapshot.json` →
`extras/eval_set/sha256`.

| corpus | snapshot lock | recomputed file SHA | verdict |
|---|---|---|---|
| base | b11170b6…d99 | b11170b6…d99 | MATCH |
| COVID | 54c4a3d1…729 | 54c4a3d1…729 | MATCH |
| QASPER | 26815e8b…c08 | 26815e8b…c08 | MATCH |

All three lock; base also matches the standalone `eval_set.sha256` sidecar. Hash-lock claim verified.

### 9. Frontier prompt symmetry — token-exact GPT verification (re-verified, not method-only)
Published method (`05_VERIFICATION_AUDIT.md` §C-9 / §E): "token-exact reconstruction for GPT 345/345,
delta=11 constant" — reconstruct the frontier prompt (sys + user, with the SAME retrieved chunks the
llama8b rag run used) and confirm the API-reported `input_tokens` exceeds the o200k_base token count by
a CONSTANT 11 across all questions (the 11 = chat-template wrapper the API adds).

Re-verification (this validator): `tiktoken` with `o200k_base` is available, so I performed the check on
a 30-question sample. For each qid I rebuilt `system_qa.txt` + `build_user_prompt(question,
format_rag_context(retrieved chunks))` (both builders reimplemented from `run_answers.py` /
`rag_adapter.py`; retrieval ids taken verbatim from the llama8b rag run, chunk text from
`rag/data/chunks.jsonl`), tokenized with o200k_base, and subtracted from the API `input_tokens` in
`answers_gpt_rag.jsonl`.
- **delta set = {11} across all 30 questions** → the delta is constant and equals 11 → MATCH.

This confirms byte/token-identical prompt construction between the reconstructed pipeline and the
actual GPT API calls; the closed-book/RAG prompt symmetry claim ("byte-identical sys prompt +
retrieval, token-exact") is verified, not merely asserted.

---

## Cross-check vs `confounds_audit.json` (compared, not trusted)
Base and COVID position/verbosity/tie reproduce the audit JSON to 3–4 decimals. QASPER differs only
because the audit was built on an earlier snapshot (~63 fewer decisive rows; one row, q0367
rag/lora_rag order-1, is also genuinely missing from the current qasper-minicpm file → 4799/4800
lines, handled as a tie per the disclosed single-order rule).

## Bottom line
Every S10 row reproduces from raw. 27/30 exact MATCH; the 3 CORRECTED rows are QASPER
position/verbosity/tie, off only in the 3rd–4th decimal due to snapshot drift, with no change to any
published verdict ("clean" / "low power"). No fabricated or directionally-wrong number found.
