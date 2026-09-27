# Evidence guide: where proof lives in a reproduction package

## Environment

- **Where it lives**:
  - In an eval bundle: under the candidate repro report's environment section, setup notes, or system block; evaluated against the issue's stated target in the issue body or repo-facts block.
  - In live mode: in the candidate repro draft (`repro.md`) under an environment or system header, compared against the target repository's environment documentation.
- **What good looks like**: The record explicitly names the concrete runtime environment (operating system, runtime/language version, commit SHA or package version, and key dependencies). The versions either match what the issue specifies or explicitly document the difference.

## Steps

- **Where it lives**:
  - In an eval bundle: inside the candidate repro report under steps, commands, or reproduction procedure blocks.
  - In live mode: in the candidate repro draft under the reproduction steps or command walkthrough section.
- **What good looks like**: The steps provide an unambiguous, runnable sequence of commands starting from a clean initial state (e.g., repository checkout/branch, virtual environment activation, installation command) through the trigger invocation. A stranger can copy and execute the commands without guessing missing parameters or implicit setup steps.

## Behavior shown

- **Where it lives**:
  - In an eval bundle: in the candidate repro report under observed output, terminal excerpts, logs, error stack traces, or screenshots; compared against the bug description in the issue block.
  - In live mode: within terminal code fences or output quotes in `repro.md`, read directly against the target issue description on GitHub.
- **What good looks like**: The artifact contains verbatim terminal output, error logs, or failure assertions that directly capture the symptom described in the issue. It verifies the targeted failure rather than an unrelated dependency error, syntax failure, or adjacent bug.

## Honesty

- **Where it lives**:
  - In an eval bundle: in the candidate claim comment's intent statements and the repro report's conclusion or outcome section, compared against the attached evidence logs.
  - In live mode: across the claim draft and repro draft, evaluating whether the asserted status is backed by the included outputs.
- **What good looks like**: The text reports exactly what the evidence demonstrates without overclaiming. An evidenced failure to reproduce (documenting clean execution without errors despite following the issue's steps) is treated as a valid pass, whereas a report asserting successful reproduction while logs show an unrelated error or no reproduction is a fail. Claim comments promise only an investigation rather than guaranteeing a fix or setting an arbitrary delivery date.

## Communications

- **Where it lives**:
  - In an eval bundle: the candidate claim comment read against the issue title and body; the candidate comments read against `CONTRIBUTING.md` and repository policies in the repo-facts block.
  - In live mode: `claim.md` and `repro.md` read against the issue thread, contributor guidelines, PR/issue templates, and repository root documentation.
- **What good looks like**: The claim explicitly references the issue's specific top