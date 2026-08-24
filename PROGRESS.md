# Progress

**Read this first in every session.** Last updated: 2026-08-21

---

## Where we are right now

**Phase 0 complete**, tagged `phase-0-complete`. Repository scaffolded, operating
contract written, plain-language walkthrough of the whole assignment delivered and
recall-checked (see `LEARNING.md`, `GLOSSARY.md`), Codabench registrations done.

**Phase 1 — Q1 reproducible data pipeline: complete**, tagged `phase-1-complete`. All
five sub-requirements done and verified against real data, not just designed: raw data
downloaded, ingested into a genuinely unified schema (proven via `pl.concat()`),
temporally split with the leakage invariant checked directly, persisted as a feature
store, and a one-command rebuild that's been run down both its success and failure
paths (including a real path-resolution bug found and fixed after Chaitanya questioned
whether it actually worked "from anywhere" — see error log).

**Phase 2 — Q2 BM25.** Not yet started content-wise. The Phase 1→2 recall quiz is
already done (3/3, see `LEARNING.md`) — the next session should go straight to teaching
BM25, not repeat it.

---

## ⚠️ Time budget flag (R8) — read this before Phase 2

Phases 0+1 were budgeted 5.5h combined; the real depth of work across this session —
every function verified against real data before being shown as done, several real bugs
caught in the process (entity-JSON delimiter collision, Float32/Float64 concat
mismatch, the null-propagation trap in cold-start history), multiple genuine R6 decision
points worked through with trade-offs (D3–D10) — almost certainly ran well past that
budget, even though no session tracked exact wall-clock hours against it. **Today is
2026-08-23; the deadline is 2026-08-27 — as little as 4–5 days remain**, against
Phases 2–6's combined 16h budget.

This is the deliberate trade-off `CLAUDE.md`'s prime directive asks for ("if forced to
choose between shipping faster and understanding, choose understanding and say what it
costs") — not a silent slip.

**Decision (2026-08-23):** raised plainly per R8, with a phase-by-phase risk breakdown
(Q5's cloud setup + large-bundle downloads + Codabench submission is the real deadline
risk, not teaching depth). Chaitanya chose to **keep the original full pace** for Q2
rather than scope down now, and revisit explicitly if Q2 is not keeping up with its 4h
budget. Also flagged: Q9 (anti-gaming ablation + leakage test) has no dedicated budget
line in the phase table — folding it into Q4's evaluation-harness phase, since the
leakage test's invariant is already built (Q1.3's `train_max < val_min < test_min`
check) and the ablation is evaluation-harness-adjacent work.

---

## Phase plan and budget

Deadline **27 Aug 2026**. Budget ~20 focused hours across 6 days.

