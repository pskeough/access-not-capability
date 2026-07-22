# F16 — Coverage matrix (coverage-critic sweep of every quantitative claim)

**Family:** F16 coverage-critic.
**Verifier script:** `C:/Research/Train_LLM/Final_Validation/scripts/F16_coverage_matrix.py`
(offline, idempotent, no API; **23/23 checks PASS, exit 0**). It re-derives from raw the
cross-cutting facts the ledger leans on — the arithmetic-error corrections (05 §B), the
stale→panel→clean frontier chain, the stale decomposition rows, the cost staleness, and the
receipt-vs-receipt OOC-abstention consistency — rather than re-deriving every study number
(F01–F15 already each do that from raw with their own scripts).

**What this receipt is.** A single ledger: one row per *distinct* quantitative claim across the 11
swept documents → which receipt covers it (file + section) → covered / uncovered / contradicted /
**STALE**. Paper-facing docs (PAPER_RUNDOWN, STATISTICAL_SUPPLEMENT, 06, 08) are swept exhaustively;
historical docs (00–04, INTERPRETATION) are marked where they carry numbers **superseded** by the
receipts (the doc stays as history; the ledger records the corrected value).

**Receipts swept:** F01–F15 in `C:/Research/Train_LLM/Final_Validation/receipts/`.
**Docs swept:** `00_EXECUTIVE_SUMMARY.md`, `01_FINDINGS.md`, `02_CONFOUNDS_AND_METHODOLOGY.md`,
`04_PAPER_OUTLINE.md`, `05_VERIFICATION_AUDIT.md`, `06_PAPER_SKELETON.md`, `07_REMEDIATION_RESULTS.md`,
`08_CLEAN_REJUDGE_RESULTS.md`, `PAPER_RUNDOWN.md`, `STATISTICAL_SUPPLEMENT.md`,
`frontier_pilot/INTERPRETATION.md` (all under `C:/Research/Train_LLM/FinalRunPack/`).

Legend: **COVERED** = a receipt verifies the number from raw (MATCH/CORRECTED/NEW noted).
**STALE** = a *historical* doc prints a number a receipt has superseded (corrected value given).
**CONTRADICTED** = two live sources disagree and the receipt resolves it against the published value.
**UNCOVERED** = no receipt verifies it (gap). The "current/correct value" column carries the
receipt-blessed number.

---

## 1. Within-model lever effects (T1) — COVERED by F01

| # | Claim | Published (doc + loc) | Receipt | Verdict | Correct value |
|---|---|---|---|---|---|
| 1.1 | RAG-over-base win-rates base 84.1/84.8/81.2 | SUPP S1; 01 §1; 00 §1 ("81–85%") | F01 §T1 | COVERED MATCH | same |
| 1.2 | RAG-over-base COVID 86.4/83.6/79.2 | SUPP S1; 01 §1 ("79–86") | F01 §T1 | COVERED MATCH | same |
| 1.3 | RAG-over-base QASPER 59.6/60.9/62.3 | SUPP S1; 01 §1 ("60–62") | F01 §T1 | COVERED MATCH | same |
| 1.4 | RAG-over-LoRA base 82–87 / COVID 80–85 / QASPER 59–62 | 01 §1; SUPP S1 ("all 18 survive") | F01 §full-table | COVERED MATCH | `lora_rag`-vs-`lora` mapping |
| 1.5 | LoRA-over-base 48.8 ns / 55.8 / 53.2 (base) | SUPP S1; 01 §1 | F01 §T1 | COVERED MATCH | same |
| 1.6 | LoRA-over-base COVID 53.0/54.4/54.9; QASPER 53.5/51.5 ns/52.9 | SUPP S1; 01 §1 | F01 §T1 | COVERED MATCH | same |
| 1.7 | RAFT-over-RAG: COVID-MiniCPM 58.0 p<1e-4; QASPER-Llama 54.6 p=8e-4 | SUPP S1; 01 §1 | F01 §T1 | COVERED MATCH | McNemar 3.7e-6 / 7.6e-4 |
| 1.8 | RAFT COVID-Phi 50.9 p=4e-4 TOST-equiv | SUPP S1; 01 §1 | F01 §McNemar/TOST | COVERED MATCH | same |
| 1.9 | McNemar p<1e-4 base/COVID retrieval | SUPP S1; 00 §1 | F01 §McNemar | COVERED MATCH | base-llama ~4e-54 |
| 1.10 | **Holm survivors 27/36; retrieval 18/18, LoRA 7/9, RAFT 2/9** | SUPP S1 (7/9,2/9); **00 §F2, 01 §1, 02 §D, 04 §4 say 7/12, 2/12** | F01 §global | **COVERED — 7/9,2/9 correct; 7/12,2/12 STALE** | **7/9, 2/9** (05 §B-2; 06 §contrib-1) |
| 1.11 | BH survivors 29/36; LoRA 8/9, RAFT 3/9 | SUPP S1 ("8/9,3/9"); 05 §B-2 | F01 §global | COVERED MATCH | 8/9, 3/9 |

> **STALE flag 1.10:** historical docs `00_EXECUTIVE_SUMMARY §F2`, `01_FINDINGS §1`,
> `02_CONFOUNDS §D`, `04_PAPER_OUTLINE §4` print Holm denominators **7/12** and **2/12** (family
> miscounted as 4×12=48). Correct family = 4 levers × **9** cells = 36 ⇒ **LoRA 7/9, RAFT 2/9**
> (SUPP S1 and 06 already use 7/9, 2/9). F01 confirms 27/36, 18/18, 7/9, 2/9 from raw. F16 verifier
> check A1.

---

## 2. Lever × corpus interaction (T2) — COVERED by F02

