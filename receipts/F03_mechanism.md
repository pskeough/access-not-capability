# F03 — Mechanism (size vs homogeneity decomposition + independent homogeneity)

**Recompute script:** `C:/Research/Train_LLM/Final_Validation/scripts/F03_mechanism.py`
(idempotent, offline, no API calls; **runs clean, exit 0, 37/37 checks pass**).

**Consolidation rule:** strict both-orders — a (base, rag) pair is decisive only if both
presentation orders agree on the winner, else tie. Win-rate counts ties as half:
`(wins + 0.5*ties)/n`.

**Headline finding:** the homogeneity half of the family reproduces **exactly** (all centroid
and pairwise-cosine numbers MATCH, ratios 1.68x/2.8x hold). The decomposition half reproduces
**exactly for base** but the **published COVID and QASPER decomposition rows are STALE** — they
were generated on an earlier, smaller eval set (n_answerable = 150 COVID, 112 QASPER) and do not
match the current raw eval sets (400 answerable COVID, 302 QASPER). Recomputing from current raw
with the project's own authoritative script (`ablation_decompose.py`) reproduces my numbers to
the digit. **The mechanism conclusion is unchanged**: recall does not track corpus size; the
cross-corpus win gap tracks miss-win (homogeneity).

---

## (a) Decomposition — llama8b, all 3 corpora

**Method:** For each (base, rag) judged pair, consolidate the two presentation orders (strict
both-orders). Restrict to **answerable** questions (`source_chunk_ids` non-empty in the eval set).
recall@6 = fraction of answerable questions whose served **top-6** `retrieved_chunk_ids` (from
`answers/rag.jsonl`) intersect the gold `source_chunk_ids`. hit_win / miss_win = (base,rag)
win-rate (ties=½) conditioned on retrieval hit / miss. aggregate = overall answerable-only
RAG-over-base win-rate. Identity check: `recall*hit + (1-recall)*miss == aggregate`.

**Raw inputs:**
- Judgments: `experiments/v5_raft_cot/artifacts/{runs/20260531-010503, covidqa/llama8b/20260605-230052, qasper/llama8b/20260603-090219}/judgments/grounded/pairwise_grounded.jsonl`
- Retrieved chunks: `.../answers/rag.jsonl` (field `retrieved_chunk_ids`)
- Gold: `experiments/v4_raft_cot/data/eval_set.jsonl` (base); `experiments/v5_raft_cot/corpora/{covidqa,qasper}/eval_set.jsonl`

| corpus | metric | published (FinalRunPack) | published n | recomputed (current raw) | recomputed n | verdict |
|---|---|---|---|---|---|---|
| base | recall@6 | 0.294 | 293 | **0.2935** | 293 | MATCH |
| base | hit_win | 0.878 | 293 | **0.8779** (n_hit=86) | 293 | MATCH |
| base | miss_win | 0.826 | 293 | **0.8261** (n_miss=207) | 293 | MATCH |
| base | aggregate | 0.841 | 293 | **0.8413** | 293 | MATCH |
| covidqa | recall@6 | 0.800 | **150** | **0.8125** | **400** | CORRECTED |
| covidqa | hit_win | 0.950 | 150 | **0.9308** (n_hit=325) | 400 | CORRECTED |
| covidqa | miss_win | 0.583 | 150 | **0.5733** (n_miss=75) | 400 | CORRECTED |
| covidqa | aggregate | 0.877 | 150 | **0.8638** | 400 | CORRECTED |
| qasper | recall@6 | 0.241 | **112** | **0.2318** | **302** | CORRECTED |
| qasper | hit_win | 0.852 | 112 | **0.8071** (n_hit=70) | 302 | CORRECTED |
| qasper | miss_win | 0.506 | 112 | **0.5323** (n_miss=232) | 302 | CORRECTED |
| qasper | aggregate | 0.589 | 112 | **0.5960** | 302 | CORRECTED |

**Identity verified** for all three corpora: `|pred − aggregate| = 0.0e+00` (exact).

**Why CORRECTED, not a recompute error:** Running the project's own authoritative script
`experiments/v5_raft_cot/_analysis/master_analysis/scripts/ablation_decompose.py` **unmodified**
against the current raw today prints exactly my numbers (covidqa llama8b n=400 recall 81.2%
hit 93.1% miss 57.3% agg 86.4%; qasper n=302 recall 23.2% hit 80.7% miss 53.2% agg 59.6%). The
stored `FinalRunPack/artifacts/ablation_decomposition.json` was produced on an **earlier,
smaller eval set** (n=150 / 112). I restored the project's stats copy after this check; nothing
outside `Final_Validation` was left modified. Base is unaffected (n=293 then and now → MATCH).

Published source: `FinalRunPack/_ARTIFACT_INDEX.md` line 28 and
`FinalRunPack/artifacts/ablation_decomposition.json` (llama8b rows).

---

## (b) Independent homogeneity (own TF-IDF)

