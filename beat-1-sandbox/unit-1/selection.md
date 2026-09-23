# Unit 1 — Issue Selection

Path: `beat-1-sandbox/unit-1/selection.md`

Record of the issue carried into Unit 2, and of the evaluation runs that produced
`eval-run.txt`. This file is graded at the path above; a copy kept anywhere else in
the repository is not read.

Complete every labelled field below. Each is graded on its own; content placed under the
wrong label is not graded.

---

## Selected issue

**Issue link**

https://github.com/codepath/pathreview-ai301-fa26-s1/issues/63

**Verdict output**
```
Accepted, in rank order
#63 — README scorer test fixture too short for its own word-count assertion — accept
Unclaimed: pass — assignees [], no open linked PRs (repo's only open PR, #74, targets #60); the one comment is AliceKindle2 (authorAssociation: NONE, a classmate) saying "I'd like to investigate this one" — per the house rule, not a blocker.
maintainer-active: pass — Aburke225 pushed to main 2026-09-16, 6 days ago.
newcomer-scope: pass — labels good first issue, tier-1; one fixture in one test file.
repo-in-use: pass — 18 forks, plus external issue/PR activity from three non-member accounts.
contribution-policy: pass — docs/CONTRIBUTING.md says "We welcome contributions!"; no AI/LLM restriction anywhere in it, the PR template, or the repo root.
settled-spec: pass — no debate, no abandoned PRs.
actionable-description (preferred): pass — exact repro and failure: pytest tests/unit/test_readme_scorer.py -q → assert 51 > 100.
Why first: the only candidate whose fix lands entirely in tests/, with the failing assertion quoted verbatim. Smallest blast radius of the three.
[
{
"item": "https://github.com/codepath/pathreview-ai301-fa26-s1/issues/63",
"checks": [
{"name": "Unclaimed", "grade": "pass", "evidence": "assignees: []; no linked PRs; sole comment is classmate AliceKindle2 (author_association NONE), which the Path Review house rule says does not block"},
{"name": "maintainer-active", "grade": "pass", "evidence": "non-bot commit by Aburke225 to main on 2026-09-16, 6 days before today"},
{"name": "newcomer-scope", "grade": "pass", "evidence": "labels 'good first issue' and 'tier-1'; single test fixture in tests/unit/test_readme_scorer.py"},
{"name": "repo-in-use", "grade": "pass", "evidence": "18 forks (>=2) plus external activity from AliceKindle2, vchlinh, and nianiiier"},
{"name": "contribution-policy", "grade": "pass", "evidence": "docs/CONTRIBUTING.md: 'We welcome contributions!'; no AI/LLM ban in it, the PR template, or the repo root"},
{"name": "settled-spec", "grade": "pass", "evidence": "no design debate in the single-comment thread and no abandoned or rejected PRs"},
{"name": "actionable-description", "grade": "pass", "evidence": "body gives repro 'pytest tests/unit/test_readme_scorer.py -q' and observed failure 'assert 51 > 100'"}
], 
"verdict": "accept"
}
]
```
--- 
## Eval iterations

**Run history**

13/20, 16/20, 17/20, 18/20

**Issue analysis**

Issue: `issue-12`
- Rubric decision: `reject`
- Gold label: `reject`
- Reasoning: `issue-12` belongs to a repository with an explicit contributing policy ban restricting pull requests or forbidding AI/LLM tooling in contributor guidelines. Our rubric's `contribution-policy` check correctly evaluated `CONTRIBUTING.md` for explicit negative bans against outside contributors and AI assistance rather than penalizing open-door projects, successfully rejecting the issue and satisfying the mandatory policy category floor.

**Check rationale**

Check: `newcomer-scope`
> Quoted check: `The task is a self-contained, bounded change (such as a documentation update, a localized bug fix, or an issue with identified causes or clear repro steps) rather than an umbrella issue, an ongoing tracking list, or an open-ended feature wish with an unmade product decision`
- Reasoning: The earlier draft failed any task that was not strictly a single atomic change, which produced false rejections on multi-step bounded bugs like `issue-04`. We revised it to focus on whether the scope is self-contained and bounded, explicitly distinguishing valid single units of work from real anti-patterns (umbrella tracking lists, repository-wide overhauls, and unmade product decisions).

**Trade-offs**

By tuning `newcomer-scope` to filter out open-ended items and umbrella issues, we accept that we still conservatively reject `issue-01` (a documentation update) and `issue-19` (a maintainer-diagnosed performance bug) because LLM evaluators often treat performance optimization as beyond newcomer boundaries. However, this trade-off protected all 4 true reject items in the scope category (`issue-05`, `issue-10`, `issue-20`) with 4/4 agreement, keeping our overall suite at 18/20 and satisfying the passing bar.

---

## Selection rationale

**Selection rationale**

1. Fit to interests and time: I was looking for a clean, testable Python task where reproduction is clear and verifiable with standard tooling (`pytest`). Issue #63 fits my background in backend testing and Python diagnostics, and its contained blast radius inside `tests/unit/test_readme_scorer.py` makes it practical to reproduce, fix, and validate within the time available for Unit 2.
2. What the verdict identified correctly vs. human judgment: The verdict correctly identified that the repository is active, the issue is bounded to a single fixture, and reproduction steps are immediately actionable (`pytest tests/unit/test_readme_scorer.py -q → assert 51 > 100`). What the rubric could not weigh was the cognitive load between editing a test assertion versus deciding on an architectural fallback in production code (like in #56); I chose #63 because isolating the fix to the test suite provides an explicit, deterministic definition of done.
3. Anticipated difficulty in claiming it: Low. Although a classmate left an initial comment expressing interest ("I'd like to investigate this one"), no pull request has been opened or linked, no assignees exist on GitHub, and the Path Review house rules explicitly permit multiple students to work on shared issues without blocking course credit.
