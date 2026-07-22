# Claims ledger — every claim, its verdict, its receipt

_186 claims adjudicated across 17 families (2026-06-11 campaign; 16 Opus validation agents + gap
closure). Roll-up: **132 MATCH · 25 CORRECTED · 22 NEW · 7 UNVERIFIABLE/retired.** Every number
below is backed by an idempotent script in `scripts/` (all exit 0 offline) and a receipt in
`receipts/`. Paper rule: cite ONLY values with a MATCH/CORRECTED/NEW ledger entry._

## Family roll-up

| family | receipt | verdicts | the one-line outcome |
|---|---|---|---|
| F01 lever-effects | F01_lever_effects.md | 12 MATCH | All 36 cells, Holm 27/36 (18/18, 7/9, 2/9), BH 29/36, TOST nulls — **exact** |
| F02 interactions | F02_lever_corpus_interactions.md | 8 MATCH, 2 NEW | All G-tests exact; NEW: permutation p=5e-4 robust to pooling; NEW: interaction non-uniform (MiniCPM saturated) |
| F03 mechanism | F03_mechanism.md | 7 MATCH, 2 CORRECTED | Base decomposition + all homogeneity values exact; **COVID/QASPER decomposition rows were stale** (old eval subsets) — corrected values below; conclusions unchanged |
| F04 ooc-honesty | F04_ooc_honesty.md | 9 MATCH, 3 NEW | 0.705/0.221 endpoints exact; NEW per-model (holds 3/3), NEW text-clustered CIs (still exclude 0.5, disjoint) |
| F05 closed-book | F05_frontier_closed_book.md | 6 MATCH, 1 CORRECTED | Panel/single not-lose exact; scripted abstention classifier: GPT ~50% attempts (MATCH), **Claude 76.8%** (band → "66–77%") |
| F06 clean-rejudge | F06_clean_rejudge_headline.md | 15 MATCH | Entire corrected headline exact (68/68 checks): net 0.600 [.580,.620] etc. |
| F07 qasper-arm | F07_qasper_arm.md | 8 MATCH, 1 CORRECTED | All 12 cells + stratification inversion + heterogeneity-null exact; abstention "~65% both strata" → ANS .669 / OOC .771 |
| F08 deepseek-v2 | F08_deepseek_v2.md | 16 MATCH | Fix verified to the answer level (14 exact-512 truncation artifacts; 0 cap hits in v2) |
| F09 arena-rouge | F09_arena_rouge.md | 11 MATCH, 1 NEW | BT rankings, alphas, full ROUGE grid exact; NEW: 3 never-beaten GOLD anchors are non-convergent → report as lower bounds |
| F10 confounds | F10_confounds.md | 7 MATCH, 3 CORRECTED | All confound verdicts hold; QASPER third-decimal drift (.4953/.4842/.7809) |
| F11 hit-miss ★NEW | F11_hit_miss_stratification.md | 4 MATCH, 3 NEW | **Tie mass is capability parity, not mutual blindness** — HIT−MISS delta ≈ 0 in all 4 pooled cells |
| F12 selective-answering ★NEW | F12_calibration_selective_answering.md | 1 MATCH, 2 CORRECTED, 3 NEW, 1 UNVERIFIABLE | **Abstention spot-check had the SIGN backwards**; NEW honesty matrix shows a corpus-homogeneity sign-flip; ECE infeasible (no logprobs) |
| F13 eval-integrity | F13_eval_set_integrity.md | 8 MATCH, 3 NEW | Base dedup/hash-locks exact; NEW: QASPER 381/400 unique (19 dups), COVID 398/400 |
| F14 cost ★NEW | F14_cost_deployment.md | 1 MATCH, 2 CORRECTED, 3 NEW | **v1 "$0.95" is a logging artifact — honest ≈ $14.7**; per-1k-question cost table built; latency/TCO = assumptions |
| F15 judge-robustness | F15_judge_robustness.md | 13 MATCH, 1 CORRECTED | Rule-sensitivity, provenance split, mistral primacy, flip matrix exact; kimi flip rate 6.9% → **9.4%** |
| F16 coverage matrix | F16_coverage_matrix.md | full doc sweep | Paper-facing docs clean; historical docs carry a documented stale set; 1 live contradiction (→F12); 5 uncovered (→F17) |
| F17 gap closure | F17_gap_closure.md | 5 MATCH, 2 NEW | gpt+claude pool, panel 60:26 q-sign, 416,698-token corpus, provenance (13 W-ids/28), decoding fallback; NEW: possible duplicate paper |

## The 25 corrections (use these values; old values retired)

