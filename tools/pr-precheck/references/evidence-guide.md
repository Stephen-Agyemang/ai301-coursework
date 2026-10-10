# Evidence guide: where evidence lives in a PR package

## Plan fidelity (harness category: silent-drift)

### Where it lives
- **Eval package**: The `plan-context` block (specifically the stated files in scope, out of scope, implementation approach, and any deviation notes), compared against the candidate PR's unified diff and the claims in the candidate PR description.
- **Live mode**: `plan.md` in your repo root (including `## Scope`, `## Proposed Approach`, and any notes under `## Deviations`), compared against `git diff main...HEAD` and the changes listed in `pr_draft.md`.

### What good looks like
- Every file modified in the diff is explicitly listed within the plan's stated file scope or accounted for in an explicit deviation note.
- The diff implements the core logic described in the plan without quietly omitting promised components or adding unapproved features, refactors, or layer redesigns.
- The PR description accurately reflects the actual modifications in the diff rather than claiming fixes that were never implemented or omitting changes that exist in the code.
- Any discrepancy between the original plan and the final diff is explicitly acknowledged and technically justified under a dedicated deviations heading or note.

## Test evidence (harness category: not-tested)

### Where it lives
- **Eval package**: The `test-evidence` section and `reproduction` block, compared against the plan's proposed test plan and expected outcomes.
- **Live mode**: `test_evidence.md` (or the Testing section of `pr_draft.md`), comparing captured command outputs against the test plan in `plan.md` and repository requirements.

### What good looks like
- Direct observable proof is provided showing the failure reproduced before the change and resolved after the change (e.g., terminal output showing an assertion failure or error log, followed by the passing test suite run).
- The repo's standard quality suites (e.g., `make test-unit`, `make test-integration`, `make lint`, `make typecheck`) are executed and their terminal summaries included, or any skipped/failing suite is explicitly documented with technical rationale.
- The evidence contains concrete test names, counts, and exit statuses rather than vague assertions like "all tests pass" or empty template checkboxes.

## Diff quality (harness category: unreviewable)

### Where it lives
- **Eval package**: The unified diff block and commit history.
- **Live mode**: The output of `git diff main...HEAD` across all modified files and `git log main..HEAD --oneline`.

### What good looks like
- The diff is tightly scoped to the minimal change set required to solve the target issue.
- The diff is completely free of debugging leftovers (such as `console.log`, `print()`, `breakpoint()`, or temporary inspection dumps).
- There is no commented-out dead code, temporary scratchpad logic, merge conflict marker debris, or accidental mass formatting/whitespace churn across unmodified lines.
- Commits are coherent and focus directly on the fix without drive-by refactors of peripheral subsystems.

## Standards and comms (harness category: standards-wall)

### Where it lives
- **Eval package**: The `repo-facts` block (template requirements, contributing rules, and AI disclosure policy), compared against the candidate PR title, body, and issue references.
- **Live mode**: Upstream `.github/PULL_REQUEST_TEMPLATE.md` and `docs/CONTRIBUTING.md`, compared against `pr_draft.md` and your branch name.

### What good looks like
- Every section required by the repository's PR template is present and contains substantive, non-placeholder content.
- The target issue is explicitly linked using the repository's expected closing syntax (e.g., `Closes #<id>` or `Fixes #<id>`).
- An explicit, plain-language AI-use disclosure is included detailing how AI tooling was utilized on the change, satisfying course and repo policy.
- The branch name adheres to repository naming conventions (e.g., `<type>/<issue-number>-<slug>`).