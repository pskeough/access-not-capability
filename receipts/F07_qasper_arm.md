# F07 — QASPER frontier arm (qasper-arm): GOLD-STANDARD receipt

**Recompute script:** `C:/Research/Train_LLM/Final_Validation/scripts/F07_qasper_arm.py` (runs offline, exit 0, 25/25 checks pass).

**Consolidation rule (all cells):** strict both-orders. Each (matchup, question) has 2 judgments (order 0/1); decisive (frontier/local) only if both orders agree, else tie. `fwr = (frontier + 0.5*tie)/n`; `local_not_lose = (local + tie)/n`. The 12-cell summary reproduces this exactly.

**Raw inputs (exact paths):**
- Judgments: `C:/Research/Train_LLM/FinalRunPack/frontier_pilot/qasper_arm/judgments.jsonl` (12 matchups × 200 q × 2 orders = 4800 rows)
- Frozen summary (compared against, not trusted): `…/qasper_arm/summary.json`
- Closed-book answers: `…/qasper_arm/answers_{gpt,claude,deepseek}_base.jsonl`
- Stratum map: `C:/Research/Train_LLM/experiments/v5_raft_cot/corpora/qasper/eval_set.jsonl` (`question_type`)
- Clean-rejudge: `…/frontier_pilot/clean_rejudge/judgments_single.jsonl` (corpus=qasper) + `summary_single.json`
- Base reference: `…/frontier_pilot/summary.json` (single judge), `…/frontier_pilot/summary_panel.json` (panel)

**Stratum definition:** arm subsample is n=200 = **149 answerable** (`question_type` ∈ {single_hop, multi_hop}; qasper_answer_type ∈ {extractive, abstractive, yes_no}) + **51 OOC** (`question_type=out_of_corpus`, qasper_answer_type=unanswerable). Pooled across 3 frontier models → 447 answerable / 153 OOC decision-units.

---

## Claim-by-claim

| # | Claim (published value + source) | Recomputed | Verdict |
|---|---|---|---|
| 1 | **All 12 cells** of `qasper_arm/summary.json` (frontier_wins/local_wins/ties per matchup) | All 12 cells reproduce **exactly** (e.g. claude:base/8B 37/59/104; gpt:base/8B 38/63/99; claude:rag/8B 19/4/177) | **MATCH** |
| 2 | **Pooled closed-book vs 8B** `fwr 0.446 / not-lose 0.813` (STATISTICAL_SUPPLEMENT.md §S8; 07_REMEDIATION_RESULTS.md §3b) | fwr **0.4458**, not-lose **0.8133**, n=600 | **MATCH** |
| 3a | **Answerable closed-book** `fwr 0.313 / not-lose 0.982` (S8 / §3b table) | fwr **0.3132**, not-lose **0.9821**, n=447 | **MATCH** |
| 3b | **OOC closed-book** `fwr 0.833 / not-lose 0.320` (S8 / §3b) | fwr **0.8333**, not-lose **0.3203**, n=153 | **MATCH** |
| 3c | **Inversion** (entire pooled lift is the OOC stratum) | monotone 0.313 < 0.446 < 0.833 confirmed | **MATCH** |
| 4 | **RAG parity pooled 0.535** vs 8B; **answerable-only 0.524 / not-lose 0.906** (§3a, §3c) | pooled **0.5350**; answerable **0.5235 / 0.9060** | **MATCH** |
| 5 | **1B more competitive on QASPER**: 0.572 pooled vs base ~0.67–0.70 (gap 0.68→0.57) (§3d) | QASPER RAG-vs-1B pooled **0.5717**; base RAG-vs-1B single `[0.665, 0.704]`; 0.572 < base floor → gap narrows | **MATCH** |
| 6 | **Heterogeneity-premium NULL**: QASPER 0.535–0.573 ≤ base 0.573–0.618; premium does NOT grow (§3c) | QASPER frontier+RAG-vs-8B max(0.535 pooled, 0.524 ans) = **0.535** ≤ base single-pooled **0.5726** ≤ base panel max **0.605** | **MATCH** |
| 7 | **Frontier closed-book abstention ~65% both strata** (S8 / §3b: "abstains ~65% regardless of stratum") | F05-rule classifier: **pure-abstain** ANS=0.669, OOC=0.771, pooled-ALL=0.695; **abstain(pure+hedged)** ANS=0.776, OOC=0.837 | **CORRECTED** |
| 8 | **Clean-rejudge QASPER cells corroborate** (cross-check vs F06) | 6/6 cells reproduce `summary_single.json` exactly; pooled-vs-8B-answerable **0.5727** (= F06), n=447 | **MATCH** |

---

## Methods (per claim, 2–3 lines)