| # | Claim | Published | Receipt | Verdict | Correct value |
|---|---|---|---|---|---|
| 2.1 | RAG-over-base base-vs-QASPER G=26.5 p=2.7e-7 | SUPP S2; 01 §2; 00 §F3; 04 §5 | F02 #1 | COVERED MATCH | G=26.477 |
| 2.2 | RAG-over-base COVID-vs-QASPER G=17.6 p=2.7e-5 | SUPP S2; 01 §2 | F02 #2 | COVERED MATCH | G=17.635 |
| 2.3 | RAG-over-base base-vs-COVID G=2.48 p=0.11 ns | SUPP S2; 01 §2 | F02 #3 | COVERED MATCH | G=2.485 |
| 2.4 | RAG-over-LoRA base-vs-QASPER G=39.8 p=2.8e-10 | SUPP S2; 01 §2 | F02 #4 | COVERED MATCH | G=39.796 |
| 2.5 | RAG-over-LoRA COVID-vs-QASPER G=38.7 p=4.9e-10 | SUPP S2; 01 §2 | F02 #5 | COVERED MATCH | G=38.729 |
| 2.6 | RAG-over-LoRA base-vs-COVID G=0.30 p=0.58 ns | SUPP S2; 01 §2 | F02 #6 | COVERED MATCH | G=0.299 |
| 2.7 | LoRA-over-base all corpus-pairs p>0.09 (0.61/0.09/0.15) | SUPP S2; 01 §2; 00 §F2 | F02 #7 | COVERED MATCH | min p=0.091 |
| 2.8 | RAFT-over-RAG all corpus-pairs p>0.09 (0.10/0.16/0.98) | SUPP S2; 01 §2 | F02 #8 | COVERED MATCH | min p=0.100 |
| 2.9 | Per-model base-vs-QASPER interaction (not uniform; MiniCPM saturated) | (not published) | F02 §(B) | NEW | llama dominates G≈25–39 |
| 2.10 | Question-block permutation p≈5e-4 corroborates G | (not published) | F02 §(C) | NEW | perm_p=0.0005 |

---

## 3. Mechanism: decomposition + homogeneity (T3) — COVERED by F03 (with STALE COVID/QASPER decomp rows)

| # | Claim | Published | Receipt | Verdict | Correct value |
|---|---|---|---|---|---|
| 3.1 | base decomp recall 0.294 / hit 0.878(86) / miss 0.826(207) / agg 0.841 | SUPP S3; 01 §3; 00 §F3 | F03 §(a) | COVERED MATCH | same (n=293) |
| 3.2 | **COVID decomp recall 0.800 / hit 0.950(120) / miss 0.583(30) / agg 0.877** | SUPP S3; 01 §3 | F03 §(a) | **STALE** (published from n=150 eval) | **recall 0.812 / hit 0.931(325) / miss 0.573(75) / agg 0.864, n=400** |
| 3.3 | **QASPER decomp recall 0.241 / hit 0.852(27) / miss 0.506(85) / agg 0.589** | SUPP S3; 01 §3 | F03 §(a) | **STALE** (published from n=112 eval) | **recall 0.232 / hit 0.807(70) / miss 0.532(232) / agg 0.596, n=302** |
| 3.4 | Decomposition identity recall·hit+(1−recall)·miss = aggregate exactly | SUPP S3; 01 §3 | F03 §(a) | COVERED MATCH | \|pred−agg\|=0 all 3 |
| 3.5 | Centroid base 0.235 / COVID 0.136 / QASPER 0.140 | SUPP S3; 01 §3; 00 §F3 | F03 §(b) | COVERED MATCH | 0.2348/0.1364/0.1395 |
| 3.6 | Pairwise cosine base 0.053 / COVID 0.018 / QASPER 0.019 | SUPP S3; 01 §3 | F03 §(b) | COVERED MATCH | 0.0531/0.0177/0.0188 |
| 3.7 | **"2× more homogeneous"** (centroid) | **00 §F3 / 01 §3 / 04 §5 say "2×"** | F03 §(b); 05 §B-3 | **STALE** | **1.68× centroid (0.235/0.140); 2.8× pairwise** |
| 3.8 | Homogeneity ratio 1.68× / 2.8× | SUPP S3 implied; 06 §contrib-2 ("1.68×/2.8×") | F03 §(b) | COVERED MATCH | 1.70× / 2.91× recomputed |
| 3.9 | Chunk counts base 593 vs QASPER 19,817; ~30–33× | 01 §3 ("33×"); SUPP S3 ("30×"); 05 §B-6 (33× vs 27.8×) | F03 §(b)/(c) | COVERED MATCH | 593 raw / 27.8× on eligible (591 vs 16,445) |
| 3.10 | COVID size 8,260 chunks (smallest store, highest recall) | (impl. 04 corpora) | F03 §(c) | COVERED NEW | 8,260 raw / 4,807 eligible |
| 3.11 | Size is not the driver (qualitative non-confound) | 00 §F3; 01 §3; 04 §5 | F03 §(c) | COVERED MATCH | 3 independent legs hold |

