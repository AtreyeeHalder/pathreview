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

**Reproduction commit link:** [link to commit documenting the reproduced issue]

**Reproduction summary:**
[1–2 sentences: How did you reproduce the issue? What did you observe?]

**PLAN.md link:** [link to PLAN.md in your fork]

**Walkthrough video (recommended):** [link to your Loom video, ≤2 min — shared for early feedback]

**Blockers or open questions:**
[Anything you're still uncertain about going into Week 9, or leave blank]