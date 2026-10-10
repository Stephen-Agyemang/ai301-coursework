# Rubric: is this pull request ready to submit?

## Checks

| Check | Evidence | Pass condition | Weight |
|---|---|---|---|
| plan-fidelity | Diff and changed files read against the plan's scope, implementation approach, and documented deviations | The diff implements the agreed plan without unapproved additions or silent omissions; any divergence or deferred work is explicitly documented and justified in the plan or description | required |
| test-verified | Test logs, commands, transcripts, or added tests in test evidence read against the plan's test plan and reproduction steps | Observable test evidence demonstrates the target behavior works as intended (via before/after output, reproduction transcripts, or passing tests covering the scoped path); the change is not left unverified or backed only by unsubstantiated assertions | required |
| diff-cleanliness | Full diff hunks inspected across all modified files | The diff is tightly scoped to the fix and free of debugging leftovers (such as print statements or debug logs), dead code experiments, and unrelated formatting or refactoring churn | required |
| standards-compliance | PR description and metadata read against repo-facts (template asks, issue linking, and stated AI policy) | The PR satisfies the target repository's stated requirements: required template sections or checklists are not visibly ignored, and an AI-use disclosure is present if the repo's stated policy requires one | required |

## Verdict rule

Accept if and only if every required check passes. A failure on any required check results in a reject verdict. Preferred checks provide diagnostic guidance and do not alter the verdict. Any check graded as unclear is treated as fail.

