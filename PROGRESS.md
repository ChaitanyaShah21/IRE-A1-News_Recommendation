# Progress

**Read this first in every session.** Last updated: 2026-08-21

---

## Where we are right now

**Phase 0 complete**, tagged `phase-0-complete`. Repository scaffolded, operating
contract written, plain-language walkthrough of the whole assignment delivered and
recall-checked (see `LEARNING.md`, `GLOSSARY.md`), Codabench registrations done.

**Phase 1 — Q1 reproducible data pipeline.** In progress. Unified-schema concept taught,
schema-design decision made (D3), concrete columns defined, raw data downloaded and
verified, dev environment (`.venv` + `requirements.txt`, D4) set up. Next: write the
ingestion code (`src/newsrec/ingest_mind.py`, `src/newsrec/ingest_ebnerd.py`) that reads
the raw files and produces the three unified-schema tables — Q1.2.

---

## Phase plan and budget

Deadline **27 Aug 2026**. Budget ~20 focused hours across 6 days.

| Phase | What | Budget | Status |
|---|---|---|---|
| 0 | Orientation & scaffolding | 1.5 h | ✅ done — tagged `phase-0-complete` |
| 1 | Q1 — reproducible data pipeline | 4 h | ⬜ not started |
| 2 | Q2 — BM25 lexical retrieval | 4 h | ⬜ not started |
| 3 | Q3 — semantic retrieval (embeddings) | 3.5 h | ⬜ not started |
| 4 | Q4 — evaluation harness | 4 h | ⬜ not started |
| 5 | Q5 — scale-up & Codabench submission | 2.5 h | ⬜ not started |
| 6 | Q6/Q7 — design note & deliverables | 2 h | ⬜ not started |