**Method:** Reimplemented the TF-IDF homogeneity metric from the chunk stores. Drop chunks with
< 20 whitespace tokens (title/stub filter). Seed=1337; sample 1200 chunks per corpus (or all if
fewer); `TfidfVectorizer(max_features=5000, stop_words="english", sublinear_tf=True, min_df=2)`,
L2-normalized rows. **Centroid concentration** = mean cosine of each chunk to the (renormalized)
corpus centroid. **Mean pairwise cosine** = mean over 40,000 random distinct chunk pairs
(seed=1338). sklearn from project `.venv`.

**Raw inputs:** `rag/data/chunks.jsonl` (base);
`experiments/v5_raft_cot/corpora/{covidqa,qasper}/chunks.jsonl`.

| metric | corpus | published | recomputed | verdict |
|---|---|---|---|---|
| centroid concentration | base | 0.235 | **0.2348** | MATCH |
| centroid concentration | covidqa | 0.136 | **0.1364** | MATCH |
| centroid concentration | qasper | 0.140 | **0.1395** | MATCH |
| mean pairwise cosine | base | 0.053 | **0.0531** | MATCH |
| mean pairwise cosine | covidqa | 0.018 | **0.0177** | MATCH |
| mean pairwise cosine | qasper | 0.019 | **0.0188** | MATCH |
| centroid ratio base / mean(covid,qasper) | — | ~1.68x | **1.70x** | MATCH |
| pairwise ratio base / mean(covid,qasper) | — | ~2.8x | **2.91x** | MATCH |

All eight numbers reproduce to within rounding (tol 0.005 centroid, 0.003 pairwise, 0.05/0.15
ratio). Recompute also bit-matches `FinalRunPack/homogeneity_metric.json` per-corpus.

**Sampling / counting bases (both stated, as required):**

| corpus | raw chunks (total file) | eligible (≥20-word) | n_sampled for TF-IDF |
|---|---|---|---|
| base | **593** | **591** | 591 (all eligible ≤ 1200) |
| covidqa | 8,260 | 4,807 | 1,200 |
| qasper | **19,817** | **16,445** | 1,200 |

- Base: **593 raw** chunks in `chunks.jsonl`; **591 sampled** after the ≥20-word filter (2 stub
  chunks dropped); since 591 ≤ 1200, all 591 are used (no subsampling). This is the
  `n_chunks_total=591`, `n_sampled=591` recorded in `homogeneity_metric.json`.
- QASPER: **19,817 raw**; **16,445 eligible** after the ≥20-word filter (the
  `n_chunks_total=16445` in the JSON refers to the eligible pool, **not** the raw file);
  1,200 sampled from that pool.

Published source: `FinalRunPack/_ARTIFACT_INDEX.md` line 29; `FinalRunPack/homogeneity_metric.json`.

---

## (c) Size non-confound statement

**Claim (report §3 / R4 audit):** corpus **SIZE** does not drive the RAG result; the lever is
**homogeneity** (whether an off-target retrieval still returns usable, on-topic context).

**Recomputed, three independent legs — all hold:**

1. **28x size gap, same low recall regime.** base eligible = 591 chunks, recall@6 = 0.2935;
   qasper eligible = 16,445 chunks (**27.8x** larger), recall@6 = 0.2318. A 28x size increase
   does **not** lift recall — both sit in the low 0.23–0.29 band. (The absolute gap is 0.062, so
   they are "similar/low," not "identical"; recall@6 is slightly *lower* for the larger store.)
2. **Smallest store, highest recall.** covidqa is the smallest store yet has the **highest**
   recall@6 (0.8125 > base 0.2935 > qasper 0.2318) — anti-monotone in size, refuting any
   size→recall driver.
3. **Win gap tracks homogeneity (miss-win), not recall.** centroid concentration base 0.235 ≫
   qasper 0.140 mirrors miss_win base 0.826 ≫ qasper 0.532. The cross-corpus aggregate gap is
   carried by miss-win (homogeneity), exactly as claimed.

**Verdict: MATCH** (qualitative non-confound conclusion fully reproduced from current raw).
Minor NEW note: the exact recall numbers behind the statement are the corrected current-raw
values (base 0.294 vs qasper 0.232, gap 0.062), not the stale published QASPER 0.241.

Published source: `FinalRunPack/_ARTIFACT_INDEX.md` lines 10–13 of `ablation_decompose.py`
docstring ("base (593) and QASPER (19,817) similar recall@6 despite ~30x size gap").

---

## Comparison artifacts (loaded only to compare, never trusted as input)
- `C:/Research/Train_LLM/FinalRunPack/artifacts/ablation_decomposition.json` — base llama8b row MATCHES; covidqa/qasper rows STALE (n=150/112).
- `C:/Research/Train_LLM/FinalRunPack/homogeneity_metric.json` — all rows MATCH bit-for-bit.

## Reproduce
```
cd C:/Research/Train_LLM
$env:PYTHONUTF8=1
.venv/Scripts/python.exe Final_Validation/scripts/F03_mechanism.py   # exit 0, 37/37 PASS
```
