# Progress

**Read this first in every session.** Last updated: 2026-08-21

---

## Where we are right now

**Phase 0 complete**, tagged `phase-0-complete`. Repository scaffolded, operating
contract written, plain-language walkthrough of the whole assignment delivered and
recall-checked (see `LEARNING.md`, `GLOSSARY.md`), Codabench registrations done.

**Phase 1 — Q1 reproducible data pipeline.** In progress. Ingestion (Q1.2) and the
temporal split (Q1.3) are both done and verified against real data for both datasets.
Next: Q1.4, the feature store — writing the split-tagged tables out to
`data/processed/` as the actual reusable artefact Q1.5's one-command rebuild produces.

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
- [x] Environment set up (D4/D5): `.venv` + `requirements.txt`, polars 1.43.2 + pyarrow
      25.0.1. Raw data downloaded and verified against notebook-confirmed row counts.
- [x] Added R10 to the operating contract (adversarial self-check before code is done),
      applied to the skill template too. Prompted by a real bug Chaitanya caught in
      `load_articles` before it ran.
- [x] `src/newsrec/ingest_mind.py::load_articles` — MIND `news.tsv` → unified `articles`.
      Verified: 51,282 rows, correct ID prefixing, entities_raw as list[str].
- [x] `src/newsrec/ingest_mind.py::load_behaviors` — MIND `behaviors.tsv` → unified
      `impressions`. R10 checks done against real data before presenting: timestamp
      format parses MIND's single-digit hours correctly, every impression token matches
      `N<digits>-[01]` exactly (no stray dashes/spaces to confuse the suffix-stripping
      regex), no null/empty impressions field, no zero-click rows in train. Verified:
      156,965 rows, row 0 matches the provided notebook's shown example exactly.
- [x] `src/newsrec/ingest_mind.py::load_history` — MIND `behaviors.tsv` → unified
      `history` (one row per user). R10 check done *before* writing this one, not after:
      verified the D3 decision's load-bearing assumption (MIND's history string is
      identical across all of a user's impression rows) against real data — 0 of 33,617
      multi-row users had more than one distinct history string. Also hit and fixed the
      exact null-propagation trap `CLAUDE.md`'s original R9 example describes:
      `.str.split()` on a cold-start user's null history stays null, not `[]`, until
      `.fill_null([])` is applied. Verified: 50,000 unique users, 0 nulls in
      `history_article_ids`, 892 cold-start users with genuine empty lists.
- [x] **`ingest_mind.py` complete** — all three unified-schema tables (articles,
      impressions, history) now produced from MIND's raw files.
- [x] Decision D6: dropped EB-NeRD's context `article_id` field (which article page a
      recommendation module was shown on) from the unified schema — found while
      inspecting real EB-NeRD data, not part of the original D3 column list. Nothing in
      Q1–Q9 needs it.
- [x] `src/newsrec/ingest_ebnerd.py` — all three unified-schema functions
      (`load_articles`, `load_behaviors`, `load_history`), built on `pl.scan_parquet`
      (lazy) with `.collect()` at the end. `load_history` needed no row-collapsing —
      EB-NeRD's history file is already one row per user, confirmed 1:1 against
      behaviors' unique user count on real demo data (1,590 = 1,590).
- [x] **`ingest_ebnerd.py` complete.** **Unified schema empirically verified**, not just
      designed: `pl.concat()`'d all three MIND/EB-NeRD table pairs and confirmed they
      combine cleanly — caught and fixed a real Float32/Float64 mismatch in the process
      (error log above). Combined: 63,059 articles, 181,689 impressions, 51,590 users'
      history.
- [x] Q1.3 — temporal split (`src/newsrec/temporal_split.py`). Verified neither dataset
      gives 3 labeled partitions at demo/small scale (D7); decided to carve val+test from
      dev/validation's tail (D7), 70/30 by row-count timestamp cutoff, uniform across
      both datasets (D8). Found and handled a real subtlety: EB-NeRD's train and
      validation history files genuinely differ per user, not the same data re-served —
      `add_history_split` reuses validation's history table for both new val/test
      sub-partitions rather than treating them as needing separate history data that
      doesn't exist. Verified on real data: row counts sum correctly, ~70.0% val ratio
      for both datasets, and the leakage invariant (`train_max < val_min`,
      `val_max < test_min`) holds strictly for both — this check is the seed of Q9's
      required `tests/test_no_leakage.py`.

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

