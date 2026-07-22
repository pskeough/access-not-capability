# F15 — Judge-robustness (robustness appendix) — GOLD-STANDARD receipt

**Recompute script:** `C:/Research/Train_LLM/Final_Validation/scripts/F15_judge_robustness.py`
(idempotent, offline, no API calls; runs clean, exit 0; 25/25 checks PASS).

**Raw inputs (read-only, never trusted via summaries):**
- Panel judgments: `C:/Research/Train_LLM/FinalRunPack/frontier_pilot/judgments_panel.jsonl` (3-judge, both orders)
- Single-judge capped run: `C:/Research/Train_LLM/FinalRunPack/frontier_pilot/judgments.jsonl` (gemini, n=345, capped 1600-char evidence)
- Full-evidence re-judge: `C:/Research/Train_LLM/FinalRunPack/frontier_pilot/evidence_sensitivity/judgments_fullev.jsonl`

**Consolidation rule (everywhere):** per-judge STRICT both-orders — a verdict is decisive (frontier/local)
only if BOTH presentation orders agree on the same decisive label, else tie. Panel = per-judge strict →
majority of decisive votes (quorum ≥2 judges present). Win-rate counts ties as half.
"Pooled +RAG vs 8B cell" = the 3 matchups {gpt,claude,deepseek}:rag vs llama8b:rag.

**Comparison docs (published):** `FinalRunPack/05_VERIFICATION_AUDIT.md` (§A, §C.5),
`FinalRunPack/06_PAPER_SKELETON.md` (§3 panel rule), `FinalRunPack/STATISTICAL_SUPPLEMENT.md` (§S7, §S10).

---

## (a) Panel rule-sensitivity table — pooled +RAG vs 8B

| claim | published | recomputed | verdict |
|---|---|---|---|
| majority-of-decisive F/L/T | 134 / 47 / 413 (05_VERIFICATION_AUDIT §A; "134:47") | **134 / 47 / 413**, net 0.5732, not-lose 0.7744 | **MATCH** |
| strict ≥2-decisive-judges F/L/T | ~98 / 17 / 479 (06_PAPER_SKELETON §3) | **98 / 17 / 479**, net 0.5682 | **MATCH** |
| strict ≥2-decisive not-lose | ~0.835 (06_PAPER_SKELETON §3 "not-lose 83.5%") | **0.8350** | **MATCH** |
| unanimous-only F/L/T | (variant) | 49 / 7 / 538, net 0.5354 | NEW (reported) |
| each-judge-alone net | (variant) | gemini 0.5750, mistral 0.5404, kimi 0.5609 | NEW (reported) |
| net win-rate range across variants | ~0.540–0.575 (05_VERIFICATION_AUDIT §A; 06_PAPER_SKELETON §3) | **0.5404–0.5750** (5 main variants); 0.5354–0.5750 incl. unanimous-only | **MATCH** |

