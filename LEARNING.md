# Required Reading

One section per concept, added before the concept is used in code.
Each entry says what to take from the source, so you can stop reading once you have it.

---

## Phase 0 — Orientation: the assignment as a whole

Read before we started building anything, to ground the plain-language walkthrough in
the actual papers rather than a summary of a summary.

| Source | What to take from it | Time |
|---|---|---|
| MIND paper — Wu et al. 2020, §3 (Dataset) | How impressions, click history, and behaviors are structured — this is the schema Q1 unifies into | ~15 min |
| EB-NeRD paper — Kruse et al. 2024, §3 (Dataset) | Same, for the Danish dataset; note the structural differences from MIND — feeds the Q3.5 cross-dataset comparison | ~15 min |
| `notebooks/00_provided_*.ipynb` (both, skim) | Actual column names, dtypes, row counts — ground truth for what gets coded against | ~15 min |
| Robertson & Zaragoza, *The Probabilistic Relevance Framework: BM25 and Beyond* (2009), §1–2 | Where BM25 comes from and why it beats raw word-counting — needed before Phase 2's BM25 implementation, not before | ~20 min |

**Recall check (end of Phase 0):** three questions asked on why temporal (not random)
splitting is required, what distinguishes Q2 (BM25) from Q3 (embeddings), and why Q4
grades ranking metrics on top of Q2/Q3's recall@K. First two answers needed re-teaching
before Phase 0 closed — logged in the decision log / error log as a gap, not an error,
since nothing broke; corrected via re-explanation, not code. Points to remember:
- Temporal split isn't just "test on later dates" — it exists so the offline test matches
  what the model will actually know at serving time (only the past, never the future).
- BM25 and embeddings differ in *matching mechanism* (literal words vs. meaning), not in
  speed or in one being objectively better — which one wins is the empirical question
  Q3.5 asks us to answer, likely differently per slice.

---

## Phase 1 → Phase 2 recall check (2026-08-23)

Three questions before starting Phase 2: why dataset-prefix every ID, why not
random-shuffle both datasets into one split, why raw data can't be deleted once the
feature store exists. All three answered correctly on the first pass — Q2 and Q3 got a
small addition each (mixing two datasets' unrelated timelines is a second problem beyond
plain leakage; raw data is the only *irreplaceable* thing in the pipeline, since
processed data is fully derived and rebuildable from it). No re-teaching needed.

---

## Phase 1 — Unified schema

Required reading for this concept was done directly against ground truth rather than
external sources: both provided notebooks (`notebooks/00_provided_*.ipynb`) were read in
full to confirm actual column names, types, row counts, and null rates before teaching
the schema-unification concept.

**Recall check:** three questions on splitting MIND's `impressions` string, why
duplicated per-dataset logic is a problem, and which dataset needs reshaping for a
per-user history table. All three answered correctly; Q2's answer needed a concrete bug
scenario added (a tokenizer fix applied to one duplicated pipeline but not the other,
silently corrupting the Q3.5 cross-dataset comparison) — logged here, not re-taught from
scratch. Confirmed structural facts to remember:
- MIND TSVs have no header row — column names are supplied by code, not the file.
- EB-NeRD Parquet is self-describing and lazily scannable, which is how the 12M+ row
  behaviors files get explored without exhausting RAM.
- The unified schema is a **superset**: dataset-unique columns (sentiment, body text,
  entity embeddings) stay in, null for the dataset that lacks them — nothing is dropped.
