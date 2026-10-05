# Rubric: is this plan ready to post and build from?

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