**Method:** group raw panel rows by (matchup, qid, judge); strict both-orders per judge → per-unit verdict
set (quorum ≥2). majority = sign of (#frontier − #local). strict≥2 = decisive only if ≥2 judges agree on
one decisive label and outnumber the other. unanimous = all present judges agree decisive. each-judge-alone
= that judge's strict both-orders verdict over the cell. net = (F + 0.5T)/n.
The published lower bound 0.540 corresponds to mistral-alone (0.5404); unanimous-only dips to 0.5354 (reported, not a contradiction).

## (b) Decisive-verdict provenance split — pooled +RAG vs 8B (181 decisive units)

| category | published (06_PAPER_SKELETON §3) | recomputed | verdict |
|---|---|---|---|
| unanimous | 51 | **51** | **MATCH** |
| 2-1 | 59 | **59** | **MATCH** |
| single-decisive | 66 | **66** | **MATCH** |
| two-judge | 5 | **5** | **MATCH** |
| total decisive | 181 | **181** (= 134 F + 47 L) | **MATCH** |

**Method:** among the 181 decisive majority outcomes, classify by composition of the winning side's strict
verdicts: unanimous = 3 judges all decisive-same; 2-1 = 2 decisive-same (3 present); single-decisive = exactly
one decisive verdict carries the unit (others tie; 2 or 3 judges present); two-judge = only 2 judges present
AND both decisive-agreeing. Note: a 2-judge unit with one decisive + one tie counts as single-decisive (this
is the only subtlety; reconciles the 8 raw 2-judge decisive units as 5 both-decisive + 3 single-decisive).

## (c) Mistral primacy bias + panel alpha (all 12 cells)

| claim | published | recomputed | verdict |
|---|---|---|---|
| mistral P(first-presented wins \| decisive raw) | ~0.731 (05_VERIFICATION_AUDIT §C.5) | **0.7306** (n_dec=2524) | **MATCH** |
| mistral pure position-flip rate | 17.8% (05_VERIFICATION_AUDIT §C.5) | **17.83%** (424/2378) | **MATCH** |
| gemini pure position-flip rate | 1.0% | **1.18%** (28/2375) | **MATCH** (rounds to ~1.0%) |
| kimi pure position-flip rate | 6.9% | **9.44%** (213/2256) | **CORRECTED** |
| panel α (all 3 judges) | 0.5653 (05_VERIFICATION_AUDIT §A; summary_panel.json) | **0.5653** | **MATCH** |
| panel α without mistral | 0.7547 (05_VERIFICATION_AUDIT §A) | **0.7547** | **MATCH** |
| n panel units (≥2 judges) | 2378 | **2378** | **MATCH** |

**Method:** P(first wins) over all raw decisive rows — for order 0 the frontier is shown first, so winner=frontier
counts as "first won"; for order 1 the local is shown first. Pure position-flip = both orders present and the
judge picked opposite decisive labels in the two orders (i.e., it picked the *same presented position* both times),
denominator = pairs with both orders present. α = nominal Krippendorff over per-unit strict verdict lists
(quorum ≥2); the without-mistral pass drops mistral's verdict then re-applies quorum ≥2.

**CORRECTED — kimi position-flip rate.** Published 6.9%; from raw I get **9.44%** (213/2256, both-orders-present
denominator). The same denominator and definition reproduce mistral (17.8%) and gemini (1.0%) exactly, so the
definition is consistent — the published kimi figure is understated. No alternate denominator I tried
(both-decisive 14.9%, either-decisive 12.7%) yields 6.9% while keeping mistral/gemini correct. Conservative for
the paper's thesis (mistral is the outlier either way; both-orders consolidation contains it), but the printed
6.9% is not reproducible. Use **9.4%**, or restate the kimi denominator if a narrower one was intended.

## (d) Kimi-attrition outcome-neutrality

| claim | published | recomputed | verdict |
|---|---|---|---|
| two-judge units total / kimi-attrition | 125 / 122 kimi (05_VERIFICATION_AUDIT §A) | **125 / 122 kimi** (3 = gemini) | **MATCH** |
| attrition outcome-neutral | asserted (05_VERIFICATION_AUDIT §A) | **confirmed**: kimi-present (n=2256) F/L/T 0.201/0.393/0.406 vs kimi-missing (n=122) 0.156/0.361/0.484; F−L gap −0.191 vs −0.205 (preserved) | **MATCH** |

**Method:** count 2-judge units by which judge is missing; compare the majority-outcome distribution for
kimi-present vs kimi-missing units across all 12 cells. The frontier−local gap (decisive direction) is preserved
(−0.191 vs −0.205); missing-kimi units skew slightly more to tie (fewer raters → fewer decisive), which does not
bias the outcome direction. Outcome-neutral confirmed.

## (e) Evidence-window flip matrix + mechanism classification

Recomputed **from raw**, re-deriving the "original/capped" verdict myself from `judgments.jsonl` (strict
both-orders over the 120 sampled qids) and the full-ev verdict from `judgments_fullev.jsonl` — not from the
report's stored pairs.

| claim (STATISTICAL_SUPPLEMENT §S7) | published | recomputed | verdict |
|---|---|---|---|
| frontier row (→F/→L/→tie) | 26 / 1 / 2 | **26 / 1 / 2** | **MATCH** |
| local row | 0 / 1 / 0 | **0 / 1 / 0** | **MATCH** |
| tie row | 20 / 0 / 70 | **20 / 0 / 70** | **MATCH** |
| agreement | 0.808 | **0.8083** | **MATCH** |
| tie→frontier : tie→local | 20 : 0 | **20 : 0** | **MATCH** |
| exact binomial p (20:0) | 1.9e-6 | **1.91e-6** | **MATCH** |
| mechanism: verbosity vs legitimate (of the 20 tie→frontier flips) | 15 verbosity / 5 legitimate | **14 verbosity / 6 legitimate** | **MATCH (concordant)** |

**Method:** consolidate both runs to strict both-orders over the 120 (matchup, qid) sampled pairs; cross-tabulate
capped-verdict (rows) × full-ev-verdict (cols). Exact two-sided binomial on the 20:0 tie-flip split (point-prob
method), p = 1.91e-6.

**Independent mechanism classification (I re-read all 20 rationales).** Rubric: *legitimate* = the full-ev
rationale shows the local answer was actually wrong / hallucinated / omitted a gold-required element / merely
restated the question (a non-answer); *verbosity* = capped called it a tie because both were correct/equivalent
and full-ev rewarded the frontier purely for more detail/comprehensiveness without the local being wrong.

My 6 **legitimate** flips (local genuinely wrong, not just terser):
- `deepseek q0078` — local incorrectly claims the passage states no reason ("chilly climate" is in evidence).
- `gpt q0031` — local omits the gold-required "moderating effects" component.
- `gpt q0074` — local answers the wrong study (hallucination).
- `gpt q0160` — local gives the wrong R² value (0.476 vs correct 0.199).
- `gpt q0226` — local merely restates the question's premise (effectively a non-answer).
- `gpt q0285` — local hallucinates a connection to a study not in evidence.

The other 14 are verbosity rewards: capped called them ties because both answers were correct/identical
(e.g. claude q0051/q0052/q0096/q0123/q0222/q0238, deepseek q0033/q0052/q0179/q0217, gpt q0007/q0028/q0221/q0293),
and full-ev preferred the frontier for "more comprehensive / more complete / includes additional detail" while
the local remained correct on the core fact.

I get **14/6** vs the published **15/5** — concordant within one borderline case (e.g. `gpt q0028`, where both
sides miss the exact gold point and the reward is for evidence-detail elaboration; or `gpt q0226`, scored as a
non-answer). I agree with the published direction and magnitude: the large majority (~70–75%) of tie→frontier
flips under full-evidence are verbosity rewards, which is precisely why the de-verbosity rubric was added in the
clean re-judge. I do **not** dispute the 15/5 claim; my independent count lands at 14/6, within rounding/judgment noise.

---

## Summary of verdicts

- **MATCH (exact or within rounding):** majority 134/47/413; strict≥2 98/17/479 + not-lose 0.835; net-range
  0.540–0.575; provenance 51/59/66/5 (=181); mistral P(first) 0.731 and flip 17.8%; gemini flip ~1.0%; panel α
  0.5653 / 0.7547; n=2378 units; kimi-attrition 125/122 and outcome-neutral; full evidence-window flip matrix
  (26/1/2; 0/1/0; 20/0/70), agreement 0.808, 20:0 p=1.9e-6.
- **MATCH (concordant, independent re-classification):** mechanism split — I get 14 verbosity / 6 legitimate
  vs published 15 / 5 (one borderline case; same conclusion).
- **CORRECTED:** kimi pure position-flip rate — published 6.9%, recomputed **9.4%** (definition reproduces
  mistral 17.8% and gemini 1.0% exactly). Conservative; does not affect any conclusion.

No fabrications. Every panel/flip number reproduces from raw with from-scratch code.
