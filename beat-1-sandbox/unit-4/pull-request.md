# Unit 4 — Test and Submit

Path: `beat-1-sandbox/unit-4/pull-request.md`

Record of the pull request you opened against the Path Review repo, and of the evaluation
runs that produced `eval-run.txt`. This file is graded at the path above; a copy kept
anywhere else in the repository is not read.

Complete every labelled field below. Each is graded on its own; content placed under the wrong
label is not graded.

---

## Your pull request

**Pull request**

https://github.com/codepath/pathreview-ai301-fa26-s1/pull/116

**Branch**

fix/63-readme-fixture-word-count

## Eval iterations

Answer all four sections. Quote source text directly; paraphrase does not satisfy these
fields.

**Run history**

20/20 scored items

**Package analysis**

pkg-07
- Rubric decision: reject
- Gold label: reject
- Explanation: The PR in pkg-07 falls into the not-tested category. While changes were made to implementation code, the submission lacked verified test evidence or execution logs demonstrating that the targeted behavior actually passed or reproduced. Our test-verified check requires observable test evidence (such as before/after command outputs, test transcripts, or passing test logs) rather than unsubstantiated claims, correctly triggering a fail on test-verified and yielding a reject verdict.

**Check rationale**

"Diff and changed files read against the plan's scope, implementation approach, and documented deviations | The diff implements the agreed plan without unapproved additions or silent omissions; any divergence or deferred work is explicitly documented and justified in the plan or description | required"

This check was designed to catch silent drift without penalizing legitimate, documented adjustments. Earlier formulations strictly failed any diff that differed from the original planned files or lines. We revised it to explicitly recognize documented deviations, allowing PRs that encounter edge cases to pass as long as divergences are justified in the description or plan, while still failing PRs that silently add unreviewed code or omit planned deliverables.

**Trade-offs**

Nothing changed, and here is how I know:
Our final full run evaluated all 20 scored packages simultaneously with python3 run_eval.py --save-run eval-run.txt, achieving 20/20 agreement (clear-accept 7/7, not-tested 4/4, silent-drift 4/4, standards-wall 2/2, unreviewable 3/3). Because our first complete calibrated run matched every gold label across all categories, no subsequent check loosenings were performed that could introduce regressions or flip existing packages.

---

Related paths: `eval-run.txt` in this directory; your skill's files in
`tools/pr-precheck/`.
