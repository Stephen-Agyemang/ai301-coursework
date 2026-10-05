# Evidence guide: where evidence lives in a plan package

## Diagnosis and grounding

- **Where it lives**: Compare the `## Diagnosis` or `Root Cause` section in the candidate plan against the package's `## Reproduction Evidence` (specifically inspecting control runs, timing tables, debug output, and error traces). In live mode, read against the posted reproduction comment and local pytest or runtime output.
- **What good looks like**: The stated cause directly accounts for the observed behavior and is fully consistent with all reproduction evidence. It must never blame a subsystem, flag, or operator that the package's control runs or debug logs explicitly ruled out.

## Scope

- **Where it lives**: The `## Scope` section ("In Scope" and "Out of Scope" / "Not in Scope") and `## Files to Touch` in the candidate plan.
- **What good looks like**: The plan proposes one bounded change focused strictly on resolving the reproduced defect. Unrelated cleanups, dependency migrations, architecture refactors, or UI redesigns are either absent or explicitly marked out of scope.

## Executability

- **Where it lives**: The `## Approach`, `## Files Touched`, and implementation steps in the candidate plan.
- **What good looks like**: Exact file paths and concrete functions or logic modifications are identified such that an unfamiliar developer can immediately begin building without having to make fundamental architectural choices or resolve open questions during the build.

## Test plan

- **Where it lives**: The `## Test Plan` section in the candidate plan read against the steps in the reproduction report.
- **What good looks like**: Provides explicit commands and names decisive, observable outcomes (such as a specific exit code, expected stdout string, or distinct assertion flip). It must not rely on subjective assertions ("should feel fast") or simply state "run the full test suite" without an observable check.

## Honesty

- **Where it lives**: The `## Risks and Unknowns` and `## Deviations` sections of the candidate plan.
- **What good looks like**: Real risks, edge cases, and known limitations are acknowledged transparently rather than presented with unearned certainty. If a trade-off is accepted or work is deferred, the reason is stated explicitly.

## Comms

- **Where it lives**: The candidate plan comment (`comment.md`) read against `## Thread Highlights`, repo facts, and `docs/CONTRIBUTING.md`.
- **What good looks like**: The comment respects maintainer directions present in the thread, engages with existing prior art or related PRs, and adheres strictly to repository contribution guidelines (including mandatory AI-usage disclosures when required by repository policy).