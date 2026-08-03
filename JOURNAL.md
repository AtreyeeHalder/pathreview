## Week 7 — Issue selection

**Issue link:** [Issue 109 link](https://github.com/ascherj/pathreview/issues/109)

**Issue title:** Test coverage for core/services/review_service.py is below 40%

**Tier:** [ ] Tier 1  [x] Tier 2  [ ] Tier 3

**Problem summary:**

`core/services/review_service.py` is the service that runs the whole review workflow, but right now most of it has no tests; coverage sits at about 22% (as seen from running `.venv/Scripts/python -m pytest tests/unit/test_review_service.py -m unit --cov=core.services.review_service --cov-report=term-missing`), which is below 40% as Issue 109 suggests. The three simple functions (`create_review`, `get_review`, `list_reviews`) have tests, but the main orchestrator function `process_review` and the four helper functions it calls are completely untested, so nothing is actually checking that the review pipeline behaves correctly. This is risky because `process_review` can end in several different ways: everything succeeds and the review is marked "complete", a step fails cleanly and it is marked "failed", or something crashes unexpectedly and it still has to fail gracefully. A successful fix means adding unit tests that exercise these success, partial-failure, and full-failure paths so the file's coverage rises well above 40% and future changes to this critical service are protected from silent breakage.

**"Is this right for me?" checklist scope reasoning:**

- **Done looks like:** Before, `review_service.py` sits at ~22% coverage and `process_review` plus its `_run_*` helpers have no tests. After, new unit tests cover the "complete" success path, the clean "failed" paths, and the unexpected-exception path, pushing coverage above 40%.
- **Tier fit:** Tier 2 is a realistic match. The work spans two files and I need to follow how `process_review` coordinates the ingestion/agent/RAG/ safety steps. My goal is to test the existing code in `review_service.py`.
- **Codebase readiness:** I have read the actual functions in `core/services/review_service.py` and read the existing tests in `tests/unit/test_review_service.py` end-to-end, so I can reuse their fixtures and mocking patterns.
- **Scope and time:** Estimated 5–7 hours, no blockers or dependencies. This is achievable before the Week 9 deadline.
- **Out of scope:** 13 pre-existing tests fail from an `AsyncMock`/`Mock` mismatch on Python 3.14; they were broken before I started and are not part of Issue 109, so I am leaving them for now unless a mentor says otherwise.

**Branch name:** `test/109-review-service-test-coverage`

**Setup confirmation:** [x] App runs locally at localhost:5173

**Cohort ledger:** [x] Issue added to cohort ledger

---

## Week 8 — Reproduction & solution planning

**Reproduction commit link:** https://github.com/AtreyeeHalder/pathreview/commit/16157b9055cfc39d98198a5a06b6ed5e9902c847

**Reproduction summary:**
I reproduced the issue by running `.venv/Scripts/python -m pytest tests/unit/test_review_service.py -m unit --cov=core.services.review_service --cov-report=term-missing` and I observed coverage at 22% (below the 40% threshold from Issue 109). The uncovered lines confirm the gap is exactly where the issue says: `process_review` (98-194) and the `_run_*` helpers (202-279) have no tests, while only the three simple CRUD functions (`create_review`, `get_review`, `list_reviews`) are exercised.

**PLAN.md link:** https://github.com/AtreyeeHalder/pathreview/blob/test/109-review-service-test-coverage/PLAN.md

**Walkthrough video (recommended):** N/A

**Blockers or open questions:**
N/A

---

## Week 9 — Solution building & PR submission

### Check-in 1 (mid-week)

**Current progress:**
Sub-task 1 (Baseline) from PLAN.md is done: I re-ran `.venv/Scripts/python -m pytest tests/unit/test_review_service.py -m unit --cov=core.services.review_service --cov-report=term-missing` and confirmed the "before" number is **22%**, with `process_review` (98-194) and the `_run_*` helpers (202-279) still uncovered. I've also finalized the mocking approach that avoids the pre-existing `AsyncMock`/`Mock` trap — returning a plain `Mock()` result and setting `.scalars.return_value.first.return_value` explicitly — so my new tests won't inherit the 13 existing failures. The `process_review` and `_run_*` tests themselves (sub-tasks 2–5) are not written yet; the test file currently still only covers `create_review`, `get_review`, and `list_reviews`.

**Next steps:**
Work through PLAN.md sub-tasks 2–6: add the helper unit tests for `_run_safety_checks`, `_run_agent_orchestration`, `_run_rag_retrieval_generation`, and `_run_ingestion_pipeline`; then the `process_review` success ("complete"), clean-failure ("failed" via missing profile / failed safety checks / review-not-found), and unexpected-exception paths; and finally re-run coverage to confirm the file clears 40% and record the "after" number.

**Blockers:**
N/A

---

### Check-in 2 (end of week)

**PR link:** [Pull Request for Issue 109](https://github.com/ascherj/pathreview/pull/471)

**Branch:** `test/109-review-service-test-coverage`

**What you built:**
Additive unit tests for `core/services/review_service.py`. The tests exercise the previously untested `process_review` orchestrator (success, clean-failure, and unexpected-exception paths) and its `_run_*` helpers, raising coverage of the file from ~22% to 93%.

**Tests added or updated:**
Only `tests/unit/test_review_service.py`. New tests cover: `_run_safety_checks` (valid output passes; no sections, incomplete section, out-of-range confidence, and a malformed non-dict section all fail); `_run_agent_orchestration` and `_run_rag_retrieval_generation` return the expected `sections`/`overall_score` keys; `_run_ingestion_pipeline` builds a source per profile field and commits, and returns `[]` (still committing) when no source fields are set; and `process_review` for the "complete" success path, the "failed" clean-failure paths (missing profile, failed safety checks, missing review → early return with no commit), and the unexpected-exception path (outer `except` sets status="failed", including when the recovery commit itself raises).

**Self-review confirmation:** [ ] make check passes  [ ] make test-unit passes

_Note: the new tests all pass and the added code is lint-clean. However, the full-suite `make test-unit` and `make check` still surface the pre-existing `AsyncMock`/`Mock` test failures and the associated lint warnings in this file, both documented as out of scope for Issue #109 in the Weeks 7–8 entries above._

**Draft PR feedback received from:** none

---

## Week 10 — Iteration & reflection

### Reviewer feedback

**Feedback received:** [ ] Yes  [x] No — still awaiting review

**Summary of feedback:**
no review in Summer 2026

**How you responded:**
N/A

---

### Reflection

**What was harder than you expected?**
[Be specific — what part of the process, codebase, or workflow
surprised you?]

**What did you learn about working in a large codebase?**
[What's different about contributing to someone else's production code
vs. building your own project?]

**How did AI tools help — and where did they fall short?**
[Where was AI assistance most useful this module? Where did you need
to go beyond what AI could give you?]

**What would you do differently if you started over?**
[Issue selection, planning, implementation, or process — anything
you'd change?]

**What are you most proud of from this module?**
[One thing — it doesn't have to be the PR itself.]