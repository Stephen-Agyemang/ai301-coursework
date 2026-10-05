# Procedure: how this skill grades a plan package

<!--
THIS IS THE PART YOU WRITE, and it is a new kind of part. Weeks 1 and
2, SKILL.md carried a numbered workflow and you only wrote judgment
files. This week the workflow is gone from the frame: SKILL.md says
"execute procedure.md", and these are the operating steps you author.
The machinery is in your hands now.

Your operator swap is the design brief. When your executor stalled
because your rubric said WHAT to decide but not HOW to find the
evidence, that was a procedure gap. This file is where those gaps get
closed: a complete procedure lets someone who has never seen a plan
package before (a groupmate, or the skill itself) grade one exactly the
way you would.

Under each stage heading below, write the concrete steps for that
stage. The one-line note under each heading says what a complete
procedure must decide there. Write steps, not intentions: "read the
repro evidence before the plan, and note what behavior it pins down"
is a step; "understand the context" is a wish.
-->

## Read order

1. **Read Repo Conventions & Thread Highlights first**: Note any explicit maintainer instructions, open PRs, or repository policies (especially AI disclosure rules). This establishes the constraints of the room.
2. **Read Reproduction Evidence & Controls second**: Note the exact failure mechanism, commands, and especially any control runs, timing outputs, or debug logs that rule out alternative explanations.
3. **Read Candidate Plan third**: Read the Diagnosis, Scope, Files Touched, Approach, and Test Plan against the evidence gathered in steps 1 and 2.
4. **Read Candidate Comment last**: Verify whether the comment faithfully reflects the plan, follows the thread context, and includes required notices.

## Evidence gathering

1. **For diagnosis-grounded**: Extract the plan's claimed root cause. Cross-check against every control step and log in the reproduction evidence. Flag any contradiction.
2. **For scope-bounded**: Extract the listed files and planned changes. Check if any change addresses matters outside the specific issue (e.g. refactoring, library migrations, new features).
3. **For actionably-buildable**: Check whether concrete file paths and code mechanisms are named. Flag if key technical choices are left as open questions for build time.
4. **For test-plan-decisive**: Check the test plan for a concrete observable outcome (specific return code, output message, or distinct test assertion). Flag if it merely says "run test suite" or uses subjective phrasing ("feels faster").
5. **For thread-and-conventions**: Read the comment text against the thread highlights and repository rules. Check whether maintainer guidance is acknowledged and whether mandatory AI disclosures are present.

## Check execution

1. Grade `thread-and-conventions`: If mandatory repo disclosures are omitted or maintainer directions are ignored, mark fail.
2. Grade `diagnosis-grounded`: If the diagnosis contradicts repro controls or debug logs, mark fail.
3. Grade `scope-bounded`: If the plan bundles unrelated cleanups, refactors, or migrations, mark fail.
4. Grade `actionably-buildable`: If no concrete files or approaches are specified, mark fail.
5. Grade `test-plan-decisive`: If no decisive, observable test outcome is stated, mark fail.
6. If any required check is uncertain or lacks necessary evidence, grade it as `unclear`.

## Verdict assembly

1. Check all required check grades.
2. If every required check is `pass`, assign final verdict `accept`.
3. If one or more required checks are `fail` or `unclear`, assign final verdict `reject`.
4. Format the final output as a JSON block with each check's grade, citing direct quotes from the package for the deciding check.
