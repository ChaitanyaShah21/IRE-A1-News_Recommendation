# AI Usage Log

Required by assignment Q7.4: all prompts, chat history, and marking of AI-generated
versus human-written code.

## Tool used
Claude Code (Anthropic), operating under the contract in `CLAUDE.md`. Model varies by
session (Opus 5 for Phase 0 scaffolding, Sonnet 5 from the Phase 1 session onward per
Chaitanya's `/model` choice) — noted per-commit in git history, not tracked here.

## Authorship marking

| File | Authorship | Notes |
|---|---|---|
| `CLAUDE.md` | AI-generated from human specification | Chaitanya specified every rule (incl. R10, added 2026-08-22 after he asked how to catch such bugs automatically); wording is AI |
| `PROMPT.md` | AI-generated | Verbatim copy of `CLAUDE.md` |
| `ARCHITECTURE.md` | AI-generated, human-decided | All decisions chosen by Chaitanya |
| `PROGRESS.md` | AI-maintained | |
| `GLOSSARY.md` | AI-generated | |
| `LEARNING.md` | AI-generated | Reading list + recall-check outcomes |
| `.gitignore` | AI-generated | |
| `requirements.txt` | AI-generated, human-decided | Chaitanya chose venv+requirements.txt over Poetry/conda (D4) |
| `notebooks/00_provided_*.ipynb` | **Supplied with the assignment** | Not our work; reference only |
| `data/raw/**`, `data/processed/**` | Not authored — downloaded/generated data | Gitignored, never committed |
| `src/newsrec/ingest_mind.py` | AI-generated, human-decided | Every schema/design choice (D3, D5) chosen by Chaitanya; one bug (entity-JSON delimiter collision) and one type fix (subcategory) caught by Chaitanya before/while running |
| `src/newsrec/ingest_ebnerd.py` | AI-generated, human-decided | Context-`article_id` inclusion (D6) decided by Chaitanya |
| `src/newsrec/temporal_split.py` | AI-generated, human-decided | Split-source and split-ratio decisions (D7, D8) chosen by Chaitanya |
| `src/newsrec/build.py` | AI-generated, human-decided | Feature-store layout (D9) chosen by Chaitanya |
| `scripts/build_pipeline.py` | AI-generated, human-decided | No-auto-download design (D10) chosen by Chaitanya |
| `configs/mind.yaml`, `configs/ebnerd.yaml` | AI-generated | Raw-data paths, per D10/the Phase 0 "no hardcoded paths" layout decision |
| `src/newsrec/retrieval/bm25.py` | AI-generated, human-decided | Tokenisation, parameters, implement-vs-import, and query-term weighting (D11, D13, D14, D16) chosen by Chaitanya |
| `tests/test_bm25_index.py` | AI-generated | R10 adversarial suite, 18 tests; the Danish-tokenisation case was recorded as a requirement in D11 before the code existed |
| `pytest.ini` | AI-generated | Puts `src/` on the path so `pytest` works from the repo root |
| `src/newsrec/retrieval/bm25_search.py` | AI-generated, human-decided | Query construction, cold-start handling, and the availability variant (D12, D15, D16, D17, D19) chosen by Chaitanya, who also challenged whether D19 was permitted by the spec — the compliance argument now in D19 exists because he asked |
| `src/newsrec/eval/recall.py` | AI-generated, human-decided | Macro-vs-micro averaging (D18) chosen by Chaitanya |
| `tests/test_bm25_search.py` | AI-generated | R10 adversarial suite, 20 tests, batch-invariance included |
| `scripts/run_bm25_recall.py` | AI-generated | Thin entry point for Q2.4 |
| `scripts/summarise_bm25_recall.py` | AI-generated | Adds the random-baseline column without which the D19 comparison misleads |
| `reports/bm25_recall_*.csv` | Generated output | Not authored; reproducible via the two scripts above |
| `requirements.txt` (Phase 3 additions) | AI-generated, human-decided | D20's route chosen by Chaitanya, who asked the question that settled it — whether using a library here conflicts with the assignment. The CPU-index lines exist because the naive install was *measured* at 2,894 MB, 2,238 MB of it unusable CUDA |
| `src/newsrec/retrieval/semantic.py` | AI-generated, human-decided | Model, search method and storage layout (D20, D21, D22) chosen by Chaitanya; the library-vs-implement question was raised by him and the compliance argument in D20 exists because he asked it |
| `tests/test_semantic_embeddings.py` | AI-generated | R10 adversarial suite, 15 tests. The two null-representation cases (MIND null vs EB-NeRD blank) were found by checking the real store before writing the code, not after a traceback |
| `scripts/build_embeddings.py` | AI-generated | Thin entry point for Q3.1; kept out of build_pipeline.py per D22 |
| `data/processed/embeddings.parquet` | Generated output | Not authored; reproducible via the script above. Gitignored |
| `src/newsrec/retrieval/semantic_search.py` | AI-generated, human-decided | Q3.3 mean pooling and the D19 two-pool constraint follow Chaitanya's earlier decisions; the -inf masking fix and the MIN_NORM threshold were found by mutation-testing the tests, not by a failure |
| `tests/test_semantic_search.py` | AI-generated | R10 adversarial suite, 21 tests, built so the wrong implementation visibly fails - verified by deliberately reintroducing each bug and confirming the tests catch it |
| `src/newsrec/retrieval/availability.py` | AI-generated | D19 logic extracted from run_bm25_recall.py so both retrievers share one implementation; regression-verified against the committed Q2 numbers |
| `scripts/run_semantic_recall.py` | AI-generated | Thin entry point for Q3.4, deliberately the same shape as the BM25 runner so Q3.5 compares methods not harnesses |
| `scripts/summarise_recall.py` | AI-generated, human-edited history | Was `summarise_bm25_recall.py`; generalised to both methods for Q3.5. The random-baseline column it adds exists because Chaitanya's D19 discussion established that absolute recall alone misleads |
| `reports/semantic_recall_*.csv`, `reports/recall_summary.csv` | Generated output | Not authored; reproducible via the scripts above |
| `CLAUDE.md` / `PROMPT.md` (R1 amendment) | AI-written, human-directed | Chaitanya directed the change: drop required reading, teach in chat plain-to-technical. Rationale and stated cost written by Claude |

| `src/newsrec/eval/metrics.py` | AI-generated, human-decided | Q4.1 AUC/MRR/nDCG. The tie policy (D23) was chosen by Chaitanya from measured tie rates; AUC uses the rank identity rather than the O(P·N) pair loop |
| `tests/test_metrics.py` | AI-generated | R10 adversarial suite, 28 tests. Verified by mutation testing: 7 deliberate bugs reintroduced (uncapped IDCG, inverted tiebreak, `rankdata` tie method, removed NaN guard, argsort-stability tiebreak, MRR off-by-one, DCG discount off-by-one) and every one was caught |

_Appended as files are created._

## Prompt log
Full session transcripts exported to `reports/ai_transcripts/` before submission.
