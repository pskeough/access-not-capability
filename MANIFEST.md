# MANIFEST — access-not-capability export

Curated allowlist export, staged 2026-07-23. Source project: `C:\Research\Train_LLM`
(originals untouched). Secrets scan: clean.

## Provenance gate

Manuscript is the reviewed 2026-07-23 build of canonical `Final_Validation\latex\main.tex`:
a pre-release audit pass corrected four receipt-mismatch issues, and the abstract,
introduction, and conclusion were editorially revised with every number and citation
verified unchanged against the receipts. PDF compiled with tectonic 2026-07-23; verified:
all corrected claims present, no dangling refs.

## Copied

| Export path | Source |
|---|---|
| paper/ (main.tex, main.pdf, references.bib, percell_stats_table.tex, figures/) | ...\Final_Validation\latex\ |
| receipts/ (F01–F22, 22 receipt documents) | ...\Final_Validation\receipts\ |
| STATISTICAL_SUPPLEMENT_VALIDATED.pdf (fresh 2026-07-12 build incl. F18–F22 sections) | ...\ResearchMasterContext\projects\train-llm\assets\stats\ |
| CLAIMS_LEDGER.md (186-claim revalidation ledger) | ...\ResearchMasterContext\projects\train-llm\assets\stats\ |
| AccessNotCapability_overleaf.zip (forward-slash paths, rebuilt from current tex) | rebuilt 2026-07-23 |

## Deliberately excluded

- Raw private-corpus model outputs (data-availability terms: private corpus stays private)
- `FinalRunPack\STATISTICAL_SUPPLEMENT.pdf` (pre-correction decoy flagged by gauntlet C3)
- Recompute scripts under `scripts/` (available on request pending packaging cleanup; the
  receipts documents themselves ship here)
- Superseded tex backups and the older `overleaf_export/`
