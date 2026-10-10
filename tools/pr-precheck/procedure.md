# Procedure: how this tool grades a PR package

## Read order

Execute the reading passes in this strict sequential order before grading any check:

1. **Read the Plan (`plan.md` or package plan)**:
   - Extract the intended issue diagnosis and root cause.
   - Record the stated file scope (files to touch and files explicitly out of scope).
   - Record the planned implementation approach and the planned test steps.
   - Record any documented deviations under `## Deviations`.
2. **Read the Diff (`git diff main...HEAD` or package diff)**:
   - List every file path modified, added, or deleted.
   - Inspect all added and removed hunks to understand the actual code modifications.
3. **Read the Test Evidence (`test_evidence.md` or package test section)**:
   - Identify reproduction outputs (before vs. after the fix).
   - Identify the results of repository test/lint/typecheck suites.
4. **Read the PR Description (`pr_draft.md` or package description)**:
   - Extract the PR title, summary, linked issue (`Closes #...`), changes list, and testing claims.
   - Extract the mandatory AI-use disclosure statement.
5. **Read Upstream Requirements (Live mode: `.github/PULL_REQUEST_TEMPLATE.md` and `docs/CONTRIBUTING.md`)**:
   - Check repository template structure, branch naming conventions, and issue test suppression rules.

Reading the plan before the diff is essential: the executor must know the agreed boundaries and deviations before inspecting code changes, preventing the diff from retroactively redefining the plan.

## Evidence gathering

Gather concrete evidence pairings for each rubric check:

1. **Plan Fidelity Pairing**:
   - Compare the set of modified files in the diff directly against the files listed in the plan's scope.
   - Compare the implementation approach in the diff hunks against the plan's proposed steps.
   - Check whether any divergence is documented and justified in `plan.md` under `## Deviations`.
   - Record the exact file paths or diff hunks that match or violate the plan scope.
2. **Test Verification Pairing**:
   - Compare the execution logs in the test evidence against the plan's test plan.
   - Verify whether the issue's failing condition is shown failing before the change and passing after the change.
   - Verify that required repository test suites (`make test-unit`, `make test-integration`, `make lint`, `make typecheck`) were run, or that failures/skips are explicitly disclosed with technical rationale.
   - Record the passing/failing command strings and terminal summaries.
3. **Diff Cleanliness Pairing**:
   - Scan diff hunks for transient debugging artifacts (e.g., `print`, `console.log`, `debugger`, `breakpoint()`), commented-out code, leftover test scratchpads, or unintended whitespace churn.
   - Record line numbers and snippet content for any detected debris.
4. **Standards Compliance Pairing**:
   - Verify that the PR description populates all required template sections (Summary, Issue, Changes, Testing, Notes for Reviewers).
   - Verify that the issue is explicitly linked using the repo's expected format (e.g., `Closes #<id>`).
   - Verify that the Notes for Reviewers section includes an honest, plain-language AI-use disclosure.
   - In live mode, evaluate the title and description against `voice-guide.md` and record any stylistic notes.

## Check execution

Grade the checks in strict rubric order using the gathered evidence pairings:

1. **Grade `plan-fidelity`**:
   - **Pass**: Diff modifications match the plan scope and implementation steps, or any deviations are explicitly documented and justified.
   - **Fail**: The diff silently touches unapproved files, adds unapproved features (silent drift), or omits agreed changes without disclosure.
   - **Unclear**: The plan scope is ambiguous or the diff cannot be mapped to the plan.
2. **Grade `test-verified`**:
   - **Pass**: Observable test output confirms the bug is fixed and repo test checks were run (or honest, justified explanations exist for known limitations).
   - **Fail**: Test evidence is missing, fabricated, lacks observable before/after proof, or hides test suite failures.
   - **Unclear**: Claimed test results cannot be verified from the provided logs.
3. **Grade `diff-cleanliness`**:
   - **Pass**: The diff is clean, minimal, and contains only code necessary to execute the fix.
   - **Fail**: The diff contains debugging debris, commented-out dead code, temporary artifacts, or unrelated refactor churn.
   - **Unclear**: Diff formatting is corrupted or incomplete.
4. **Grade `standards-compliance`**:
   - **Pass**: All required template fields are completed, the issue is linked, and the AI disclosure is present.
   - **Fail**: Required template sections are empty, the issue link is omitted, or AI disclosure is missing.
   - **Unclear**: Required sections are ambiguous or cannot be verified.

If evidence for any check is missing, grade that check as `fail` (or `unclear`, which evaluates to `fail` under the verdict rule). Never invent or assume facts outside the package.

## Verdict assembly

Assemble the final outcome strictly per `rubric.md`:

1. **Tally Results**:
   - If all required checks (`plan-fidelity`, `test-verified`, `diff-cleanliness`, `standards-compliance`) received `pass`, the final verdict is `accept`.
   - If any required check received `fail` or `unclear`, the final verdict is `reject`.
2. **Format Deciding Evidence**:
   - For each check, produce a concise one-line factual summary citing the specific file, line, command, or quote that determined the grade.
   - In the human-readable summary, if the verdict is `reject`, prominently highlight the first failing required check in rubric order as the primary blocker.
3. **Emit Terminal JSON Block**:
   - Output the valid, closed JSON block matching the contract schema as the very last element of the response, with nothing after it.