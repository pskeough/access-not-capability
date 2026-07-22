# F13 — Eval-Set Integrity (GOLD receipt)

**Family:** F13 (NEW) — eval-set integrity: dedup/near-dup tables, SHA-256 hash-locks, gold
provenance, question_type distributions across the three eval sets.

**Recompute script:** `C:/Research/Train_LLM/Final_Validation/scripts/F13_eval_set_integrity.py`
(offline, idempotent, no API). Ran clean, **exit 0, 39/39 checks PASS**.

**Raw inputs (only sources trusted):**
- Eval sets:
  - base `C:/Research/Train_LLM/experiments/v4_raft_cot/data/eval_set.jsonl` (n=345)
  - COVID `C:/Research/Train_LLM/experiments/v5_raft_cot/corpora/covidqa/eval_set.jsonl` (n=400)
  - QASPER `C:/Research/Train_LLM/experiments/v5_raft_cot/corpora/qasper/eval_set.jsonl` (n=400)
- SHA-256 sidecars: `eval_set.sha256` next to each eval set.
- Config snapshots (recorded hashes in `extras.eval_set.sha256`): one per model per corpus, e.g.
  `.../artifacts/runs_minicpm1b/20260531-111249/config_snapshot.json`,
  `.../artifacts/runs/20260531-010503/config_snapshot.json`,
  `.../artifacts/runs_phi4mini/20260601-122450/config_snapshot.json`,
  `.../artifacts/covidqa/{llama8b/20260605-230052,minicpm1b/20260606-120325,phi4mini/20260606-105705}/config_snapshot.json`,
  `.../artifacts/qasper/{llama8b/20260603-090219,minicpm1b/20260603-153951,phi4mini/20260603-135816}/config_snapshot.json`.

**Normalization** (for dedup): lowercase, strip, collapse internal whitespace, strip trailing
`? . ! : ; ,`. Exact-match after normalization = "duplicate text"; this captures the
case/punctuation/whitespace near-dups the audit flagged.

---

## (a) Dedup / near-dup

### Per-claim verdicts

| Claim | Published (cite) | Recomputed | Verdict |
|---|---|---|---|
| base qids | 345 (05_VERIFICATION_AUDIT.md §3; SUPP. S? "315/345") | 345 | MATCH |
| base unique texts | 315 (STATISTICAL_SUPPLEMENT.md L161; 05_VERIFICATION_AUDIT.md §3; 06_PAPER_SKELETON.md L98) | 315 | MATCH |
| base OOC qids | 52 (SUPP. L62; AUDIT §3) | 52 | MATCH |
| base OOC unique texts | 27 (SUPP. L62; AUDIT §3; F04 receipt L92) | 27 | MATCH |
| base OOC multiplicity | task spec: one ×6, two ×4, three ×3, eight ×2; F04 receipt `{1×13,2×8,3×3,4×2,6×1}` | `{6:1, 4:2, 3:3, 2:8, 1:13}` (i.e. 1 text×6, 2×4, 3×3, 8×2, 13 singletons) | MATCH |
| base answerable duplicate pairs | "5 more duplicate pairs among answerable" (AUDIT §3) | 5 groups, each size-2 (5 extra qids) | MATCH |
| base paraphrase triplets | 11 (06_PAPER_SKELETON.md L98) | 11 groups, all size 3 (`paraphrase_group_id`); 33 `question_type==paraphrase` rows = 11×3 | MATCH |
| QASPER OOC qids/unique | 98 / 94 (F04 receipt L93) | 98 / 94 | MATCH |
| COVID dedup | not previously checked | n=400, **398 unique**, 2 dup qids (2 pairs) | NEW |
| QASPER dedup | not previously checked | n=400, **381 unique**, 19 dup qids; mult `{2:7, 3:3, 4:2}` | NEW |

**Reconciliation (independent cross-check):** 345 − OOC_extra(25) − answerable_extra(5) = **315 unique**. Confirmed.

### base OOC repeated-text groups (qid lists)
- ×6: q0309, q0327, q0336, q0337, q0340, q0341 — "key factors influencing mentorship satisfaction…"
- ×4: q0310, q0324, q0338, q0343 — "recommended field training protocols for marine ecology…"
- ×4: q0313, q0323, q0325, q0328 — "how effective are VR simulations… industrial chemists…"
- ×3: q0301, q0321, q0330 — "specific requirements for board certification…"
- ×3: q0305, q0322, q0342 — "primary factors contributing to burnout among public health…"
- ×3: q0318, q0331, q0335 — "most effective strategies for communicating climate science…"
- ×2 (eight groups): {q0300,q0332}, {q0303,q0326}, {q0306,q0329}, {q0307,q0345}, {q0312,q0334}, {q0315,q0339}, {q0316,q0333}, {q0320,q0344}
- ×1: 13 singletons.
Totals: 1+2+3+8 = 14 multi-occurrence texts + 13 singletons = **27 unique**; 6+4+4+3+3+3+2·8 = 39 + 13 = **52 qids**.