Total ≈ 21.5 h against ~20 h. **Designated drop-first if we slip:** computing our own
embeddings in Phase 3 (use EB-NeRD's provided ones instead), and the extra ablations
in Phase 4 beyond what Q9 requires.

---

## Done

### Phase 0
- [x] `git init` on branch `main`; `.gitignore` blocking data, archives, checkpoints
- [x] Directory scaffold (`src/newsrec/`, `scripts/`, `tests/`, `configs/`, `notebooks/`, `reports/`, `data/`)
- [x] `CLAUDE.md` + `PROMPT.md` — the operating contract
- [x] Living documents created
- [x] Working mode packaged as a reusable skill at `~/.claude/skills/assignment/`,
      so future assignments in other folders start with the same contract, templates,
      intake interview and phase playbook. Invoke with `/assignment`.
- [x] Plain-language walkthrough of the full assignment (analogy → Q1–Q9 mapping),
      required reading logged in `LEARNING.md`, new vocabulary logged in `GLOSSARY.md`.
      Recall quiz run: temporal-split reasoning and the BM25-vs-embeddings distinction
      needed re-teaching before moving on — corrected points logged in `LEARNING.md`.

---

### Phase 1
- [x] Read both provided notebooks in full (`00_provided_mind_analysis.ipynb`,
      `00_provided_ebnerd_analysis.ipynb`) to confirm actual schemas, row counts, null
      rates — ground truth for everything below, not summary-of-a-summary
- [x] Taught "unified schema" concept (analogy → technical → recall check, 3/3 correct
      with one gap filled). Logged in `GLOSSARY.md`, `LEARNING.md`.
- [x] Schema-design decision (D3): 3 tables (articles, impressions, history), Chaitanya
      chose over a 4-table (+users) and a 2-table (inline history) alternative. Concrete
      columns defined in `ARCHITECTURE.md`.

## Next step

Plain-language walkthrough of the assignment, then the technical framing. After that,
Chaitanya's two manual actions (below) unblock the critical path.

---

## Blocked on Chaitanya (do these early — they gate everything)

- [x] **Register on Codabench, MIND competition** — done 2026-08-21
- [x] **Register on Codabench, RecSys 2024 / EB-NeRD** — done 2026-08-21
- [ ] **Accept the GitHub Classroom assignment** — **blocked: invite link not yet found
      on Moodle.** Chaitanya checked and couldn't locate it as of 2026-08-21. Does not
      block Phase 1 (local pipeline work needs no remote). Only affects where this repo
      eventually gets pushed. Check assignment page body text, announcements/forum, and
      course email before asking the instructor/classmates.

Repo URL: _not yet provided_

- [x] **Grant HuggingFace access to `yjw1029/MIND`** — done 2026-08-21, MINDsmall_train.zip
      and MINDsmall_dev.zip downloaded successfully by Chaitanya.

---

## Where the data lives

| Dataset | Bundle | Path | Status |
|---|---|---|---|
| MIND | small (train + dev) | `data/raw/mind/MINDsmall_{train,dev}/` | ✅ downloaded, row counts verified against provided notebook (51,282 / 156,965 train; 42,416 / 73,152 dev) |
| MIND | large test | `data/raw/mind/` | ⬜ not downloaded (needed only for Codabench submission, Phase 5) |
| EB-NeRD | demo | `data/raw/ebnerd/ebnerd_demo/` | ✅ downloaded, valid parquet confirmed (11,777 articles, 24,724 train behaviors, 1,590 train history rows — a different, smaller bundle than the "large" one the provided notebook explored, so these counts are expected to differ) |
| EB-NeRD | small | `data/raw/ebnerd/` | ⬜ not downloaded |
| EB-NeRD | large + testset | cloud only (too big for 7 GB RAM local) | ⬜ not downloaded |

**Environment:** `.venv/` created (Python 3.12.3, standard `venv`), dependencies pinned
in `requirements.txt` (`polars==1.43.2`, `pyarrow==25.0.1` — see D4 in `ARCHITECTURE.md`).
Rebuild with `python3 -m venv .venv && .venv/bin/pip install -r requirements.txt`.

---

## Open questions

- Cloud platform for large-scale runs (Kaggle vs Colab vs Lightning AI) — **deliberately
  deferred to Phase 5**, to be decided against real measured memory numbers.

---

## Error log

Every error hit, its root cause, and the fix chosen. Consult before debugging anything —
we may have already solved it.

| Date | Error | Root cause | Fix chosen | Trade-off accepted |
|---|---|---|---|---|
| 2026-08-21 | `wget`/anonymous download of MIND from `huggingface.co/datasets/yjw1029/MIND` returns HTTP 401, `x-error-code: GatedRepo` | The HF mirror is a gated repo — requires a logged-in, access-granted HuggingFace account, not just a public URL. Likely there to gate MIND's original license terms. | Chaitanya creates a free HF account, requests access (usually instant), generates a read-only access token; download resumes once shared. EB-NeRD-demo is unaffected (open S3 bucket, no gate). | Adds a manual step outside the pipeline's control before MIND ingestion can start; considered going to the official MIND site instead but that's gated the same way, so no trade-off actually avoided. — **resolved**, Chaitanya downloaded both files manually. |
| 2026-08-21 | `python -m zipfile` on the first `ebnerd_demo.zip` download raised `BadZipFile: File is not a zip file`, even though `file` identified it as a valid zip | Download was truncated mid-transfer over an unstable connection (actual size 21,187,446 bytes vs. the server's reported `Content-Length` of 21,499,083 — ~311 KB missing from the end, exactly where a zip's central directory lives). The backgrounded `wget` still reported exit code 0 despite this, so exit code alone wasn't a reliable success signal. | Chaitanya re-downloaded manually; new file's byte count matches `Content-Length` and opens cleanly with `zipfile`. | Considered `wget -c` (resume) to avoid re-pulling ~20 MB, but resume can silently fail to reconcile on an unstable connection — chose a clean re-download instead since the file is small enough that the cost difference is negligible. |
