# Rubric: is this reproduction package ready to post?

<!--
THIS IS THE PART YOU WRITE. The skill in SKILL.md executes whatever
checks you define here. It ships empty on purpose: the judgment is your
work.

A filled rubric must contain:

1. At least one row in the checks table. Each row needs all four
   columns:
   - Check: a short name (used in the output JSON).
   - Evidence: exactly what to look at, and where in the package. Name
     the part (the claim comment, the repro report's environment
     record, the artifacts read against the issue's description, the
     repo-facts block) or a location from your
     references/evidence-guide.md. "The report" is not a source; "the
     output excerpt read against the error the issue describes" is.
   - Pass condition: a decision rule about the OUTCOME that someone
     else could apply and get your answer. Judge the thing itself (does
     the artifact show the issue's behavior?), never the write-up's
     shape (how many steps it has, how long it is, whether it uses a
     template's headings). Structure-shaped checks are what make
     graders disagree with themselves.
   - Weight: `required` (a fail here holds the package) or `preferred`
     (never changes the verdict).

2. A verdict rule below the table: how the check grades combine into
   accept (ready) or reject (hold), including how `unclear` is
   treated. The verdict space is binary. If you write no rule for
   `unclear`, the skill treats it as fail.

Cover what actually gets bad packages posted. The lecture named the
proof families: the environment is recorded, the steps are complete
and followable, the behavior shown matches the issue (not an adjacent
one), the outcome is stated honestly (an evidenced cannot-reproduce is
a pass, a confident wrong-target is not), and the words respect the
repo's conventions. A rubric that ignores a family will fail eval
packages designed around that family.
-->

## Checks

| Check | Evidence | Pass condition | Weight |
| --- | --- | --- | --- |
| names-the-issue | Candidate claim comment and issue title/body | The claim comment explicitly references the specific issue number, title, or distinctive symptom being investigated rather than an ungrounded generic claim or mere '+1' | required |
| promise-not-assertion | Candidate claim comment | The claim states an intent to investigate or verify reproduction, without promising a guaranteed fix, asserting an ETA, or declaring premature resolution | required |
| environment-recorded | Candidate repro report | The report records the concrete reproduction environment (such as OS, runtime/tool versions, commit SHA, or specific test dependencies) | required |
| steps-rerunnable | Candidate repro report | The report provides explicit, runnable commands or unambiguous step-by-step instructions that allow a stranger to reproduce the test from scratch | required |
| observed-matches-issue | Candidate repro report read against the issue description | The observed outcome or terminal output directly addresses the issue's reported failure (or provides an evidenced cannot-reproduce report), avoiding speculative diagnosing or tangential behavior | required |
| repo-conventions | Candidate comments read against repo facts and CONTRIBUTING.md | The comments conform to repository-specific contribution rules, including any mandatory AI/LLM disclosure policies or required issue template formats | required |

## Verdict rule

Accept if every required check passes. Reject if any required check fails or is evaluated as unclear. Preferred checks provide diagnostic guidance but never change the verdict (none yet so everything is good).