### base answerable (non-OOC) duplicate pairs (5)
- {q0075, q0125} — "average grade difference in college…"
- {q0008, q0076} — "proportion of students followed through…"
- {q0176, q0181} — "what does 'collaborating' refer to…"
- {q0035, q0130} — "estimated probability of a high mathematics achiever…"
- {q0091, q0103} — "why did the researchers choose broad categories of professional…"

### NEW — QASPER duplicate groups
QASPER eval set carries non-trivial duplication previously unaudited: **19 of 400 qids are
duplicate text** (381 unique), with 7 pairs (×2), 3 triplets (×3) and 2 quartets (×4). COVID is
near-clean (398/400, 2 pairs). The paper's dedup disclosure should extend to QASPER, not just base.

---

## (b) SHA-256 hash-locks

All three eval sets hash-match their `.sha256` sidecar **and** the `extras.eval_set.sha256` field
in every run config snapshot that references them (3 snapshots per corpus). Verdict: **MATCH** (9/9 snapshot checks + 3/3 sidecar checks).

| Corpus | SHA-256 (file = sidecar = all snapshots) |
|---|---|
| base | `b11170b6822c97d1c1dd53bf534ac9d4d0260cd69deb4bf59ab9a997a15fd2de` |
| COVID | `54c4a3d1f5525e4585a36ff8f16791d036ac1d25caedf1acd83f6d575445c729` |
| QASPER | `26815e8b78abd4ae6bc98ad29f7792f0886d5f2faec4bd92cd6472b0df605c08` |

The base hash is recorded identically in the llama8b, minicpm1b and phi4mini base-run snapshots;
COVID and QASPER hashes are recorded identically across their three per-model run snapshots.

---

## (c) Gold provenance

| Corpus | n | `generated_by` (exact) | `generated_at` (exact) | Provenance | Verdict |
|---|---|---|---|---|---|
| base | 345 | `qwen2.5-14b-instruct-q4_k_m.gguf` | per-row ISO timestamps `2026-05-24T11:54…12:12Z` (gen run) | LLM-generated gold (Qwen-2.5-14B) | MATCH |
| COVID | 400 | `covidqa_human_annotators` | `covid-qa-deepset` | Human gold (deepset COVID-QA) | MATCH |
| QASPER | 400 | `qasper_human_annotators` | `qasper-v0.3` | Human gold (QASPER v0.3) | MATCH |

Matches published provenance (STATISTICAL_SUPPLEMENT.md L162; _ARTIFACT_INDEX.md L37: "base
qwen2.5-14b; COVID/QASPER human"). The base set carries a single `generated_by` value across all
345 rows; COVID/QASPER each carry a single human-annotator tag.

---

## (d) question_type distributions

| Corpus | single_hop | multi_hop | out_of_corpus | paraphrase | definitional | total | Verdict |
|---|---|---|---|---|---|---|---|
| base | 172 | 70 | 52 | 33 | 18 | 345 | MATCH/NEW |
| COVID | 400 | — | — | — | — | 400 | NEW |
| QASPER | 177 | 125 | 98 | — | — | 400 | NEW |

Base proportions track the config-snapshot target distribution (single 0.50, multi 0.20,
paraphrase 0.10, OOC 0.15, definitional 0.05). COVID is entirely single_hop; QASPER mixes
single_hop/multi_hop/out_of_corpus.

---

## Clean dedup-disclosure table (paper-ready)

| Eval set | qids | Unique texts | Dup qids | Multiplicity (texts × occurrences) | OOC qids → unique | Gold provenance | SHA-256 (locked) |
|---|---|---|---|---|---|---|---|
| **base** | 345 | **315** | 30 | OOC `{6×1,4×2,3×3,2×8}` + answerable 5 pairs; 11 paraphrase triplets | 52 → **27** | Qwen-2.5-14B (LLM) | `b11170b6…fd2de` |
| **COVID** | 400 | **398** | 2 | 2 pairs | n/a (all single_hop) | human (covid-qa-deepset) | `54c4a3d1…45c729` |
| **QASPER** | 400 | **381** | 19 | `{2×7, 3×3, 4×2}` | 98 → **94** | human (qasper-v0.3) | `26815e8b…605c08` |

---

## Verdict summary
All published F13 claims **MATCH**. Two **NEW** facts surfaced by extending the dedup scan to
the previously-unchecked corpora: QASPER has 19 duplicate qids (381/400 unique) and COVID has 2
(398/400). Hash-locks are intact across every run snapshot. No corrections required.
