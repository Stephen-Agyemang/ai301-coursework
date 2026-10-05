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

<https://github.com/codepath/pathreview-ai301-fa26-s1/issues/63#issuecomment-5986851270>

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

`fix/63-readme-fixture-word-count`

**Evidence**

Before the fix:

```text
pytest tests/unit/test_readme_scorer.py -k test_readme_with_all_quality_signals --runxfail

tests/unit/test_readme_scorer.py:60: AssertionError
>       assert data['word_count'] > 100
E       assert 51 > 100

tests/unit/test_readme_scorer.py:60: AssertionError
----------------------------------------------------------------------------------------------------------- Captured stdout call -----------------------------------------------------------------------------------------------------------
2026-10-04 21:26:36 [info     ] readme_scored                 category=minimal score=0.8717142857142858 word_count=51
========================================================================================================= short test summary info ==========================================================================================================
FAILED tests/unit/test_readme_scorer.py::TestReadmeScorer::test_readme_with_all_quality_signals - assert 51 > 100
===================================================================================================== 1 failed, 22 deselected in 0.40s =====================================================================================================
```

After the fix:

```text
pytest tests/unit/test_readme_scorer.py -v

tests/unit/test_readme_scorer.py::TestReadmeScorer::test_readme_with_all_quality_signals PASSED                                                 [  4%]
tests/unit/test_readme_scorer.py::TestReadmeScorer::test_readme_with_no_content PASSED                                                          [  8%]
tests/unit/test_readme_scorer.py::TestReadmeScorer::test_readme_with_only_title PASSED                                                          [ 13%]
tests/unit/test_readme_scorer.py::TestReadmeScorer::test_word_count_category_minimal PASSED                                                     [ 17%]
tests/unit/test_readme_scorer.py::TestReadmeScorer::test_word_count_category_adequate PASSED                                                    [ 21%]
tests/unit/test_readme_scorer.py::TestReadmeScorer::test_word_count_category_comprehensive PASSED                                               [ 26%]
tests/unit/test_readme_scorer.py::TestReadmeScorer::test_installation_section_detection PASSED                                                  [ 30%]
tests/unit/test_readme_scorer.py::TestReadmeScorer::test_usage_section_detection PASSED                                                         [ 34%]
tests/unit/test_readme_scorer.py::TestReadmeScorer::test_setup_keyword_counts_as_installation PASSED                                            [ 39%]
tests/unit/test_readme_scorer.py::TestReadmeScorer::test_quickstart_counts_as_usage PASSED                                                      [ 43%]
tests/unit/test_readme_scorer.py::TestReadmeScorer::test_badge_detection PASSED                                                                 [ 47%]
tests/unit/test_readme_scorer.py::TestReadmeScorer::test_demo_link_detection PASSED                                                             [ 52%]
tests/unit/test_readme_scorer.py::TestReadmeScorer::test_tech_stack_section_detection PASSED                                                    [ 56%]
tests/unit/test_readme_scorer.py::TestReadmeScorer::test_technologies_keyword_counts PASSED                                                     [ 60%]
tests/unit/test_readme_scorer.py::TestReadmeScorer::test_overall_score_calculation PASSED                                                       [ 65%]
tests/unit/test_readme_scorer.py::TestReadmeScorer::test_missing_readme_content_key PASSED                                                      [ 69%]
tests/unit/test_readme_scorer.py::TestReadmeScorer::test_result_has_all_required_fields PASSED                                                  [ 73%]
tests/unit/test_readme_scorer.py::TestReadmeScorer::test_case_insensitive_section_detection PASSED                                              [ 78%]
tests/unit/test_readme_scorer.py::TestReadmeScorer::test_whitespace_only_readme PASSED                                                          [ 82%]
tests/unit/test_readme_scorer.py::TestReadmeScorer::test_example_keyword_counts_as_usage PASSED                                                 [ 86%]
tests/unit/test_readme_scorer.py::TestReadmeScorer::test_score_scales_with_word_count PASSED                                                    [ 91%]
tests/unit/test_readme_scorer.py::TestReadmeScorer::test_try_it_as_demo_indicator PASSED                                                        [ 95%]
tests/unit/test_readme_scorer.py::TestReadmeScorer::test_built_with_counts_as_tech_stack PASSED                                                 [100%]

================================================================= 23 passed in 0.12s ==================================================================

```

## Eval iterations

Answer all four sections. Quote source text directly; paraphrase does not satisfy these
fields.

**Run history**

```text
agreement: 18/20 scored items  (bar: 18/20: PASS)
```

**Package analysis**

```text
pkg-01:
Gold label: reject
Rubric decision: reject
Category: wrong-cause
Explanation: The package proposed a diagnosis and approach that misidentified the root cause of the test failure, targeting peripheral module logic rather than the actual failing mechanism. The rubric's diagnosis-grounded check evaluated the claims against the issue evidence and flagged the discrepancy, grading the check as fail and issuing an overall reject verdict matching the gold label.
```

**Check rationale**

> "diagnosis-grounded: Does the diagnosis identify the root cause supported by the reproduction evidence, without misattributing the failure or ignoring key failure signals?"

This check reads this way because early test iterations revealed that plans addressing only the immediate line of an AssertionError often failed to account for tightly coupled secondary assertions (such as word_count_category == "comprehensive" requiring >= 500 words). The check was refined to require that the diagnosis account for all observed failure signals and thresholds rather than settling for a partial surface-level explanation.

**Trade-offs**

By requiring concrete implementation steps in actionably-buildable to prevent underspecified plans from passing, the check trades off agreement on borderline packages such as pkg-13 and pkg-14 (gold accept, but rejected by the rubric for omitting explicit architecture boundaries). In exchange, it guarantees zero false accepts across scope-creep (4/4), thread-convention (2/2), unbuildable (3/3), and wrong-cause (4/4).

Related paths: `plan.md` and `eval-run.txt` in this directory; your skill's files in
`tools/plan-check/`.
