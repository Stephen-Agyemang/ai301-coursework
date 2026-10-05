# Unit 3 — Plan and Build

Path: `beat-1-sandbox/unit-3/plan-and-implement.md`

Record of your plan, the branch you built it on, and the evaluation runs that produced
`eval-run.txt`. This file is graded at the path above; a copy kept anywhere else in the
repository is not read.

Complete every labelled field below. Each is graded on its own; content placed under the wrong
label is not graded.

---

## Posted upstream

**GitHub username**

Stephen-Agyemang

**Plan comment**

[Paste the direct permalink to your plan comment on https://github.com/codepath/pathreview-ai301-fa26-s1/issues/63 here]

### Proposed Plan for Issue #63

#### Context and Reproduction
Reproduction verified on macOS 27.0 (Darwin arm64), Python 3.14.7, pytest 9.1.1 at commit `f89c06f` ([reproduction comment](https://github.com/codepath/pathreview-ai301-fa26-s1/issues/63#issuecomment-5852951394)). Running `pytest tests/unit/test_readme_scorer.py -k test_readme_with_all_quality_signals --runxfail` yields `assert 51 > 100` and logs `category=minimal word_count=51`.

#### Root Cause
As noted by AliceKindle2 on the thread, `test_readme_with_all_quality_signals` asserts both `data['word_count'] > 100` and `data['word_count_category'] == "comprehensive"`. Under `agent/tools/readme_scorer.py:68-73`, the `"comprehensive"` classification requires >= 500 words. The current 51-word fixture triggers `category=minimal`. Modifying the fixture to under 500 words leaves the category assertion failing. The scorer logic is correct; only the test fixture content is insufficient.

#### Proposed Approach
1. Create branch `fix/63-readme-fixture-word-count`.
2. Expand the inline `readme` fixture in `tests/unit/test_readme_scorer.py` (`test_readme_with_all_quality_signals`) to ~530 words, adding realistic project overview, configuration, and troubleshooting documentation while keeping all headers, code fences, badges, and links intact.
3. Remove the `@pytest.mark.xfail` decorator from `test_readme_with_all_quality_signals`.
4. Verify that `pytest tests/unit/test_readme_scorer.py` passes all 23 tests without `xfail`.

#### Scope
- Files touched: `tests/unit/test_readme_scorer.py`
- Out of scope: `agent/tools/readme_scorer.py` (scorer implementation unchanged).

---

## Your branch

**Branch**

fix/63-readme-fixture-word-count

**Evidence**

Before the fix:
```bash
pytest tests/unit/test_readme_scorer.py -k test_readme_with_all_quality_signals --runxfail