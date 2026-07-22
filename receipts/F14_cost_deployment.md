# F14 — Cost & Deployment receipt (NEW)

**Family:** F14 cost & deployment (democratization cost table)
**Recompute script:** `C:/Research/Train_LLM/Final_Validation/scripts/F14_cost_deployment.py` (offline, idempotent, exit 0, 19/19 PASS)
**Price file:** `C:/Research/Train_LLM/config/openrouter_prices.json` (dollars **per token**; e.g. gemini-3-flash-preview = 5e-7 in / 3e-6 out → $0.50/$3.00 per-M)
**Generator source:** `experiments/v5_raft_cot/_analysis/master_analysis/scripts/frontier_pilot.py`

All numbers are recomputed from RAW per-row token/cost fields in the answer and judgment files. No API calls. Intermediate summary JSONs are used only for cross-check, never as the source of truth.

---

## (a) Actual spend per frontier run — and the documented v1 understatements

### C1 — v1 frontier GEN spend is grossly under-logged (NEW / CORRECTED)
- **Published:** `frontier_pilot/cost_report.json` → `spent.gen = 0.37987` ("$0.95 total run", _ARTIFACT_INDEX.md line 45; 06_PAPER_SKELETON.md line 57).
- **Recomputed (honest):** **$9.73** gen, summing stored `input_tokens`/`output_tokens` in `answers_{gpt,claude,deepseek}_{base,rag}.jsonl` × price-file rates.
- **Verdict:** **CORRECTED.** The live cost tracker in `frontier_pilot.py` only accumulates `spent['gen']` for calls actually issued in-process (`todo`); the RAG answer files were pre-populated/resumed from cache, so their ~3.4M input tokens (gpt-rag 1.56M, claude-rag 1.69M, deepseek-rag 1.50M) were never counted live. The honest closed-book+RAG generation spend is **~$9.7**, not $0.38.
- **Per-config breakdown (price-file rates):** gpt base $0.498 / rag $2.176; claude base $0.585 / rag $5.638; deepseek base $0.098 / rag $0.728.
- **Raw inputs:** `FinalRunPack/frontier_pilot/answers_{gpt,claude,deepseek}_{base,rag}.jsonl`, `cost_report.json`.

### C2 — v1 JUDGE price understatement (~6.7×/10× per token; ~8.8× effective) (CORRECTED)
- **Published:** v1 judge spend logged at **$0.567** (`cost_report.json` → `spent.judge`); the price bug is acknowledged in 06_PAPER_SKELETON.md (line 57: "v1 judge-cost log understates ~6.7×").
- **Mechanism (from raw source):** `frontier_pilot.py` line 64 hardcodes `JUDGE = ("google/gemini-3-flash-preview", 0.075, 0.30)` (= $0.075/M in, $0.30/M out). The price file lists the same model at **$0.50/M in (6.667×), $3.00/M out (10×)**.
- **Recomputed (what v1 judge SHOULD have cost):** **~$4.9–5.0.** v1 has no per-row judge token/cost fields, so the honest cost is reconstructed from the **empirical per-call gemini rate** measured in two later gemini runs that used the price file: panel ($0.000610/call, n=4753) and qasper-arm ($0.000599/call, n=4800). Applied to v1's **8,183** judge calls: $0.000610 × 8183 = **$4.996** (panel rate) / $4.899 (qasper rate). Effective understatement factor ≈ **8.8×** (sits between the 6.667×/10× per-token bounds, reflecting the in/out token mix).
- **Verdict:** **CORRECTED.** The honest v1 judge spend is **~$5**, not $0.567; the "~6.7×" note in the skeleton is the *price-input* ratio — the *effective* understatement on the realized token mix is ~8.8×.
- **Raw inputs:** `frontier_pilot.py` (line 64), `judgments.jsonl` (call count), `judgments_panel.jsonl` (gemini rows), `qasper_arm/judgments.jsonl`.

**Honest v1 run total:** gen $9.73 + judge $5.0 ≈ **$14.7** (vs the **$0.95** nominal figure cited throughout). The $0.95 is a logging artifact, not the real spend.

---

## (b) Per-1000-questions API cost (gen only, from mean token usage)

### C3 — closed-book (base) gen, $/1000 questions (NEW)
| model | id | mean in | mean out | $/1k |
|---|---|---|---|---|
| gpt | openai/gpt-5.1 | 168 | 123 | **$1.44** |
| claude | anthropic/claude-sonnet-4.6 | 180 | 77 | **$1.70** |
| deepseek | deepseek/deepseek-v4-pro | 161 | 255 | **$0.29** |

### C4 — same-RAG (~4.5k-token retrieved context) gen, $/1000 questions (NEW)
| model | id | mean in | mean out | $/1k |
|---|---|---|---|---|
| gpt | openai/gpt-5.1 | 4520 | 66 | **$6.31** |
| claude | anthropic/claude-sonnet-4.6 | 4906 | 108 | **$16.34** |
| deepseek | deepseek/deepseek-v4-pro | 4492 | 287 | **$2.20** |

