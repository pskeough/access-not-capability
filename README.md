# Access, Not Capability
### What Retrieval Buys Small Local Models on Private-Corpus QA

What does a frontier API actually sell you on *your own documents*? Mostly access to them.
A controlled audit crossing four configurations (base, retrieval, LoRA, both) × three local
model sizes × three corpora, judged blind in both presentation orders under family-wise
error control, against three frontier models under byte-identical retrieval.

📄 **Paper:** [`paper/main.pdf`](paper/main.pdf) · Patrick S. Keough & Robert H. Tai · TMLR-targeted; arXiv preprint in preparation
📦 **Overleaf-ready source:** [`AccessNotCapability_overleaf.zip`](AccessNotCapability_overleaf.zip)

## Headline findings

- **Closed-book, the frontier loses the documents war:** it fails to beat even a 1B local
  model with retrieval in over 90% of comparisons; against the 8B, 95.8%.
- **Given identical retrieved context**, a grounded judge cannot separate the local 8B from
  the frontier in ~77% of pairs; the residual frontier premium is real but small and bounded
  (net win-rate 0.600 [0.580, 0.620]).
- **The whole corpus in a million-token window doesn't change it:** a whole-corpus frontier
  gains no larger an edge (0.561 [0.508, 0.614]) than retrieval already gives it, and
  retrieval beats full context even for the frontier itself (0.433 [0.392, 0.475]).
- **Corpus homogeneity, not corpus size, predicts retrieval's value**, and retrieval flips
  abstention behavior with corpus structure (62→100% correct abstention on the homogeneous
  corpus; 63→17% on the heterogeneous one).

## Receipts

Every quantitative claim maps to one of twenty-two receipt scripts (F01–F22) in
[`receipts/`](receipts/) and re-derives offline from raw model outputs; a 186-claim
adversarial revalidation ledger (132 exact matches, 25 corrections, 22 new results) ships
as [`CLAIMS_LEDGER.md`](CLAIMS_LEDGER.md), with the validated statistical supplement at
[`STATISTICAL_SUPPLEMENT_VALIDATED.pdf`](STATISTICAL_SUPPLEMENT_VALIDATED.pdf).

Known open gate (disclosed in the paper): the private-corpus reference answers are
LLM-drafted and LLM-judged; an external human-gold review is in progress and gates the
TMLR submission, not this working release.

## Context

Part of a research program auditing LLM behavior with psychometric method.
Program index: [Research_Collection_Patrick_Keough](https://github.com/pskeough/Research_Collection_Patrick_Keough).

## Authorship note

Drafts prepared with AI assistance under the author's direction; all research questions,
experimental design, analysis decisions, and claims are the author's own, and every
empirical claim re-derives from the receipts in this repository.

## License

Code: Apache-2.0 · Paper text and figures: CC BY 4.0 · Derived data: CC BY 4.0 (please cite)
