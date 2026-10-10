---
name: pr-precheck
description: Grade a PR package (a candidate pull request read against the plan it claims to implement and the issue that plan belongs to) and decide whether it is ready to submit. Use when checking your own branch, draft PR title, and description before opening the pull request, or when grading an eval package bundle.
---

# pr-precheck: rubric-driven PR grading

## The question

Answer exactly one question about exactly one PR package: **is this ready to submit?**

A PR package is a candidate pull request (title, description, branch commits, diff, and test evidence) read against the plan it claims to implement and the upstream issue that plan belongs to. Never answer any other question, never grade more than one package per invocation, and never decide based on intuition or polish. Always execute the rubric and procedure specified in this directory to produce an objective, evidence-based determination.

## Inputs and modes

Operate strictly in one of two modes:

1. **Live mode**:
   - Check a candidate PR before opening it upstream.
   - Read the following inputs from the local working repository and upstream repo:
     - The approved plan and any documented deviations in `plan.md` (or house plan for a house-chain student).
     - The full code diff produced by running `git diff main...HEAD` (three dots) from the root of the branch working copy.
     - The draft PR title and description in `pr_draft.md`.
     - The test run outputs and verification logs in `test_evidence.md`.
     - The issue thread, PR template (`.github/PULL_REQUEST_TEMPLATE.md`), and contributing guidelines (`docs/CONTRIBUTING.md`) fetched from the repository.
   - Refuse to grade if any required input artifact is missing.

2. **Eval mode**:
   - Grade a standalone evaluation package bundle (e.g., from `eval/packages/`).
   - The bundle text is the entire world. Do not access external networks, do not fetch remote repos or issues, and do not inspect the local working directory.
   - Evaluate all checks against the bundle contents using the complete verdict rule.

## The scope seam (live mode only)

In live mode, read `scope.md` before executing any other step:
- Verify that the target repository matches the `Repo:` configuration.
- If the `Repo:` line contains a bracketed placeholder (e.g., `[your-repo]`) or is blank, stop immediately without grading and instruct the student to populate `Repo:` with their assigned Path Review repo.
- If the PR targets an upstream repository outside the scoped repository, stop immediately and refuse to grade.
- In eval mode, ignore `scope.md` completely.

## The voice seam (live mode only)

In live mode, read `voice-guide.md` after gathering evidence:
- Evaluate the outgoing PR title and description in `pr_draft.md` against the communication standards defined in `voice-guide.md`.
- Report any stylistic or voice violations in the pre-verdict summary output.
- Do not alter or fail the final verdict based solely on `voice-guide.md` unless an explicit check in `rubric.md` mandates it.
- In eval mode, ignore `voice-guide.md` completely.

## Component reads

Execute the evaluation workflow strictly via the constituent components:
1. Read `rubric.md` to load the evaluation checks, evidence expectations, pass conditions, weights, and verdict rules.
2. Read `references/evidence-guide.md` to map each check to its corresponding evidence locations across the plan, diff, draft description, and test logs.
3. Read and execute `procedure.md` in sequential order.

**Refusal Rule**: If either `rubric.md` or `procedure.md` lacks substantive operational instructions (e.g., is empty or contains only unpopulated template comments), stop execution immediately, refuse to grade, and state that the required rubric or procedure component is missing. If `procedure.md` is silent on a necessary operational step, halt and report the procedural gap; never improvise or invent steps outside the procedure.

## Verdict and output

Produce a binary verdict:
- `accept`: The PR package is complete, faithful to the plan, verified by test evidence, and ready to submit upstream.
- `reject`: The PR package has unaddressed failures, unexplained deviations, test gaps, debris, or standards violations and must be held.

Precede the machine verdict with a human-readable summary listing each check, its grade, and its deciding evidence. End the output with the fixed JSON block below, valid, completely populated, and with no trailing text:

```json
{
  "item": "<PR URL bundle id or>",
  "checks": [
    {"name": "<check name>", "grade": "pass|fail|unclear",
     "evidence": "<one line: the fact or quote that decided it>"}
  ],
  "verdict": "accept|reject"
}