| Phase | What | Budget | Status |
|---|---|---|---|
| 0 | Orientation & scaffolding | 1.5 h | ✅ done — tagged `phase-0-complete` |
| 1 | Q1 — reproducible data pipeline | 4 h | ✅ done — tagged `phase-1-complete` (ran well over budget, see note below) |
| 2 | Q2 — BM25 lexical retrieval | 4 h | 🔄 in progress, started 2026-08-23 — pacing checkpoint: revisit budget conversation if this runs over |
| 3 | Q3 — semantic retrieval (embeddings) | 3.5 h | ⬜ not started |
| 4 | Q4 — evaluation harness + Q9 (folded in, no separate budget line) | 4 h | ⬜ not started |
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
- [x] Q1.4 — feature store (`src/newsrec/build.py`, decision D9: 3 combined files).
      Verified before writing: MIND's train/dev `news.tsv` files are genuinely different
      crawls (13,956 of dev's 42,416 articles aren't in train's file at all), so the
      concat+dedupe on `article_id` is necessary, not defensive boilerplate — and for the
      28,460 articles present in both, content is identical (0 title mismatches sampled),
      so `keep="first"` is safe. Ran the full build end to end (2.81s) and verified the
      Parquet round-trip: 77,015 articles (65,238 MIND after dedupe + 11,777 EB-NeRD,
      exact expected count), 280,197 impressions, 154,714 history rows — every
      split/dataset count matches Q1.3's numbers exactly, and list/datetime dtypes
      survived the round-trip intact.
- [x] Closed a standing gap: `AI_USAGE.md`'s authorship table only tracked docs, not the
      actual `src/newsrec/*.py` files written this session — added entries for all four.
- [x] Q1.5 — one-command rebuild (`scripts/build_pipeline.py`, `configs/mind.yaml`,
      `configs/ebnerd.yaml`). Decision D10: no auto-download for either dataset (check
      raw files exist, print manual instructions and exit cleanly if not) — simpler than
      partial automation, and MIND's gating means full automation was never uniform
      anyway. Tested **both** paths for real, not just the happy one: a full clean
      `python scripts/build_pipeline.py` run (exit 0, rebuilds the feature store) and a
      deliberately broken config pointing at a nonexistent raw_root (exit 1, prints the
      actual download instructions instead of crashing with a bare traceback).
- [x] **Q1 (all five sub-requirements) is now complete.** Tagged `phase-1-complete`.

### Phase 2 (in progress)
- [x] BM25 concept taught per R1 (analogy → formula broken into named parts → reading →
      comprehension check). Three questions asked; two needed correcting and are logged in
      `LEARNING.md`: IDF cannot explain why one *document* beats another (it depends only
      on term and corpus), and `k₁=0` deletes length normalisation too, because the whole
      length bracket is multiplied by `k₁`. A follow-up confirmed BM25 damps repetition
      with a saturating rational function, **not** a log of term frequency — that's TF-IDF.
- [x] CSR/CSC re-taught from scratch on request, worked by hand on constructed corpora.
      Chaitanya's second attempt got the row structure and the empty-row case right;
      the correction was that `data[p]` pairs with `indices[p]` by tape position, not
      with the vocabulary in alphabetical order.
- [x] Decisions D11–D16 taken (see `ARCHITECTURE.md`), each against measured facts read
      off the real feature store rather than assumed — including the two that set the
      recall ceiling: **100%** of val ground-truth clicks exist in our `articles` table
      (so no artificial cap), and only **0.19% / 0.47%** are articles the user had already
      read (so D15's history exclusion costs at most half a percent).
- [x] Q2.1 — inverted index (`src/newsrec/retrieval/bm25.py`): Unicode tokeniser +
      sparse document-term matrix with BM25 document-side weights precomputed.
      **18/18 adversarial tests pass** (`tests/test_bm25_index.py`), and verified on the
      real corpora: MIND 65,238 docs / 60,951 terms / 2.36 M non-zeros in 1.70 s;
      EB-NeRD 11,777 / 31,642 / 269 K in 0.19 s; peak RSS 0.51 GB. Sanity checks that
      confirm the concept rather than just the code: `the` has IDF 0.26 against a corpus
      max of 10.68; EB-NeRD's top terms come out as the Danish function words `i`/`og`/`er`
      with `ø`/`þ` intact; and `n(t)` equals the posting-list length for all 92,593 terms.
- [x] `pytest.ini` added so `pytest` works from the repo root without `PYTHONPATH=src` —
      same class of "only works from the right directory" bug as the Q1.5 error-log entry.
- [x] `scipy==1.18.1`, `pytest==9.1.1` pinned in `requirements.txt`.

## Next step

**Q2.2/2.3/2.4 — the query side.** Build each user's query from the titles of their last
10 clicked articles (D12), score in batches against the index (D14's memory constraint —
a full dense score matrix is ~9.9 GB for MIND val alone, see `SCALE_NOTES.md`), exclude
articles already in the user's history (D15), take top-K, and report recall@{50, 100, 200}
for both datasets. Run the binary-query ablation from D16 alongside it. Watch for the
cold-start case: 1,407 MIND val users have empty history and therefore no query at all —
decide and document what they retrieve rather than letting them fall out silently.

---

<details>
<summary>Superseded next-step note from the previous session</summary>

**Phase 2 — Q2, BM25 lexical retrieval.** The Phase 1→2 recall quiz is already done
(3/3 correct, see `LEARNING.md`) — a new session does **not** need to repeat it.
Start directly with teaching BM25 (Best Match 25) itself: real-world analogy → technical
definition (term frequency, inverse document frequency, length normalisation, broken
into named parts) → required reading → comprehension check, per R1, before any code.
Pacing note: Chaitanya chose to keep full R10/R6 depth into Phase 2 rather than scope
down now (see the time-budget flag above) — revisit that conversation explicitly if Q2
is not keeping pace with its 4h budget, don't silently let it slide either way.

</details>

The pacing note above still stands: full R10/R6 depth was chosen for Q2 — revisit
explicitly if Q2 runs past its 4h budget, don't let it slide silently either way.

---

## Blocked on Chaitanya (do these early — they gate everything)

- [x] **Register on Codabench, MIND competition** — done 2026-08-21
- [x] **Register on Codabench, RecSys 2024 / EB-NeRD** — done 2026-08-21
- [x] **Download the large test bundles** — done 2026-08-24, and **verified complete**,
      not just present (the truncated-`ebnerd_demo.zip` lesson): MIND test's last line
      carries all 5 fields with `impression_id` 2,370,727 exactly equal to its line count
      and the file ends in a newline; every EB-NeRD parquet reports a row count, which a
      truncated file could not do since the schema and footer live at the *end*. Counts
      match the spec exactly (13,536,710 EB-NeRD test impressions; 2,370,727 MIND).
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
| MIND | large test | `data/raw/mind/MINDlarge_test/` | ✅ downloaded 2026-08-24, verified complete — 2,370,727 behaviors (spec: 2.37M), 120,961 news, unlabeled as expected. 1.5 GB |
| EB-NeRD | demo | `data/raw/ebnerd/ebnerd_demo/` | ✅ downloaded, valid parquet confirmed (11,777 articles, 24,724 train behaviors, 1,590 train history rows — a different, smaller bundle than the "large" one the provided notebook explored, so these counts are expected to differ) |
| EB-NeRD | small | `data/raw/ebnerd/` | ⬜ not downloaded (not needed — demo for dev, large for the leaderboard) |
| EB-NeRD | large | `data/raw/ebnerd/ebnerd_large/` | ✅ downloaded 2026-08-24, verified — 12,063,890 train + 12,566,385 validation behaviors, 125,541 articles. 3.4 GB. Schema identical to demo. |
| EB-NeRD | testset | `data/raw/ebnerd/ebnerd_testset/ebnerd_testset/` (note the **doubled** directory name, from the zip's own layout) | ✅ downloaded 2026-08-24, verified — 13,536,710 test behaviors (spec: 13.5M). 1.8 GB. Schema differs from demo — see the Phase 5 landmines under Open questions. |

Disk after these: 6.8 GB of raw data, 901 GB free — not a constraint. The `__MACOSX/`
and `DS_Store` entries inside both EB-NeRD bundles are zip packaging junk; ignore them.

**Environment:** `.venv/` created (Python 3.12.3, standard `venv`), dependencies pinned
in `requirements.txt` (`polars==1.43.2`, `pyarrow==25.0.1` — see D4 in `ARCHITECTURE.md`).
Rebuild with `python3 -m venv .venv && .venv/bin/pip install -r requirements.txt`.

---

## Open questions

- Cloud platform for large-scale runs (Kaggle vs Colab vs Lightning AI) — **deliberately
  deferred to Phase 5**, to be decided against real measured memory numbers.

### Phase 5 landmines found early (2026-08-24, while verifying the large downloads)

Neither is a bug now; both will bite in Phase 5 and are cheaper to know about than to
rediscover under deadline. Not fixed yet — Phase 5 work, logged so it isn't re-derived.

1. **EB-NeRD's test set has no `article_ids_clicked` column at all**, and
   `ingest_ebnerd.load_behaviors` selects it unconditionally. It will raise
   `ColumnNotFoundError` on the test bundle. Also absent: `article_id` (the D6 context
   field we already drop), `next_read_time`, `next_scroll_percentage`. Fails loudly,
   which is the good failure mode. **`ebnerd_large`'s schema is byte-identical to demo's**
   (same columns, same dtypes, articles too), so only the *testset* path needs the change.
2. **MIND's test set has no `-0`/`-1` click suffixes** — impression tokens are bare
   `N<digits>` (verified: 0 non-matching of 7,836,608 tokens sampled). `load_behaviors`
   degrades *correctly but silently* here: the `-[01]$` strip becomes a no-op so
   candidates are right, and the `-1` filter yields an empty clicked list for every row,
   which is exactly what D3's schema specifies for unlabeled test rows. Worth an explicit
   assertion in Phase 5 rather than relying on the coincidence.
3. **EB-NeRD test carries a new `is_beyond_accuracy` flag**: 200,000 rows true,
   13,336,710 false. That is the RecSys 2024 challenge's separately-scored
   beyond-accuracy subset — it affects the submission format, so check the Codabench
   rules before generating predictions.
4. MIND test history is 2,453 empty-history rows per 200,000 sampled (~1.2% cold-start,
   comparable to MIND-small's 2.8% in val).

---

## Error log

Every error hit, its root cause, and the fix chosen. Consult before debugging anything —
we may have already solved it.

| Date | Error | Root cause | Fix chosen | Trade-off accepted |
|---|---|---|---|---|
| 2026-08-23 | `python scripts/build_pipeline.py` run from a directory other than the repo root (e.g. `/tmp`) crashed with `FileNotFoundError: configs/mind.yaml` — found because Chaitanya questioned whether the script's `sys.path` fix really made it runnable "from anywhere" | The `sys.path.insert` fix only made the *import* of `newsrec.build` independent of the caller's working directory (via `__file__`, which always resolves to the script's real location). `Path("configs") / ...` and `OUTPUT_DIR = Path("data/processed")` were still plain relative paths, resolved against whatever the shell's cwd happened to be — an inconsistency between two parts of the same file | Introduced one `REPO_ROOT = Path(__file__).resolve().parent.parent` constant and anchored every path in the script to it — `src/`, `configs/`, `data/processed/`, and the `raw_root` value read out of each YAML config | None meaningful — this is strictly more correct with no added complexity; verified by re-running the exact `/tmp` invocation that first exposed it, plus re-checking the happy path and the failure-message path both still work |
| 2026-08-22 | Not our bug — a caveat about the provided notebook: its printed MIND train time range (`"11/10/2019 10:00 AM to 11/9/2019 9:59:58 AM"`) is chronologically wrong | The notebook computed it with plain Python `min()`/`max()` on the raw time **strings**, which compares them lexicographically (character by character), not chronologically — `"11/10/..."` sorts before `"11/9/..."` as text even though Nov 9 is earlier in time | None needed in our code — `ingest_mind.load_behaviors` already parses `time` into a real `Datetime` via `.str.strptime()`, so `.min()`/`.max()` on our `timestamp` column are correct (verified: MIND train is actually Nov 9–14, dev is Nov 15). Just don't trust the provided notebook's printed ranges at face value. | None — this only cost us noticing it before it fed into the temporal-split design |
| 2026-08-22 | `pl.concat([mind_articles_df, ebnerd_articles_df])` raised `type Float32 is incompatible with expected type Float64` (column `sentiment_score`), and the same for `impressions`' `read_time` | MIND's null placeholder columns default to `Float64` (`pl.lit(None, dtype=pl.Float64)`), but EB-NeRD's real `sentiment_score`/`read_time`/`scroll_percentage` columns are natively `Float32` in the source Parquet files - the two tables' schemas looked compatible by eye (both "float") but weren't bit-for-bit identical types | Cast all three EB-NeRD columns to `Float64` explicitly in `ingest_ebnerd.py` | None meaningful - Float64 is strictly more precise, so casting up loses nothing; found by actually running `pl.concat()` as an adversarial test (R10) rather than assuming matching column names implied matching dtypes |
| 2026-08-22 | Caught before running, not a runtime error: `ingest_mind.load_articles` originally joined `title_entities`/`abstract_entities` into one string with `pl.concat_str(..., separator="||")` | A stray literal `"||"` inside either JSON string (e.g. inside `SurfaceForms` text pulled from an article) would make a later split on `"||"` produce more than two pieces, silently corrupting that row's entity data — flagged by Chaitanya, not found by testing | Switched to `pl.concat_list([...])`, storing the two JSON strings as a genuine 2-element list column instead of a delimited string — no separator, so nothing to collide with | None meaningful — `list[str]` is the more natural Polars representation here anyway; no downside versus the string-join approach it replaced |
| 2026-08-21 | `wget`/anonymous download of MIND from `huggingface.co/datasets/yjw1029/MIND` returns HTTP 401, `x-error-code: GatedRepo` | The HF mirror is a gated repo — requires a logged-in, access-granted HuggingFace account, not just a public URL. Likely there to gate MIND's original license terms. | Chaitanya creates a free HF account, requests access (usually instant), generates a read-only access token; download resumes once shared. EB-NeRD-demo is unaffected (open S3 bucket, no gate). | Adds a manual step outside the pipeline's control before MIND ingestion can start; considered going to the official MIND site instead but that's gated the same way, so no trade-off actually avoided. — **resolved**, Chaitanya downloaded both files manually. |
| 2026-08-21 | `python -m zipfile` on the first `ebnerd_demo.zip` download raised `BadZipFile: File is not a zip file`, even though `file` identified it as a valid zip | Download was truncated mid-transfer over an unstable connection (actual size 21,187,446 bytes vs. the server's reported `Content-Length` of 21,499,083 — ~311 KB missing from the end, exactly where a zip's central directory lives). The backgrounded `wget` still reported exit code 0 despite this, so exit code alone wasn't a reliable success signal. | Chaitanya re-downloaded manually; new file's byte count matches `Content-Length` and opens cleanly with `zipfile`. | Considered `wget -c` (resume) to avoid re-pulling ~20 MB, but resume can silently fail to reconcile on an unstable connection — chose a clean re-download instead since the file is small enough that the cost difference is negligible. |
