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
