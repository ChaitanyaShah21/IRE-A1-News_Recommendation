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

## Phase 2 — BM25 (Best Match 25) and the inverted index

Issued 2026-08-23, before any Phase 2 code. Read in this order — each one assumes the
previous.

| Source | What to take from it | Time |
|---|---|---|
| Manning, Raghavan & Schütze, *Introduction to Information Retrieval*, §1.1–1.2 (free online) | What an inverted index *is* as a data structure — the term → posting-list mapping, and why you never score a query by scanning every document. Stop once you can draw the index for a 3-document toy corpus. | ~15 min |
| Same book, §6.2 (term frequency & weighting) then §11.4.3 (Okapi BM25) | Why raw term counts are a bad relevance score, how inverse document frequency fixes part of it, and then the exact BM25 formula with `k1` and `b`. §11.4.3 is two pages — that's the core of Q2. | ~25 min |
| Robertson & Zaragoza, *The Probabilistic Relevance Framework: BM25 and Beyond* (2009), §1–2 | Where BM25 comes from — that it's a *probabilistic* model of relevance, not a hand-tuned heuristic that happens to work. Read for the intuition, not the derivations. | ~20 min |

Concepts taught before this reading was issued (analogy → technical definition → check):
term frequency saturation, inverse document frequency, document-length normalisation,
`k1`/`b`, and the retrieval-vs-re-ranking distinction restated in BM25 terms.

**Comprehension check (2026-08-24), three questions.** Right conclusions, wrong
mechanisms twice — both corrected with worked arithmetic before any code was written:

- **IDF cannot explain why one *document* outranks another.** Its formula contains only
  `N` and `n(t)` — corpus and term, no `D` at all — so for a given query term it is one
  constant multiplying every document's score equally. IDF differentiates between
  *terms*, not between documents. What separated the two documents in the question was
  the length bracket: three mentions in 12 tokens scores 1.72, three in 60 tokens scores
  1.10 (`avgdl`=20, `k1`=1.5, `b`=0.75) — same numerator, all the difference in the
  denominator.
- **`k1 = 0` deletes length normalisation as well as saturation.** The whole length
  bracket is multiplied by `k1`, so zeroing it makes `b` unreachable. The score collapses
  to `sum of IDF(t)` over query terms present: length-blind and repetition-blind, not
  "shorter documents win".
- **BM25 does not take a log of term frequency — that's TF-IDF.** BM25 damps repetition
  with a saturating rational function `f(k1+1)/(f + k1*...)`, which has a hard ceiling of
  `k1+1`; a log grows forever, just slowly. The only logarithm in BM25 is in IDF. Worked:
  3x the mentions buys 1.44x, and infinite mentions would buy only 1.84x.
- The third question (why a 300-title query retrieves worse than a 10-title one) was not
  attempted, and is not derivable from the formula — which was the point of asking. Taught
  instead: **BM25 contains no time term of any kind**, so topic drift cannot be tuned
  away and must be handled in query construction or not at all. This became D12.

**CSR/CSC re-taught from scratch on request (2026-08-24)**, worked by hand on constructed
corpora rather than described. The second attempt got the row structure and the empty-row
case right; the correction was that `data[p]` pairs with `indices[p]` **by tape
position**, not with the vocabulary in alphabetical order. Three arrays, three jobs:
`data` = what, `indices` = which column, `indptr` = which row. Points to remember:
- `indptr` holds **positions into `data`/`indices`**, never column numbers, and has one
  more entry than there are rows — n regions need n+1 boundaries.
- Document numbers repeat freely across `indices`; each term's run is an independent list.
- `indptr[j+1] - indptr[j]` is the posting-list length, which **is** `n(t)`, which feeds
  IDF — the data structure and the formula describing the same thing from two directions.

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
