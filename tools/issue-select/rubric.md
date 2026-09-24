# Rubric: is this a good first issue?

<!--
THIS IS THE PART YOU WRITE. The skill in SKILL.md executes whatever checks
you define here. It ships empty on purpose: the judgment is your work.

A filled rubric must contain:

1. At least one row in the checks table. Each row needs all four columns:
   - Check: a short name (used in the output JSON).
   - Evidence: exactly what to look at, and where. Name the source
     (repo-facts block, issue body, comment thread, or the locations in
     references/evidence-guide.md). "The repo" is not a source; "the last
     5 default-branch commit dates" is.
   - Pass condition: a condition someone else could apply and get your
     answer. Prefer thresholds with numbers ("a maintainer commented
     within 30 days") over adjectives ("maintainer is responsive").
   - Weight: `required` (a fail here rejects the issue) or `preferred`
     (never changes the verdict; a nice-to-have that helps rank the
     issues you accept).

2. A verdict rule below the table: how the check grades combine into
   accept or reject, including how `unclear` is treated. The verdict
   space is binary. If you write no rule for `unclear`, the skill treats
   it as fail.

Cover what actually kills first contributions. The lecture named four
families: the maintainer is alive, the repo is in use, the scope fits a
newcomer, and nobody else is already on it. A rubric that ignores a family
will fail eval issues designed around that family.
-->

## Checks

| Check | Evidence | Pass condition | Weight |
| --- | --- | --- | --- |
| Unclaimed | Issue assignee field, linked pull requests, and the comment thread | Assignee list is empty, there are no open linked PRs referencing the issue, and no comment within the last 14 days claiming to work on it | required |
| maintainer-active | Default branch commit history and recent issue/PR comments | At least 1 non-bot commit or maintainer comment within the last 90–180 days | required |
| newcomer-scope | Issue labels and issue body description | The task is not an umbrella tracking issue, an open-ended design discussion or feature wish, or a repository-wide refactor | required |
| actionable-description | Issue body text | Body includes explicit steps to reproduce, code snippets/logs, or a clear statement of expected vs. actual behavior | preferred |
| repo-in-use | repo-facts (stars, forks) or README links | The repo has tagged releases, evidence of external user issues/discussions, OR at least 5 stars / 2 forks | required |
| contribution-policy | CONTRIBUTING.md, issue guidelines, or README | The repository does not explicitly forbid outside contributors, restrict issues to specific internal groups, or explicitly ban the use of AI/LLM tools in its contributing guidelines | required |
| settled-spec | Issue comment thread and linked pull requests | The issue discussion has no ongoing controversies, unresolved design debates, or competing proposals regarding the intended implementation, and does not have multiple abandoned or rejected pull requests | required |



## Verdict rule

<!-- State how the grades above combine into accept or reject, and how
unclear is treated. Example shape (write your own): "accept if every
required check passes; preferred checks never change the verdict, they
rank accepted issues; unclear counts as fail." -->

Accept: An issue is marked accept if and only if all required checks pass.
Reject: An issue is marked reject if any required check fails or is evaluated as unclear.
Ranking: Passing preferred checks help rank accepted issues higher.
