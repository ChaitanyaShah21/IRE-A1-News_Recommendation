# Progress

**Read this first in every session.** Last updated: 2026-08-21

---

## Where we are right now

**Phase 0 — Orientation & Scaffolding.** Nearly done.
Repository scaffolded, operating contract written, plain-language walkthrough of the
whole assignment delivered and recall-checked (see `LEARNING.md`, `GLOSSARY.md`). Next:
unblock the three manual actions below, then start Phase 1 (Q1 pipeline).

---

## Phase plan and budget

Deadline **27 Aug 2026**. Budget ~20 focused hours across 6 days.

| Phase | What | Budget | Status |
|---|---|---|---|
| 0 | Orientation & scaffolding | 1.5 h | 🔄 in progress |
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

---

## Where the data lives

Nothing downloaded yet. Local machine has no dataset files as of 2026-08-21.

| Dataset | Bundle | Path | Status |
|---|---|---|---|
| MIND | small (train + dev) | `data/raw/mind/` | ⬜ not downloaded |
| MIND | large test | `data/raw/mind/` | ⬜ not downloaded |
| EB-NeRD | demo | `data/raw/ebnerd/` | ⬜ not downloaded |
| EB-NeRD | small | `data/raw/ebnerd/` | ⬜ not downloaded |
| EB-NeRD | large + testset | cloud only (too big for 7 GB RAM local) | ⬜ not downloaded |

---

## Open questions

- Cloud platform for large-scale runs (Kaggle vs Colab vs Lightning AI) — **deliberately
  deferred to Phase 5**, to be decided against real measured memory numbers.

---

## Error log

Every error hit, its root cause, and the fix chosen. Consult before debugging anything —
we may have already solved it.

_(empty so far)_

| Date | Error | Root cause | Fix chosen | Trade-off accepted |
|---|---|---|---|---|