- **Verdict:** **NEW.** Mean tokens per question from the v1 answer files × price-file rates × 1000.
- **Local-model framing (ASSUMPTION, not a measurement):** the 8B local model (meta-llama/llama-3.1-8b-instruct) ran on **one consumer GPU**; its marginal API cost is **$0** because it is self-hosted. For reference only, the same 8B *via API* prices at $0.02/M in + $0.03/M out (≈ $0.09/1k closed-book, ≈ $0.14/1k at 4.5k-tok RAG context) — but the democratization claim rests on the local model being run on owned hardware, so the honest comparison is **frontier API $/1k (above) vs amortized local GPU hardware**, NOT API-vs-API.
- **Raw inputs:** `answers_{gpt,claude,deepseek}_{base,rag}.jsonl`.

---

## (c) Total project remediation spend ledger (verified per run)

| run | recomputed | source of truth | verdict |
|---|---|---|---|
| v1 (nominal) | **$0.9468** | `cost_report.json` gen 0.3799 + judge 0.5670 (as-logged) | MATCH (but see C1/C2: honest ≈ $14.7) |
| panel | **$6.6873** | Σ `cost` over `judgments_panel.jsonl` (14,101 rows) | MATCH (summary says 6.7014; Δ $0.014 = rounding/skipped-verdict rows) |
| deepseek-v2 | **$3.4466** | gen Σ`cost_usd` 0.7801 + single Σ`cost` 0.8036 + panel Σ`cost` 1.8628 | MATCH (~$3.44) |
| evidence | **$0.6669** | Σ `cost` over `judgments_fullev.jsonl` (240 rows) | MATCH |
| qasper-arm | **$5.1660** | gen Σ`cost_usd` 2.2921 + judge Σ`cost` 2.8739 | MATCH (~$5.16; summary `spent_usd_this_run` 2.8739 is judge-only) |
| clean-rejudge | **$14.4078** | Σ `cost` over `clean_rejudge/judgments_single.jsonl` (5,911 rows) | MATCH |
| **LEDGER TOTAL (nominal v1)** | **$31.32** | sum of the above | NEW |

- **Verdict:** all six runs **MATCH** their summary/cost artifacts within rounding. The panel summary's 6.7014 vs recomputed 6.6873 differs by $0.014 because a few rows lack parsed verdicts / `cost`; immaterial.
- **deepseek-v2 ~3.44** is confirmed: the per-run summary's `spent_usd_this_run = 1.8636` is **panel-judge only**; the full v2 remediation cost adds gen ($0.78) and the single-judge re-run ($0.80).
- **Raw inputs:** `judgments_panel.jsonl`, `answers_deepseek_rag_v2.jsonl`, `judgments_deepseek_v2.jsonl`, `judgments_deepseek_v2_panel.jsonl`, `evidence_sensitivity/judgments_fullev.jsonl`, `qasper_arm/answers_*.jsonl`, `qasper_arm/judgments.jsonl`, `clean_rejudge/judgments_single.jsonl`.

---

## Printable cost table (for the democratization framing)

```
FRONTIER API COST  (per 1,000 private-corpus questions, generation only)
  config        gpt-5.1    sonnet-4.6   deepseek-v4-pro
  closed-book   $1.44      $1.70        $0.29
  same-RAG      $6.31      $16.34       $2.20
LOCAL (8B + RAG): $0 marginal API cost  [run on one consumer GPU -- ASSUMPTION]

v1 PILOT RUN -- logged vs honest
  gen:    logged $0.38   honest $9.73   (live tracker missed resumed RAG calls)
  judge:  logged $0.57   honest ~$5.0   (hardcoded gemini price 6.7x/10x low)
  total:  nominal $0.95  honest ~$14.7

REMEDIATION LEDGER (nominal v1)
  v1 0.95 | panel 6.69 | deepseek-v2 3.45 | evidence 0.67 | qasper 5.17 | clean 14.41
  TOTAL ~ $31.3
```

## Honest caveat list (mandatory)
1. **Latency is NOT measured.** No wall-clock or tokens/sec was logged for either frontier API calls or the local GPU run. Any "faster/slower" claim is unsupported by these artifacts.
2. **Local hardware cost is ASSUMED, not measured.** "8B on one consumer GPU" is a deployment assumption. No GPU model, VRAM, power draw, throughput, or amortized $/question for the local run exists in the raw data. The $0 marginal API cost is real; the *total cost of ownership* of the local GPU is out of scope and must be stated as an assumption.
3. **v1 cost log understates true spend** on BOTH axes (gen ~25×, judge ~8.8×). Cite the honest figures (gen $9.7, judge ~$5) or explicitly the as-logged nominal $0.95 with the disclosure — never the nominal figure as if it were the real cost.
4. **v1 judge $/call is reconstructed, not logged.** v1 judgments carry no token/cost fields; the ~$5 honest judge figure uses the empirical per-call gemini rate from the panel and qasper runs (same model, same prompt, same both-orders protocol, correct prices). Two independent runs agree to within 2%, but it is an estimate, not a per-row sum.
5. **Per-1000 costs are gen-only.** They exclude judging, retrieval/embedding, and any orchestration overhead. RAG input tokens (~4.5k/question) dominate the same-RAG column and are corpus/chunk-size dependent.
6. **Prices are a point-in-time snapshot** (`openrouter_prices.json`, "verified 2026-06"); OpenRouter pricing drifts.