> **STALE flags 3.2/3.3:** `FinalRunPack/artifacts/ablation_decomposition.json` (the file 01/SUPP
> cite) holds COVID/QASPER rows computed on an **earlier, smaller** eval set (n=150 / 112). Current
> raw eval is n=400 (COVID answerable) / 302 (QASPER answerable). F03 re-ran the project's own
> `ablation_decompose.py` unmodified on current raw and reproduced the corrected values to the digit.
> base row is unaffected (n=293 then and now). **Mechanism conclusion is unchanged** (recall still
> doesn't track size; miss-win still tracks homogeneity). F16 verifier checks C1–C3.
> **STALE flag 3.7:** "2× more homogeneous" should read **1.68×** on the cited centroid metric.

---

## 4. OOC / retrieval-honesty (T4) — COVERED by F04, with CONTRADICTION resolved by F12

| # | Claim | Published | Receipt | Verdict | Correct value |
|---|---|---|---|---|---|
| 4.1 | RAG-over-base OOC base 71% (n=156) / QASPER 22% (n=294) | SUPP S4; 01 §4; 00 §F4; 04 §7 | F04 core | COVERED MATCH | 0.7051 / 0.2211 |
| 4.2 | RAG-over-LoRA OOC base 74% / QASPER 37% | 01 §4 | F04 core | COVERED MATCH | 0.7436 / 0.3656 (lora_rag-vs-lora def) |
| 4.3 | LoRA-over-base OOC base 48% / QASPER 33% | 01 §4 | F04 core | COVERED MATCH | 0.4808 / 0.3316 |
| 4.4 | Answerable RAG helps base/COVID 80–86%, QASPER 60–61% | 01 §4; SUPP S4 | F04 core | COVERED MATCH | base 0.834 covid 0.831 qasper 0.609 |
| 4.5 | base wins 78% of QASPER OOC decisive | 00 §F4; 01 §4 | F04 §scoped | COVERED MATCH | consistent w/ 0.221 |
| 4.6 | Per-model OOC breakouts (3/3 same direction) | (not published) | F04 NEW(i) | NEW | base 0.61–0.78; QASPER 0.15–0.27 |
| 4.7 | Text-clustered CIs (27 base / 94 QASPER clusters) | (not published) | F04 NEW(ii) | NEW | base [0.59,0.83] excl 0.5 |
| 4.8 | **"abstention frequency similar (Llama OOC base 36/98 vs RAG 42/98)"** | **01 §4 footnote; 00 §F4; SUPP S4** | **F12 §4** | **CONTRADICTED** | **base 62/98 vs RAG 17/98 — RAG abstains FAR LESS** |
| 4.9 | Audit rederivation 11/98 vs 25/98 | 05 §C-8 | F12 §4 | CONTRADICTED (audit also wrong magnitude) | both 36/42 & 11/25 unreproducible |

> **CONTRADICTION 4.8 (the load-bearing one):** the published footnote frames OOC abstention as
> "frequency similar; the effect is *quality* when engaging." F12 recomputes (extended classifier) on
> raw llama8b QASPER-OOC answers: **base 62/98 abstain vs RAG 17/98** (pure: 58 vs 6). RAG abstains
> *far less* — it engages retrieved heterogeneous passages. The 05-audit rederivation (11 vs 25) has
> the right *direction* but a 3× different magnitude and is itself unreproducible. F16 verifier check
> D1 reproduces the **direction** from raw with an independent lexical signal (base 63/98 > RAG
> 18/98). The paper's *conclusion* (RAG harms OOC honesty on QASPER) survives and is in fact
> *stronger* than stated; only the "frequency similar" hedge is wrong. F04 explicitly defers the
> frequency spot-check (so F04 and F12 are consistent: F04 verifies the quality win-rates, F12 owns
> the frequency claim). **Action for the paper: delete "abstention frequency is similar" and the
> 36/98 vs 42/98 spot-check.**

---

## 5. Arena (Bradley-Terry + ROUGE, T5/T6) — COVERED by F09

| # | Claim | Published | Receipt | Verdict | Correct value |
|---|---|---|---|---|---|
| 5.1 | GOLD dominates every ranking | 01 §5; 00 cross-model; 04 figs | F09 #1 | COVERED MATCH | GOLD top all 6 cells |
| 5.2 | base-rag GOLD BT 56.5 vs Phi 0.61 | 01 §5; SUPP/_INDEX L38 | F09 #2 | COVERED MATCH | 56.5316 / 0.6145 |
| 5.3 | base-lora_rag GOLD BT 192.8 vs Llama 0.36 | 01 §5; _INDEX L38 | F09 #3 | COVERED MATCH (only @n_iter=400) | 192.84 (divergent — see 5.10) |
| 5.4 | COVID best → Llama both configs (BT 0.72–0.79 vs Phi 0.30–0.52) | 01 §5 | F09 #4 | COVERED MATCH | llama best non-GOLD |
| 5.5 | QASPER best → Phi both configs (BT 0.26–0.31 vs Llama 0.15–0.19) | 01 §5 | F09 #5 | COVERED MATCH | phi best non-GOLD |
| 5.6 | base flips: rag→Phi, lora_rag→Llama | 01 §5; 00 cross-model | F09 #6 | COVERED MATCH | confirmed |
| 5.7 | Panel α base 0.63 / COVID 0.64 / QASPER 0.57 | 01 §5; SUPP S10; 02 §A; _INDEX L46 | F09 #7–9 | COVERED MATCH | 0.6308/0.6409/0.5659 |
| 5.8 | ROUGE base-llama 0.223/0.480/0.172/0.559 | 01 §6; SUPP S11; _INDEX L41 | F09 #10 | COVERED MATCH | exact |
| 5.9 | ROUGE rag/lora_rag > base/lora all 9 rows | 01 §6; 00; 04; SUPP S11 | F09 #11 | COVERED MATCH | min(rag,lora_rag)>max(base,lora) |
| 5.10 | 3 GOLD anchors non-convergent (n_iter artifacts) | (not published) | F09 §NEW | NEW | base-lora_rag/qasper-rag/qasper-lora_rag |
| 5.11 | Base arena recompute reproduces published, max diff 0.0 | _INDEX L39; 00 §cross-model; 01 §5 | F09 (impl.) | COVERED MATCH | rouge grid max diff 0.000 |

---

## 6. Frontier closed-book — COVERED by F05; SINGLE-judge numbers are STALE vs panel

| # | Claim | Published | Receipt | Verdict | Correct value |
|---|---|---|---|---|---|
| 6.1 | **Single closed-book not-lose 98.8% (8B) / 96.0% (1B)** | INTERP §Headline; SUPP S5; _INDEX L42; 04 §6 ("96–99%") | F05 (a) | **COVERED MATCH but STALE as headline** (panel supersedes) | single 98.8/96.0 correct; **headline → panel 95.8/91.4** |
| 6.2 | **Panel closed-book not-lose 95.8% / 91.4%** | 05 §A; 06 §6; SUPP S5; 08 (impl.) | F05 (a) | COVERED MATCH | 95.8 / 91.4 |
| 6.3 | Panel outright loss 75.0% (8B) / 65.9% (1B) | 05 §A; SUPP S5; 06 §5 | F05 (a) | COVERED MATCH | 446/595, 392/595 |
| 6.4 | "Abstract closed-book 96–99%" → corrected 91–96% | 05 §A; 04 §6 stale | F05 (a) | STALE (04/abstract) | 91–96% (1B falls outside 96–99) |
| 6.5 | Per-model frontier win-rate single 0.14–0.22 | INTERP §Headline | F05 (a) | COVERED MATCH | [0.142,0.220] |
| 6.6 | **"GPT attempts ~51%, never formal abstention"** | INTERP; SUPP S5; 05 §C-7; 02 | F05 (b) | COVERED MATCH | 49.9% attempt |
| 6.7 | **"Claude/DeepSeek abstain 66–72%"** | SUPP S5; 05 §C-7 | F05 (b) | **CORRECTED** | Claude **76.8%** (>72), DeepSeek 65.8%; band threshold-sensitive |
| 6.8 | pure-abstain on OOC never scored a loss (0/98) | 05 §E ("0/45"); INTERP | F05 §cross-tab | COVERED MATCH (stronger) | 0/98 |
| 6.9 | Scripted abstention classifier (citable rules) | (checklist item, never run) | F05 (b) NEW | NEW | RULE 1–3 verbatim |

> **STALE flag 6.1/6.4:** `INTERPRETATION.md` and `04_PAPER_OUTLINE` lead with the **single-judge**
> closed-book numbers (98.8/96.0, "96–99%"). The 3-judge **panel** supersedes them as the headline:
> **95.8/91.4** (05 §A is explicit: "panel supersedes every frontier headline number"; the abstract's
> 96–99% becomes 91–96%). The single-judge numbers are not *wrong* — they're the single-judge
> motivation row — but they must not be the headline. F05 verifies both; F16 verifier checks B1/B2.
> **CORRECTED 6.7:** Claude formally abstains ~77%, above the published 72% ceiling.

---

## 7. Frontier same-retrieval (capability premium) — the most-revised cluster

This is the single most superseded number in the corpus. The chain is:
**single-judge (INTERP/04, STALE) → panel (05/06) → clean re-judge (08/SUPP S6, the committed headline).**

| # | Claim | Published | Receipt | Verdict | Correct value |
|---|---|---|---|---|---|
| 7.1 | **Single same-RAG not-lose 81.7% (8B) "82%"** | INTERP §Headline; 04 §6 abstract ("82%"); SUPP S7 (capped 0.817) | F05/F06; F11 | **STALE** | **panel 77.4% → clean 78.7%** |
| 7.2 | **Single same-RAG not-lose 58.7% (1B) "59%"** | INTERP; 04 §6 ("59%") | F06 | **STALE** | **panel 55.7% (CI crosses 50) → clean 54.6%** |
| 7.3 | **"net win-rate 0.55" (8B)** | INTERP; 04 §6/abstract | F06; 05 §B-4 | **STALE / ill-defined** | **clean 0.600 [0.580,0.620]** (capped panel 0.573) |
| 7.4 | Single net 1B 0.69 / 0.66–0.70 | INTERP; 04 §6 | F06; F08 | STALE | clean 0.720 [0.695,0.746]; v2 panel 0.706 |
| 7.5 | **"frontier wins 83% (186:38) decisive vs 8B; sign p≈2.4e-12"** | INTERP; _INDEX L44; 04 §6 | F06; 05 §B-1 | **STALE (record) + CONTRADICTED (p)** | panel 74% (134:47); **clean 94% (219:14)**; correct exact p(186:38)=**1.30e-24** |
| 7.6 | **"frontier wins 92% (423:35) vs 1B; sign p≈2.1e-12"** | INTERP; _INDEX L44 | F06; 05 §B-1 | **STALE + CONTRADICTED (p)** | panel 85.4%; clean 97% (467:14); correct p(423:35)=**1.01e-85** |
| 7.7 | **Clean net vs 8B base-all 0.600 [0.580,0.620], not-lose 0.787, tie 77%, share 94% (219:14)** | 08; SUPP S6; 06 register | F06 §S6 | **COVERED MATCH (the committed headline)** | exact |
| 7.8 | Clean net vs 8B base-answerable 0.618 [0.595,0.641], not-lose 0.749 | 08; SUPP S6 | F06 §S6 | COVERED MATCH | exact |
| 7.9 | Clean net vs 8B QASPER-answerable 0.573 [0.543,0.603], not-lose 0.810 | 08; SUPP S6 | F06 §S6 | COVERED MATCH | exact |
| 7.10 | Clean net vs 1B base-all 0.720; answerable 0.741; QASPER 0.764 | 08; SUPP S6 | F06 §S6 | COVERED MATCH | exact |
| 7.11 | Clean q-sign p ≤2.6e-19 (8B) / ≤4e-25 (1B) | 08; SUPP S6 | F06 §S6 | COVERED MATCH | 2.65e-19 / 5.2e-40 etc. |
| 7.12 | Per-cell base vs 8B: gpt 66/5/271 .589; claude 87/3/254 .622; deepseek 66/6/272 .587 | SUPP S6 | F06 §per-cell | COVERED MATCH | exact incl Wilson CIs |
| 7.13 | Per-cell QASPER vs 8B: gpt .554; claude .604; deepseek .560 | SUPP S6 | F06 §per-cell | COVERED MATCH | exact |
| 7.14 | Local decisive wins vs 8B 14/1030 = 1.4% (clean) | 08; SUPP S6 | F06 | COVERED MATCH | 14/1030 |
| 7.15 | S7 capped panel 0.573/0.774; capped single 0.573/0.817; clean 0.600/0.787 | SUPP S7; 08 | F06 §S7 | COVERED MATCH | exact (single 0.578 w/ v2 deepseek; noted) |
| 7.16 | S7 evidence-window flip 26/1/2; 0/1/0; 20/0/70; agree 0.808; 20:0 p=1.9e-6 | SUPP S7; 07 §2; 05 §C-2 | F15 (e) | COVERED MATCH | exact |
| 7.17 | S7 mechanism 15/20 verbosity, 5/20 legitimate | SUPP S7; 07 §2; 08 | F15 (e) | COVERED MATCH (concordant) | independent 14/6 (within 1) |
| 7.18 | 07 §2 extrapolated not-lose "0.774→~0.64" (full-ev only) | 07 §2 | F15(e)/F06 | COVERED (superseded by 08) | full-ev was 75% verbosity; clean 0.787 |
| 7.19 | Hit/miss stratification: tie mass = capability, not retrieval blindness | (06 §6 demanded; never run) | F11 NEW | NEW | HIT−MISS Δnet CI incl 0 all 4 cells |
| 7.20 | 8B near-parity ON HITS (base net 0.607; QASPER 0.616) | (not published) | F11 #5–6 | NEW | excludes 0.5 |

> **STALE cluster 7.1–7.6:** every single-judge same-retrieval number in `INTERPRETATION.md` and
> `04_PAPER_OUTLINE` (82% / 59% / net 0.55 / 83%:92% / p≈2e-12) is superseded by the **clean re-judge**
> (08 / SUPP S6), which is the value the v2 skeleton (06) and 08 commit to. The sign-test p-values
> (2.4e-12, 2.1e-12) are additionally **arithmetically wrong** (05 §B-1): the correct exact two-sided
> binomial on 186:38 and 423:35 is 1.30e-24 and 1.01e-85 (F16 verifier checks A3a/A3b reproduce both).
> `verify_scaled_run.py` was wrongly cited as the p-value source — it computes none. **"Parity/match/
> equivalence" language is dead** (06 §review-1): the corrected frontier edge is real and significant.

---

## 8. QASPER frontier arm — COVERED by F07

| # | Claim | Published | Receipt | Verdict | Correct value |
|---|---|---|---|---|---|
| 8.1 | All 12 qasper-arm cells reproduce | 07 §3; SUPP S8 | F07 #1 | COVERED MATCH | exact |
| 8.2 | Pooled closed-book vs 8B fwr 0.446 / not-lose 0.813 | SUPP S8; 07 §3b | F07 #2 | COVERED MATCH | 0.4458 / 0.8133 |
| 8.3 | Answerable closed-book fwr 0.313 / not-lose 0.982 | SUPP S8; 07 §3b | F07 #3a | COVERED MATCH | 0.3132 / 0.9821 |
| 8.4 | OOC closed-book fwr 0.833 / not-lose 0.320 | SUPP S8; 07 §3b | F07 #3b | COVERED MATCH | 0.8333 / 0.3203 |
| 8.5 | Inversion: entire lift is the OOC stratum | SUPP S8; 07 §3b | F07 #3c | COVERED MATCH | 0.313<0.446<0.833 |
| 8.6 | RAG parity QASPER fwr 0.535 pooled / 0.524 answerable, not-lose 0.906 | 07 §3a; SUPP S8 | F07 #4 | COVERED MATCH | exact |
| 8.7 | 1B more competitive on QASPER (gap 0.68→0.57) | 07 §3d; SUPP S8 | F07 #5 | COVERED MATCH | 0.572 pooled |
| 8.8 | Heterogeneity-premium NULL (QASPER ≤ base) | 07 §3c; 08 #3; SUPP S6/S8; 06 register | F07 #6 | COVERED MATCH | 0.535 ≤ 0.573–0.618 |
| 8.9 | private base pooled closed-book reference ~0.157 (0.16) | SUPP S8 | F07 (impl.) | COVERED MATCH | ~0.157 |
| 8.10 | "frontier abstains ~65% regardless of stratum" | SUPP S8; 07 §3b | F07 #7 | **CORRECTED** | pure-abstain ANS 0.669 / OOC 0.771; pooled 0.695 — higher on OOC |

> **CORRECTED 8.10:** "~65% regardless of stratum" holds only for the pooled pure-abstain rate; OOC
> pure-abstain is ~77%, answerable ~67%. The load-bearing inference (frontier abstains heavily in
> *both* strata ⇒ not memorizing public answers) is unaffected.

---

## 9. DeepSeek v2 confound fix — COVERED by F08

| # | Claim | Published | Receipt | Verdict | Correct value |
|---|---|---|---|---|---|
| 9.1 | v2 single vs 8B 51/9/285 fwr 0.561; vs 1B 133/12/200 fwr 0.675 | SUPP S9; 07 §1 | F08 #1–2 | COVERED MATCH | exact |
| 9.2 | v2 panel vs 8B 69/32/244 fwr 0.554; vs 1B 164/22/159 fwr 0.706 | SUPP S9; 07 §1 | F08 #3–4 | COVERED MATCH | exact |
| 9.3 | Δfwr +0.016/+0.011/+0.018/+0.036 | SUPP S9; 07 §1 | F08 #5–8 | COVERED MATCH | exact |
| 9.4 | 24 v1 decisive local-wins; 16 dissolved (15 tie,1 frontier); 14 provable artifacts; 17/24 tainted | SUPP S9; 07 §1; 05 §C-1 | F08 #9–13 | COVERED MATCH | exact |
| 9.5 | v1 41 exact-512-token answers; v2 0 cap hits (max 1984), 0 CoT leaks | SUPP S9; 07 §1 | F08 #12,14,15 | COVERED MATCH | exact |
| 9.6 | Panel attrition 664/26/0; 3-judge-only shift ≤0.003 | 07 §1; SUPP S9 | F08 #16–17 | COVERED MATCH | exact |

---

## 10. Confounds (S10) — COVERED by F10 (3 QASPER rows CORRECTED for snapshot drift)

| # | Claim | Published | Receipt | Verdict | Correct value |
|---|---|---|---|---|---|
| 10.1 | Position bias base 0.490 / COVID 0.497 / QASPER 0.496 | SUPP S10; 02 §A; 00 §clean | F10 #1 | COVERED (base/COVID MATCH; **QASPER CORRECTED 0.4953**) | snapshot drift, verdict unchanged |
| 10.2 | Verbosity base 0.501 / COVID 0.428 / QASPER 0.487 | SUPP S10; 02 §A; 00 | F10 #2 | COVERED (QASPER CORRECTED 0.4842) | COVID<0.5 holds |
| 10.3 | Corpus-routing 1.0 all 18 cells; 0 empty retrievals | SUPP S10; 02 §B; 00 | F10 #3 | COVERED MATCH | 18/18 = 1.0 |
| 10.4 | Answer completeness 400/config (base 345), 0 empty | SUPP S10; 02 §C; 00 §clean | F10 #4 | COVERED MATCH | 36/36 cells |
| 10.5 | recall@6 base 0.294 / COVID 0.800 / QASPER 0.241 (covariate) | SUPP S10; 02 §B | F10/F03 | COVERED (COVID/QASPER STALE — see 3.2/3.3) | corrected 0.294/0.812/0.232 |
| 10.6 | MiniCPM truncation lora_rag 136 / lora 78 / rag 23 (QASPER); COVID lora 22/rag 12; phi 2 | SUPP S10; 02 §C; 05 §C-6 | F10 #5 | COVERED MATCH | all 7 counts exact |
| 10.7 | Tie-rate base 0.527 / COVID 0.526 / QASPER 0.782 | SUPP S10; 02 §D; 00 limitation | F10 #6 | COVERED (QASPER CORRECTED 0.7809) | low-power conclusion holds |
| 10.8 | Frontier prompt symmetry byte/token-exact (GPT 345/345 delta=11) | SUPP S10; 05 §E/§C-9 | F10 #9 | COVERED MATCH | delta={11} on 30-q sample |
| 10.9 | Panel α base 0.63/COVID 0.64/QASPER 0.57; frontier panel 0.565 | SUPP S10; 02 §A; 05 §A | F10/F09/F15 | COVERED MATCH | see 5.7, 12.6 |
| 10.10 | Mistral primacy 73% first-position of decisive raw | SUPP S10; 05 §C-5; 06 §3 | F15 (c) | COVERED MATCH | 0.7306 |
| 10.11 | Eval duplicates base 315/345 unique; OOC 27/52 unique | SUPP S10; 05 §C-3; 02 | F13 | COVERED MATCH | 315 / 27 |
| 10.12 | Gold provenance base Qwen-2.5-14B; COVID/QASPER human | SUPP S10; 02 §C; 00; _INDEX L37 | F10 #7 / F13 (c) | COVERED MATCH | exact tags |

---

## 11. Methods / consolidation (S11, 02 §D) — COVERED across receipts

| # | Claim | Published | Receipt | Verdict |
|---|---|---|---|---|
| 11.1 | Strict both-orders consolidation; ties=½ | SUPP S11; 02 §D; 04 §3 | F01–F11 all | COVERED MATCH (applied throughout) |
| 11.2 | Tests: exact McNemar / binomial sign / bootstrap / Wilson / TOST / G / BT / Krippendorff | SUPP S11; 02 §D | F01/F02/F06/F09/F15 | COVERED MATCH |
| 11.3 | Global Holm + BH over 36-test family | SUPP S11; 02 §D | F01 | COVERED MATCH |
| 11.4 | Frontier arm estimation-first (clustered CIs, descriptive sign) | SUPP S11; 06 §3 | F06/F11 | COVERED MATCH |
| 11.5 | ROUGE-L triangulation reproduces ordering | SUPP S11; 01 §6 | F09 | COVERED MATCH |
| 11.6 | Per-cell n frontier 334–345 single / 194–200 panel; clean 342–344 base, 149 QASPER ans | SUPP S11; 05 §B-5 | F05/F06/F08 | COVERED MATCH | (05 §B-5 corrects blanket "n=345") |

---

## 12. Judge robustness (05 §A/§C-5; 06 §3) — COVERED by F15

| # | Claim | Published | Receipt | Verdict | Correct value |
|---|---|---|---|---|---|
| 12.1 | Panel majority-of-decisive 134/47/413 net 0.573 not-lose 0.774 | 05 §A; SUPP S7 | F15 (a) | COVERED MATCH | exact |
| 12.2 | Strict ≥2-decisive 98/17/479 not-lose 0.835 net 0.568 | 06 §3 | F15 (a) | COVERED MATCH | exact |
| 12.3 | Net-range across 5 variants 0.540–0.575 | 05 §A; 06 §3 | F15 (a) | COVERED MATCH | 0.5404–0.5750 |
| 12.4 | Decisive provenance 51 unanimous / 59 2-1 / 66 single / 5 two-judge (=181) | 06 §3 | F15 (b) | COVERED MATCH | exact |
| 12.5 | Mistral P(first)=0.731, flip 17.8%; gemini 1.0%; **kimi 6.9%** | 05 §C-5 | F15 (c) | COVERED (kimi **CORRECTED 9.4%**) | mistral/gemini MATCH |
| 12.6 | Panel α 0.5653 (all) / 0.7547 (no mistral); 2378 units | 05 §A; SUPP S10 | F15 (c) | COVERED MATCH | exact |
| 12.7 | Kimi attrition 125/122; outcome-neutral | 05 §A | F15 (d) | COVERED MATCH | gap preserved |
| 12.8 | Robustness: 8B near-parity holds all 5 variants; 1B–8B gap +0.036–0.110 | 05 §A | F15 (a) | COVERED MATCH | net 0.540–0.575 |

> **CORRECTED 12.5:** kimi pure position-flip is **9.4%** (213/2256), not 6.9%; same definition
> reproduces mistral 17.8% and gemini 1.0% exactly. Conservative; contained by both-orders.

---

## 13. Eval-set integrity — COVERED by F13

| # | Claim | Published | Receipt | Verdict |
|---|---|---|---|---|
| 13.1 | base 345 qids / 315 unique / 52 OOC / 27 OOC-unique | SUPP S10; 05 §C-3; 06 §3 | F13 (a) | COVERED MATCH |
| 13.2 | base OOC multiplicity (one ×6, …) ; 5 answerable dup pairs; 11 paraphrase triplets | 05 §C-3; 06 §3 | F13 (a) | COVERED MATCH |
| 13.3 | QASPER OOC 98 / 94 unique | F04 L93 | F13 (a) | COVERED MATCH |
| 13.4 | SHA-256 hash-locks (3 corpora × all snapshots) | SUPP S10; 02 §C | F13 (b) | COVERED MATCH |
| 13.5 | Gold provenance tags + timestamps | SUPP S10; 02 §C; 00 | F13 (c) | COVERED MATCH |
| 13.6 | question_type distributions per corpus | (impl. 04 corpora table) | F13 (d) | COVERED NEW |
| 13.7 | QASPER 19 dup qids (381 unique); COVID 2 (398) | (not previously audited) | F13 (a) | NEW |

---

## 14. Cost & deployment — COVERED by F14 (the "$0.95" figure is STALE/CORRECTED)

| # | Claim | Published | Receipt | Verdict | Correct value |
|---|---|---|---|---|---|
| 14.1 | **"Frontier run cost $0.95"** | INTERP §QC ("$0.9468"); _INDEX L45; 04 (n=345 run); 06 §pitch | F14 (a)/(c) | **CORRECTED** | nominal $0.95 is logging artifact; **honest ~$14.7** (gen $9.73 + judge ~$5.0) |
| 14.2 | v1 gen logged $0.38 | cost_report.json; _INDEX L45 | F14 C1 | COVERED MATCH (but understated) | honest gen $9.73 (~25×) |
| 14.3 | v1 judge log understates ~6.7× | 06 §pitch | F14 C2 | COVERED (refined) | per-token 6.7×/10×; effective ~8.8× ⇒ ~$5.0 |
| 14.4 | Remediation run costs: panel $6.7 / deepseek-v2 ~$3.44 / evidence $0.67 / qasper $5.16 / clean $14.41 | 07 §intro ("~$8"); 08 ("$14.41"); 06 §exec | F14 (c) | COVERED MATCH | ledger total ~$31.3 |
| 14.5 | Remediation "this run ~$8" ($5.8 ds + $0.67 ev + $2.87 qasper judge + $2.29 qasper gen) | 07 §intro | F14 (c) | COVERED MATCH | components reconcile |
| 14.6 | Per-1k-question API cost (gen) closed-book/same-RAG | (06 §pitch cost table demanded; never built) | F14 C3/C4 | NEW | gpt $1.44/$6.31; claude $1.70/$16.34; deepseek $0.29/$2.20 |
| 14.7 | Latency claims | (none asserted) | F14 caveat 1 | UNCOVERED — not measured (no wall-clock logged) |
| 14.8 | Local 8B "$0 marginal / one consumer GPU" | 04 motivation; 06 conclusion | F14 (b) caveat 2 | UNCOVERED — deployment ASSUMPTION (no GPU/VRAM/throughput logged) |

> **CORRECTED 14.1 (paper-critical):** the "$0.95" frontier run cost cited in INTERPRETATION,
> _ARTIFACT_INDEX, and implied by 04/06 is a **logging artifact** — the live tracker missed
> ~3.4M resumed RAG input tokens (gen) and hardcoded a gemini judge price 6.7×/10× too low. Honest
> spend ≈ **$14.7**. F16 verifier checks E1 (logged 0.38) and E2 (honest gen $9.73 from raw token
> fields). Any "frontier cost" sentence in the paper must use the honest figure or the explicitly
> disclosed nominal. **14.7/14.8 are UNCOVERED gaps**: no latency was logged and local hardware cost
> is an assumption — neither can support a quantitative deployment claim.

---

## 15. PAPER_RUNDOWN.md (paper-facing, plain-language) — every number traced

| # | Claim (RUNDOWN) | Maps to | Verdict |
|---|---|---|---|
| 15.1 | "indistinguishable on ~3 of 4 questions" / "~77%" tie | 7.7 (tie 77%) | COVERED MATCH |
| 15.2 | RAG beats no-RAG 18/18 survive correction | 1.10 | COVERED MATCH |
| 15.3 | Fine-tuning "survives in only 7/9 cells" | 1.10 | COVERED MATCH (RUNDOWN uses correct 7/9) |
| 15.4 | RAFT "almost never beats retrieval alone (2/9)" | 1.10 | COVERED MATCH (correct 2/9) |
| 15.5 | "~1.7× more homogeneous; 0.83 vs 0.51 miss-win" | 3.7/3.3 | COVERED MATCH (1.68×; miss-win base 0.826 / QASPER STALE 0.506→0.532) |
| 15.6 | "30× size gap does not drive this" | 3.9 | COVERED MATCH |
| 15.7 | closed-book frontier fails to beat 91–96% | 6.2 (panel) | COVERED MATCH (RUNDOWN uses panel) |
| 15.8 | same docs: judge can't distinguish 8B ~77%; net 0.57–0.60 | 7.7 | COVERED MATCH (uses clean) |
| 15.9 | 1B net ~0.72–0.76 | 7.10 | COVERED MATCH |
| 15.10 | "replicates on QASPER (~0.57 net)" | 7.9 | COVERED MATCH |
| 15.11 | OOC "71% homogeneous vs 22% heterogeneous" | 4.1 | COVERED MATCH |
| 15.12 | judge-fix "barely moved" 8B ~77%, net ~0.60 vs orig 0.57 | 7.15 | COVERED MATCH |

> RUNDOWN is the **cleanest** paper-facing doc: it already uses 7/9, 2/9, 1.7×, panel 91–96%, and
> clean 0.57–0.60. No stale numbers. One residual: 15.5's "0.51 miss-win" is the **stale** QASPER
> decomp value (corrected 0.532, F03); immaterial to the "0.83 vs 0.51" contrast.

---

## 16. 06_PAPER_SKELETON.md claims register (paper-facing, committed) — COVERED

| # | Claim | Maps to | Verdict |
|---|---|---|---|
| 16.1 | 8B premium net 0.600 [0.580,0.620] base / 0.573 [0.543,0.603] QASPER ans | 7.7/7.9 | COVERED MATCH |
| 16.2 | 8B can't-distinguish ~77% ties; local decisive 1.4% | 7.7/7.14 | COVERED MATCH |
| 16.3 | Headline robust capped 0.573→clean 0.600 (Δ+0.027); ev-window 75% verbosity | 7.15/7.17 | COVERED MATCH |
| 16.4 | 1B net 0.720 [0.695,0.746] base / 0.764 QASPER; not-lose 0.44–0.55 | 7.10 | COVERED MATCH |
| 16.5 | Pooled sign tests q-clustered p≤2.6e-19 (8B)/≤4e-25 (1B) | 7.11 | COVERED MATCH |
| 16.6 | Closed-book fails-to-beat 91.4–95.8%, outright 65.9–75.0% | 6.2/6.3 | COVERED MATCH |
| 16.7 | gpt+claude-only pooled not-lose 75.2%, net 0.591 | 06 §review-4 | UNCOVERED (deepseek-excluded pool not separately recomputed) |
| 16.8 | Clustered sign test vs 8B 60:26 p=3.2e-4 (per-matchup gpt 50:8, claude 49:18, deepseek 35:21 ns) | 06 §review-3 | UNCOVERED (the *question-clustered* 60:26 pooled-3-matchup variant not in a receipt) |
| 16.9 | Lever ranking 18/18 / 7/9 / 2/9; RAFT engages Zhang et al. | 1.10 | COVERED MATCH |
| 16.10 | Mechanism 1.68×/2.8× | 3.8 | COVERED MATCH |
| 16.11 | Hazard 71% vs 22% two-corpus | 4.1 | COVERED MATCH |
| 16.12 | Corpus ~420K tokens "it FITS" (frontier-native arm) | 06 §6 | UNCOVERED (token-count of corpus-in-context not recomputed) |
| 16.13 | "28 published papers" / OpenAlex IDs (contamination-free removed) | 06 §review-2 | UNCOVERED (paper-id provenance not separately verified; F13 confirms gold provenance only) |
| 16.14 | Decoding params / temperature-drop-on-400 fallback disclosure | 06 §3 | UNCOVERED (decoding-config audit not in any receipt) |

---

## UNCOVERED claims (gaps — each is its own structured verdict)

1. **16.7** gpt+claude-only pooled not-lose 75.2% / net 0.591 — the deepseek-excluded "clean-cells"
   pool (06 §review-4) is not recomputed in any receipt. F06 recomputes the full 3-matchup pool and
   the clean-judge pool, but not the gpt+claude-only subset.
2. **16.8** question-clustered pooled sign test **60:26, p=3.2e-4** vs 8B (and per-matchup gpt 50:8 /
   claude 49:18 / deepseek 35:21 ns) — 06 §review-3's clustered records. F06 verifies the clean
   q-sign tests (98:10 etc.) but not this *panel* question-level 60:26 aggregation.
3. **16.12** corpus measured at "~420K tokens … it FITS" — the corpus token count is asserted in
   06 §6 for the proposed frontier-native arm; no receipt recomputes it.
4. **16.13** "28 *published* papers, OpenAlex/Tai-et-al. IDs" — the reframed provenance statement
   (06 §review-2). F13 verifies gold-answer provenance (Qwen vs human) but not the paper-id /
   publication-status claim that replaced "contamination-free".
5. **16.14** decoding-parameters table + "silent temperature-drop-on-400 fallback" (06 §3) — no
   receipt audits decoding configs.
6. **14.7** frontier/local **latency** — explicitly **not measured** (F14 caveat 1); any speed claim
   is unsupported by raw artifacts.
7. **14.8** local 8B **"$0 marginal / one consumer GPU"** total-cost-of-ownership — a deployment
   **assumption** (F14 caveat 2); no GPU/VRAM/power/throughput logged.

(Items 1–5 are paper-*proposed* analyses / disclosures, not yet-published headline numbers; 6–7 are
genuine measurement gaps that bound what the deployment section may claim.)

---

## CONTRADICTED claims (live source vs receipt) — each its own verdict

- **C-1 (4.8):** "OOC abstention frequency is similar (Llama 36/98 vs 42/98)" [01 §4, 00 §F4, SUPP S4]
  **vs F12**: base 62/98 vs RAG 17/98 (RAG abstains **far less**). Direction in the doc is *backwards*;
  magnitude unreproducible. The 05-audit's own rederivation (11/98 vs 25/98) is also wrong-magnitude.
  → delete the spot-check; conclusion survives (stronger).
- **C-2 (7.5/7.6):** sign-test p "≈2.4e-12 / ≈2.1e-12" [INTERP, _INDEX] **vs** exact 1.30e-24 /
  1.01e-85 [05 §B-1, F16 verifier A3]. Source script (`verify_scaled_run.py`) computes no p-values.
- **C-3 (6.7):** "Claude/DeepSeek abstain 66–72%" [SUPP S5] **vs F05**: Claude 76.8% > 72% ceiling.
- **C-4 (8.10):** "frontier abstains ~65% regardless of stratum" [SUPP S8] **vs F07**: OOC 77% > ANS 67%.
- **C-5 (12.5):** kimi flip "6.9%" [05 §C-5] **vs F15**: 9.4% (definition reproduces the other two).
- **C-6 (10.1/10.2/10.7):** QASPER position 0.496 / verbosity 0.487 / tie 0.782 [SUPP S10] **vs F10**:
  0.4953 / 0.4842 / 0.7809 (snapshot drift; verdicts unchanged).

---

## STALE numbers in historical docs (00–04, INTERPRETATION) — ledger of supersessions

These docs stay as history; the ledger records each superseded number and its corrected value.

| Doc | Stale number | Location | Corrected value | Receipt |
|---|---|---|---|---|
| 00, 01, 02, 04, _INDEX | Holm **7/12, 2/12** | 00 §F2; 01 §1,§3-summary; 02 §D; 04 §4; _ARTIFACT_INDEX L25 | **7/9, 2/9** | F01 |
| 00, 01, 04 | **"2× more homogeneous"** | 00 §F3; 01 §3; 04 §5 | **1.68×** (centroid) | F03 |
| 01, 00 (impl.), SUPP S3 | COVID decomp **0.800/0.950/0.583/0.877 (n=150)** | 01 §3; SUPP S3 | **0.812/0.931/0.573/0.864 (n=400)** | F03 |
| 01, 00 (impl.), SUPP S3 | QASPER decomp **0.241/0.852/0.506/0.589 (n=112)** | 01 §3; SUPP S3 | **0.232/0.807/0.532/0.596 (n=302)** | F03 |
| 02 §B / SUPP S10 | recall@6 COVID 0.800 / QASPER 0.241 | 02 §B; SUPP S10 | 0.812 / 0.232 | F03 |
| INTERP, 04 | closed-book **"96–99%"** headline | INTERP §Headline; 04 §6 | panel **91–96%** (1B 91.4 outside 96–99) | F05 |
| INTERP, 04 | same-RAG **"82%" 8B not-lose** | INTERP; 04 §6/abstract | panel 77.4% → clean **78.7%** | F06 |
| INTERP, 04 | same-RAG **"59%" 1B not-lose** | INTERP; 04 §6 | panel **55.7%** (CI crosses 50) → clean 54.6% | F06 |
| INTERP, 04 | **"net win-rate 0.55"** | INTERP; 04 §6/abstract | clean **0.600** | F06 |
| INTERP, _INDEX | decisive **"83% (186:38)" / "92% (423:35)"** | INTERP; _INDEX L44 | panel 74%/85.4%; clean **94%/97%** | F06 |
| INTERP, _INDEX | sign p **2.4e-12 / 2.1e-12** | INTERP; _INDEX L44 | **1.30e-24 / 1.01e-85** | 05 §B-1 / F16 verifier |
| INTERP, _INDEX, 04 | frontier run cost **"$0.95"** | INTERP §QC; _INDEX L45 | honest **~$14.7** | F14 |
| 04, INTERP | "equivalent-or-better 82% / parity / coin-flip" framing | 04 abstract; INTERP | "parity language dead"; bounded estimation net 0.57–0.62 | 06/08/F06 |
| 02 §C | "Llama/Phi truncation 0 everywhere" | 02 §C | phi COVID rag=1, lora=1; QASPER-lora=78/rag=23 | F10 / 05 §C-6 |

---

## Receipt-vs-receipt inconsistencies found

After sweeping all 15 receipts, **no contradictory recomputed numbers** exist between receipts. The
one place two receipts touch the same quantity with different *published* targets is resolved
consistently:

- **F04 vs F12 on OOC abstention frequency:** F04 explicitly **defers** the abstention-frequency
  spot-check ("not re-derived here … lives in 02 §A") and verifies only the *quality* win-rates; F12
  **owns** the frequency claim and CORRECTS it (base 62/98 vs RAG 17/98). The two are complementary,
  not contradictory — F04 never recomputed a frequency number that F12 then disputed. **Consistent.**
- **F05 vs F07 abstention classifier:** both apply the same F05 RULE-1/2/3 classifier; F05 reports
  base-corpus closed-book rates, F07 reports QASPER-arm rates. Both independently find the published
  abstention bands understated (F05: Claude 76.8% > 72; F07: OOC 0.771 > "~65%"). **Consistent
  (same direction, different corpora).**
- **F03 vs F10 on recall@6:** F10 carries the published COVID/QASPER recall (0.800/0.241) as the S10
  covariate row while F03 CORRECTS it (0.812/0.232) from current raw. F10's row mirrors the supplement
  verbatim (its job), and F10 itself flags the STALE linkage to F03 — so the divergence is *flagged*,
  not silent. Ledger marks recall@6 STALE under 3.2/3.3/10.5. **Consistent (both point to F03).**
- **F11 8B-on-HITS net 0.607 (base) vs F06 base-all net 0.600:** F11's HIT-stratum (n_q=86) is a
  subset of F06's full pool (0.5995); the stratum net differs by construction, both reproduce the
  ALL-pool 0.5995 exactly (F11 CHECK A). **Consistent.**

No receipt asserts a recomputed value another receipt contradicts.

---

## Bottom line

- **Paper-facing docs (PAPER_RUNDOWN, STATISTICAL_SUPPLEMENT, 06, 08): clean.** Every quantitative
  claim is COVERED by a receipt, with the handful of CORRECTED items (Claude abstention band, kimi
  flip rate, QASPER snapshot-drift confounds, the "~65% regardless of stratum" phrasing) all
  conservative and non-conclusion-changing. RUNDOWN and 08 already use the corrected
  panel/clean/7-9 numbers.
- **Historical docs (00–04, INTERPRETATION): carry the documented STALE set above** — chiefly the
  Holm denominators (7/12→7/9, 2/12→2/9), "2×"→1.68× homogeneity, the n=150/112 decomposition rows,
  the single-judge frontier headline (82/59/0.55/83%/92%/p≈2e-12) superseded by panel+clean, and the
  "$0.95" cost. These are flagged for history, not re-edited.
- **One live CONTRADICTION the paper must act on:** the "OOC abstention frequency is similar"
  footnote (C-1) is directionally wrong — delete it.
- **Genuine UNCOVERED gaps:** latency (never measured) and local TCO (assumed) bound the deployment
  section; five 06-proposed analyses/disclosures (gpt+claude-only pool, clustered 60:26, corpus
  token count, paper-id provenance, decoding-config) are not yet receipted.
- **No receipt-vs-receipt numeric inconsistency.**
