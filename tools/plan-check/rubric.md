# Rubric: is this plan ready to post and build from?

<!--
THIS IS THE PART YOU WRITE. The skill in SKILL.md executes whatever
checks you define here (via your procedure.md). It ships empty on
purpose: the judgment is your work.

A filled rubric must contain:

1. At least one row in the checks table. Each row needs all four
   columns:
   - Check: a short name (used in the output JSON).
   - Evidence: exactly what to look at, and where in the package. Name
     the part (the plan's scope statement, the test plan read against
     the repro evidence's steps, the plan comment read against the
     thread highlights, the repo-facts block) or a location from your
     references/evidence-guide.md. "The plan" is not a source; "the
     plan's stated cause read against what the repro evidence shows"
     is.
   - Pass condition: a decision rule about the OUTCOME that someone
     else could apply and get your answer. Judge the thing itself (is
     this one bounded change? could a stranger start executing it?),
     never the write-up's shape (how many sections it has, how long it
     is, whether it uses headings). Structure-shaped checks are what
     make graders disagree with themselves.
   - Weight: `required` (a fail here holds the package) or `preferred`
     (never changes the verdict).

2. A verdict rule below the table: how the check grades combine into
   accept (ready) or reject (hold), including how `unclear` is
   treated. The verdict space is binary. If you write no rule for
   `unclear`, the skill treats it as fail.

Cover what actually gets bad plans posted. The lecture named the
failure families: the diagnosis ignores or contradicts the reproduced
evidence, the change is unbounded (scope creep), the plan targets the
symptom while the evidence points at the cause, a stranger could not
start executing it, the test plan proves nothing observable, the
unknowns are dressed up as certainty, and the comment ignores what the
thread or the repo's stated conventions ask. A rubric that ignores a
family will fail eval packages designed around that family.
-->

## Checks

| Check | Evidence | Pass condition | Weight |
|---|---|---|---|
| diagnosis-grounded | Plan's stated cause read against repro evidence and control runs | The diagnosis identifies a root cause consistent with all reproduction evidence and controls; it does not blame a component or mechanism that control steps or debug output already ruled out | required |
| scope-bounded | Plan's scope and files-touched read against the issue | The proposed change is strictly bounded to resolving the reported issue; it does not introduce scope creep such as refactors, architecture redesigns, framework migrations, or unsolicited feature additions | required |
| actionably-buildable | Plan's approach and concrete files touched | The plan specifies concrete files and a clear implementation approach that a stranger could immediately execute without deferring core architecture decisions or guessing layers | required |
| test-plan-decisive | Plan's test plan read against repro steps | The test plan re-runs reproduction steps or adds regression assertions that produce a concrete, observable before-and-after outcome; vague statements like "should feel fast" or merely running the full test suite without a specific observable check fail | required |
| thread-and-conventions | Candidate plan comment read against thread highlights and repo conventions/CONTRIBUTING | The comment follows maintainer guidance in the thread, engages with existing PRs/prior art, and strictly adheres to repository conventions (including mandatory AI disclosure policies where required) | required |

## Verdict rule

Accept if every required check passes. Reject if any required check fails or is evaluated as unclear. Preferred checks provide diagnostic guidance but never change the verdict.