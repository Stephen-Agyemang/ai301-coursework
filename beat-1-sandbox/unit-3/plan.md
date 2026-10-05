# Plan: Fix README scorer test fixture word count (Issue #63)

## Diagnosis
In `tests/unit/test_readme_scorer.py`, `TestReadmeScorer::test_readme_with_all_quality_signals` fails under `--runxfail` with `AssertionError: assert 51 > 100`. The test makes two distinct assertions: `assert data['word_count'] > 100` and `assert data['word_count_category'] == "comprehensive"`. In `agent/tools/readme_scorer.py:68-73`, word count categories are assigned as `"minimal"` (< 100 words), `"adequate"` (100–499 words), and `"comprehensive"` (>= 500 words). The captured test execution logs `category=minimal word_count=51`. The scorer implementation in `agent/tools/readme_scorer.py` functions correctly; the inline markdown fixture in `tests/unit/test_readme_scorer.py` is too short (51 words) to satisfy either assertion.

## Scope
- In Scope:
  - Expand the markdown fixture string in `tests/unit/test_readme_scorer.py` under `test_readme_with_all_quality_signals` to exceed 520 words (targeting ~530 words) while preserving all existing quality signals (headings, installation, usage, code fences, badges, and demo links).
  - Remove the `@pytest.mark.xfail` decorator from `test_readme_with_all_quality_signals`.
- Out of Scope:
  - Modifying `agent/tools/readme_scorer.py`.
  - Modifying any other test cases in `tests/unit/test_readme_scorer.py`.

## Files Touched
- `tests/unit/test_readme_scorer.py`

## Approach
1. Open `tests/unit/test_readme_scorer.py` at `test_readme_with_all_quality_signals`.
2. Expand the inline `readme` fixture string by enriching project overview, architecture, configuration details, and troubleshooting notes to reach ~530 words, safely exceeding the 500-word threshold for `"comprehensive"`.
3. Verify that all quality signals are preserved so that `data['has_quickstart']`, `data['has_installation']`, `data['word_count'] > 100`, `data['word_count_category'] == "comprehensive"`, and `data['overall_score'] > 0.7` all pass.
4. Remove the `@pytest.mark.xfail` decorator from `test_readme_with_all_quality_signals`.
5. Run `pytest tests/unit/test_readme_scorer.py` to confirm all 23 tests pass.

## Test Plan
- Pre-fix baseline:
  - `pytest tests/unit/test_readme_scorer.py -k test_readme_with_all_quality_signals --runxfail` -> fails with `assert 51 > 100` and logs `category=minimal word_count=51`.
- Post-fix verification:
  - `pytest tests/unit/test_readme_scorer.py -k test_readme_with_all_quality_signals` -> passes with exit code 0.
  - Full module verification: `pytest tests/unit/test_readme_scorer.py` -> 23 passed, 0 failed, 0 xfailed.

## Risks and Unknowns
- Low risk. Changes are isolated to test fixture data. We will verify that adding text does not degrade the code-to-text balance or heading structures needed to keep `overall_score > 0.7` passing.

## Deviations
No deviations from the final plan. The fixture was expanded past 500 words to satisfy the comprehensive threshold, and the xfail decorator was removed.