- **Cells / pooled / stratified (1–6):** group `judgments.jsonl` by (matchup, qid); consolidate strict both-orders; tally frontier/local/tie. Pool by summing the per-question decisions across gpt+claude+deepseek for a fixed (config, local). Stratify by `eval_set.question_type` (out_of_corpus = OOC, else answerable). Rates: `fwr=(F+0.5T)/n`, `not_lose=(L+T)/n`.
- **Base reference (5,6):** single-judge base RAG-vs-{8B,1B} winrates read directly from `frontier_pilot/summary.json`; panel from `summary_panel.json`. QASPER premium compared against both regimes.
- **Abstention (7):** deterministic regex/rule classifier (verbatim F05 rules; see below) applied to each closed-book answer → {pure_abstain, hedged, substantive}; tabulated by stratum.
- **Clean-rejudge (8):** same strict both-orders consolidation on `clean_rejudge/judgments_single.jsonl` filtered to corpus=qasper (answerable n=149/model); compared cell-by-cell to `summary_single.json` and to F06's recomputed 0.5727.

### F05-style abstention classifier rules (documented, used in Claim 7)
Split each answer into sentences (on `.!?` + newlines). Unicode-normalized (curly→straight quotes, en/em-dash→hyphen), case-insensitive.
- **RULE 1 (abstention sentence):** matches refusals on grounds of missing context/corpus — "I don't have / don't know / cannot / am unable to", "no/without {passage|context|paper|corpus|information|…} … {provided|given|available|shared}", "not {covered|included|available} in the {context|corpus|…}", "you {haven't|didn't} {provided|shared}", "unable to {answer|provide|determine}", "can't {say|provide|determine|…}".
- **RULE 2 (offer/filler):** "if you {provide|share|paste}", "please {share|provide}", "happy to help", "could you {share|specify}", "I can help", "let me know", "feel free" — not substantive.
- **RULE 3 (hedge bridge):** general non-corpus knowledge markers — "in general", "generally", "typically", "that said", "the (broader) literature", "based on general/common/prior", etc.
- **Classes:** **substantive** = no RULE-1 sentence. Else let `rw` = word count of residual (sentences matching neither RULE 1 nor RULE 2): **hedged** if (hedge-bridge AND rw≥12) OR rw≥30; otherwise **pure_abstain**. Derived: abstain = (pure+hedged)/n, pure-abstain = pure/n.

### Claim 7 — CORRECTED, with interpretation
The published phrasing "abstains ~65% regardless of stratum" is a rough characterization that holds **only** for the pooled **pure-abstain** rate (0.695 pooled-ALL; ANS 0.669 ≈ 65%). It is **not** uniform across strata: pure-abstain is markedly higher on OOC (0.771) than answerable (0.669), and the broader abstain rate (pure+hedged) is 0.776 / 0.837. Verdict **CORRECTED**: the *directional* claim is sound (frontier abstains heavily in both strata, ≥0.6 both — so it is NOT recalling answers from memory on either stratum, which is the load-bearing inference), but "~65% regardless of stratum" understates OOC abstention by ~10pp and should read "≈67–77% pure-abstention (higher on OOC), ≈78–84% including hedged disclaimers." The qualitative conclusion (the contamination-attack pre-emption) is unaffected.

---

## Question-mix-artifact interpretation (precise)

Naively, closed-book frontier appears far stronger on QASPER (pooled fwr **0.446**) than on the private base corpus (≈0.157). The recompute shows the **entire** pooled lift lives in the OOC stratum: on OOC the gold answer *is* "unanswerable," so a closed-book model that correctly abstains wins/ties (frontier fwr **0.833**, local not-lose only **0.320**). On **answerable** QASPER, closed-book frontier **loses to the RAG-8B exactly as on the private corpus** (fwr **0.313**, local not-lose **0.982**) — and frontier abstains heavily there too (pure-abstain ≈0.67), i.e. it is **not** recalling memorized public QASPER answers. Therefore the apparent "QASPER contamination lift" is a **question-mix artifact** (51/200 = 25.5% unanswerable items inflate the pooled frontier number), **not** memorization. This is *positive* evidence that the private-corpus democratization result is not a "frontier-lacks-public-knowledge" artifact, which pre-empts the contamination attack more strongly than a "we demonstrate contamination" framing.

**Heterogeneity-premium null:** the hypothesis that the frontier+RAG premium *grows* with corpus heterogeneity is rejected — QASPER (heterogeneous, public) frontier+RAG-vs-8B fwr is **flat-to-smaller** (0.535 pooled / 0.524 answerable; 0.5727 under clean re-judge) than homogeneous base (0.573 single / up to 0.605 panel / 0.5995 clean-rejudge pooled). QASPER is therefore framed as a parity-robustness + contamination-control check that *succeeds*, not as a heterogeneity-premium mechanism.

**Cross-regime robustness:** clean re-judging (full retrieved context + de-verbosity rubric) shifts the answerable RAG-vs-8B number up from 0.5235 (v1 single judge) to 0.5727, exactly as the doc cautions — but the null still holds because clean QASPER (0.5727) ≈ base single (0.573) and < base panel/clean-rejudge (~0.60–0.62).
