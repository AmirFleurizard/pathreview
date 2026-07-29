## Week 7 — Issue selection

**Issue link:** (https://github.com/ascherj/pathreview/issues/112)

**Issue title:** Add a performance benchmark for the ingestion pipeline to detect regressions

**Tier:** [ ] Tier 1  [ ] Tier 2  [X] Tier 3

**Problem summary:**
The issue I'm working on is about making the ingestion pipeline fast enough for large portfolio processing without introducing hidden slowdowns. Currently, the codebase lacks an automated performance check to catch regressions at ingestion time, so a slowdown could go unnoticed until it affects users. A successful fix should add a benchmark test that measures a representative ingestions workload and fail if the average runtime exceeds the 30 second threshold. The fix would primary affect the ingestion pipeline and the ingestion and test folders. 

**Branch name:** feature/112-ingestion-performance-benchmark

**Setup confirmation:** [X] App runs locally at localhost:5173

**Cohort ledger:** [X] Issue added to cohort ledger

**Is This Issue Right for Me?**
Part 1 — Understanding the Issue
Can I explain what this issue is asking for in my own words? 
[X] I can explain the problem and the expected behavior in 2–3 sentences without reading the issue.
I've located the relevant files and confirmed they exist in the codebase.
[X] I can describe a concrete before-and-after: what the user sees before the fix and what they see after.
[X] Is the tier a realistic match for where I am right now?

If this is my first open source contribution: I'm choosing Tier 1.
[X] If I've contributed to large codebases before: Tier 2 or 3 is fair game.
[X] I'm not choosing a Tier 3 issue to "challenge myself" if I haven't completed a Tier 1 or 2 first — scope surprises in Week 9 don't have a safety net.

[X] I've found and read the specific code the issue references (not just the file — the function or section).
[X] I've read enough surrounding context that I can write a rough plan for the fix without looking anything up.
[X] I've found the test file for my module and read at least one test end-to-end.

[X] I've checked the issue comments and the ledger's Claims count, and I'm fine with how many others are on this issue.
[X] I've estimated the time this will take and I'm confident I can complete it before the Week 9 deadline.
[X] This issue has no open blockers or dependencies on other unresolved issues.


## Week 8 — Reproduction & solution planning

**Reproduction commit link:** 

**Reproduction summary:**
This is a feature gap, not a bug, so "reproduction" meant confirming the gap rather than triggering an error: `tests/benchmarks/` contains only an empty `__init__.py`, `pytest-benchmark` is listed as a dev dependency in `pyproject.toml` but is never imported anywhere in the codebase, and there is no existing code path (in `ingestion/pipeline.py` or elsewhere) that ingests a full portfolio (resume + multiple repos) in one call — confirming the benchmark test named in the issue genuinely does not exist yet.

**PLAN.md link:** `https://github.com/AmirFleurizard/pathreview/blob/feature/112-ingestion-performance-benchmark/PLAN.md`.

**Walkthrough video (recommended):** [link to your Loom video, ≤2 min — recommended, not graded]

**Blockers or open questions:**
- `IngestionPipeline._check_skip` (`ingestion/pipeline.py:280`) queries `db_session` in a way that, if mocked naively (bare `Mock()`), makes every ingestion silently skip — need to make sure the benchmark's mock db session returns `None` from `.first()` so it measures real work, and add a correctness assertion (not just timing) to catch this failing silently in the future.
- Unsure whether this benchmark should be wired into `ci.yml` (it currently only runs `test-unit`/`test-integration`) or stay a local/manual `make test-benchmark` check — benchmark timing tends to be noisy on shared CI runners. Leaning toward local-only for now but want to confirm with @jamjamgobambam before Week 9.
- Want to verify the 30s threshold can actually catch a regression (not just always pass trivially) by temporarily introducing a slowdown locally and confirming the test fails, before considering this done.