Repo URL: _not yet provided_ (GitHub Classroom, still blocked — see above)

**Private backup remote** (separate from GitHub Classroom): `origin` →
`git@github.com:ChaitanyaShah21/IRE-A1-News_Recommendation.git` (private), set up
2026-08-22 via SSH (key already registered with GitHub, confirmed via `ssh -T`). Pushed
`main` + `phase-0-complete` tag. Push after every commit going forward, same as local
git discipline (R13) — this isn't a substitute for the eventual GitHub Classroom repo,
just insurance until that's found.

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
| 2026-08-22 | Not our bug — a caveat about the provided notebook: its printed MIND train time range (`"11/10/2019 10:00 AM to 11/9/2019 9:59:58 AM"`) is chronologically wrong | The notebook computed it with plain Python `min()`/`max()` on the raw time **strings**, which compares them lexicographically (character by character), not chronologically — `"11/10/..."` sorts before `"11/9/..."` as text even though Nov 9 is earlier in time | None needed in our code — `ingest_mind.load_behaviors` already parses `time` into a real `Datetime` via `.str.strptime()`, so `.min()`/`.max()` on our `timestamp` column are correct (verified: MIND train is actually Nov 9–14, dev is Nov 15). Just don't trust the provided notebook's printed ranges at face value. | None — this only cost us noticing it before it fed into the temporal-split design |
| 2026-08-22 | `pl.concat([mind_articles_df, ebnerd_articles_df])` raised `type Float32 is incompatible with expected type Float64` (column `sentiment_score`), and the same for `impressions`' `read_time` | MIND's null placeholder columns default to `Float64` (`pl.lit(None, dtype=pl.Float64)`), but EB-NeRD's real `sentiment_score`/`read_time`/`scroll_percentage` columns are natively `Float32` in the source Parquet files - the two tables' schemas looked compatible by eye (both "float") but weren't bit-for-bit identical types | Cast all three EB-NeRD columns to `Float64` explicitly in `ingest_ebnerd.py` | None meaningful - Float64 is strictly more precise, so casting up loses nothing; found by actually running `pl.concat()` as an adversarial test (R10) rather than assuming matching column names implied matching dtypes |
| 2026-08-22 | Caught before running, not a runtime error: `ingest_mind.load_articles` originally joined `title_entities`/`abstract_entities` into one string with `pl.concat_str(..., separator="||")` | A stray literal `"||"` inside either JSON string (e.g. inside `SurfaceForms` text pulled from an article) would make a later split on `"||"` produce more than two pieces, silently corrupting that row's entity data — flagged by Chaitanya, not found by testing | Switched to `pl.concat_list([...])`, storing the two JSON strings as a genuine 2-element list column instead of a delimited string — no separator, so nothing to collide with | None meaningful — `list[str]` is the more natural Polars representation here anyway; no downside versus the string-join approach it replaced |
| 2026-08-21 | `wget`/anonymous download of MIND from `huggingface.co/datasets/yjw1029/MIND` returns HTTP 401, `x-error-code: GatedRepo` | The HF mirror is a gated repo — requires a logged-in, access-granted HuggingFace account, not just a public URL. Likely there to gate MIND's original license terms. | Chaitanya creates a free HF account, requests access (usually instant), generates a read-only access token; download resumes once shared. EB-NeRD-demo is unaffected (open S3 bucket, no gate). | Adds a manual step outside the pipeline's control before MIND ingestion can start; considered going to the official MIND site instead but that's gated the same way, so no trade-off actually avoided. — **resolved**, Chaitanya downloaded both files manually. |
| 2026-08-21 | `python -m zipfile` on the first `ebnerd_demo.zip` download raised `BadZipFile: File is not a zip file`, even though `file` identified it as a valid zip | Download was truncated mid-transfer over an unstable connection (actual size 21,187,446 bytes vs. the server's reported `Content-Length` of 21,499,083 — ~311 KB missing from the end, exactly where a zip's central directory lives). The backgrounded `wget` still reported exit code 0 despite this, so exit code alone wasn't a reliable success signal. | Chaitanya re-downloaded manually; new file's byte count matches `Content-Length` and opens cleanly with `zipfile`. | Considered `wget -c` (resume) to avoid re-pulling ~20 MB, but resume can silently fail to reconcile on an unstable connection — chose a clean re-download instead since the file is small enough that the cost difference is negligible. |