| # | claim | old → corrected | receipt |
|---|---|---|---|
| 1 | COVID decomposition | .800/.950/.583/.877 (n=150) → **.8125/.9308/.5733/.8638** (n=400) | F03 |
| 2 | QASPER decomposition | .241/.852/.506/.589 (n=112) → **.2318/.8071/.5323/.5960** (n=302) | F03 |
| 3 | "≈equal recall" wording | → "similar low band" (base .294 vs QASPER .232; gap .062) | F03 |
| 4 | OOC abstention spot-check 36/98 vs 42/98 ("RAG abstains more") | **SIGN BACKWARDS — DELETE.** RAG abstains far LESS on QASPER OOC: 62/98 → 17/98 (strict: 58 → 6) | F12 |
| 5 | "abstention frequency similar; effect is quality-only" | → RAG **shifts behavior**, sign flips with homogeneity: base correct-abstain 61.5%→**100%**; QASPER 63.3%→**17.3%** | F12 |
| 6 | Claude/DeepSeek closed-book abstain "66–72%" | → Claude **76.8%**, DeepSeek 65.8% (state "≈66–77%", threshold-sensitive) | F05 |
| 7 | QASPER-arm abstention "~65% both strata" | → ANS .669 / OOC .771 (high in both, not uniform) | F07 |
| 8 | kimi position-flip rate 6.9% | → **9.4%** (213/2256) | F15 |
| 9 | QASPER position/verbosity/tie 3rd decimal | → .4953/.4842/.7809 (snapshot drift; verdicts unchanged) | F10 |
| 10 | v1 frontier run cost "$0.95" | → honest **≈$14.7** (gen $9.73 + judge ≈$5.0; logging artifact: resumed tokens uncounted, judge price 6.7×/10× low → ~8.8× effective) | F14 |
| 11 | "6.7× judge understatement" | → 6.67× input / 10× output ≈ **8.8× effective** | F14 |
| 12 | BT GOLD anchors 192.8/275.0/301.1 | → non-convergent (never-beaten); report as **lower bounds @ n_iter=400** | F09 |
| 13 | Holm "7/12, 2/12" (historical docs) | → **7/9, 2/9** (family=36) | F01/F16 |
| 14 | homogeneity "2×" (historical docs) | → **1.68×** centroid / 2.8× pairwise | F03/F16 |
| 15 | sign-test "p≈2e-12" (historical docs) | → 1.30e-24 / 1.01e-85 (iid) — and the PRIMARY stat is question-clustered (F06/F17) | F16 |
| 16 | closed-book "96–99%" (historical docs) | → panel 91.4–95.8% | F05/F16 |
| 17 | 8B "82%" / 1B "59%" / net "0.55" (historical docs) | → clean 78.7% / floor framing / net 0.600 | F06/F16 |
| 18–25 | minor: capped-single baseline source-mixing (±0.005); pooled n "~1030"→1025/1026; per-cell n nonuniform; F06 n=342 gpt cell; COVID-minicpm n=393–395; etc. | per receipts | F06/F05/F01 |

## The NEW science (validated additions)

| finding | value | receipt |
|---|---|---|
| **Tie mass = capability parity** | HIT−MISS net-win delta: base +0.009 [−.037,+.057]; QASPER +0.057 [−.029,+.142]; 8B ties frontier ~77% even WITH gold chunk retrieved | F11 |
| **Honesty sign-flip (upgrades Result 4)** | RAG correct-abstain on OOC: base 61.5→100%; QASPER 63.3→17.3%; risk-coverage: QASPER-RAG engages 93.9% of unanswerables | F12 |
| Interaction robustness + scope | permutation p=5e-4; driven by Llama8b+Phi, MiniCPM saturated | F02 |
| Text-clustered OOC CIs | base [.59,.83], QASPER [.19,.26] — disjoint, both exclude 0.5 | F04 |
| Per-1k-question frontier cost | RAG: gpt $6.31 / claude $16.34 / deepseek $2.20; closed-book $1.44/$1.70/$0.29 | F14 |
| Eval dedup, all corpora | base 315/345, QASPER 381/400, COVID 398/400 unique | F13 |
| Corpus size | 416,698 o200k tokens (fits 1M windows) | F17 |
| Possible duplicate paper | "Planning Early for Careers in Science" under 2 paper_ids | F17 |

## Retired / unverifiable-by-design (3 + 2 deleted claims)

- **ECE/AURC calibration** — structurally infeasible (no logprobs in any artifact). Honest negative; would require a logprob-capturing rerun. (F12)
- **Latency** — never measured; no speed claim is supportable. (F14)
- **Local-GPU TCO** — assumption only; $0 *marginal API* cost is the grounded statement. (F14)
- Deleted: the 36/98-vs-42/98 abstention spot-check and the audit's 11/25 rederivation (both wrong; superseded by F12's scripted classifier).
