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

_Appended as files are created._

## Prompt log
Full session transcripts exported to `reports/ai_transcripts/` before submission.
