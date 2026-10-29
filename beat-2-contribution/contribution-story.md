# Contribution: Migrate bt_home_hub_5 from DeviceScanner to ScannerEntity

Path: `beat-2-contribution/contribution-story.md`

**Contribution Number:** 1
**Student:** Stephen Agyemang
**GitHub Username:** Stephen-Agyemang
**Issue:** https://github.com/home-assistant/core/issues/143022
**Pull Request:** [GitHub PR link, once opened]

---

## Toolkit

This section tracks the weekly development of my agent toolkit across units 5 through 8, documenting what was built, what broke during real evaluation runs, and what changed as a result.

**Unit 5: Upgraded issue-select to issue-scout with CLI search strategy and GitHub-wide scope**

Built an automated issue discovery workflow by upgrading issue-select into issue-scout, linking gh search queries directly to my Python and Java backend profile sentences in scope.md. During initial scans, ranking by fewest comments surfaced zero-comment repositories created the same day with no community adoption (failing repo-in-use). In addition, a naive substring filter on "ui" inadvertently dropped issues labeled "enquiry". 

To fix the filter, I patched it to enforce exact label matching, verifying that "enquiry" issues were preserved while pure UI labels were excluded. Furthermore, evaluation across `apache/airflow` candidates revealed that every issue with 3 or more comments already had active competing pull requests. I instituted a strict 3-comment cutoff rule in the scout rubric to detect candidate contention before multi-contributor race conditions form.

**Unit 6: Installed voice-guard hook and configured repository memory for home-assistant**

Integrated a PreToolUse voice-guard hook into settings.json to enforce concrete claims and prevent unvalidated comment submissions. During the wiring check, the hook successfully intercepted a body-less issue comment command, verifying the guard's gatekeeper behavior. In core, I configured CLAUDE.local.md with targeted pytest commands for the bt_home_hub_5 component and added it to .git/info/exclude to ensure local tooling configuration remains completely detached from upstream commits.

**Unit 7: [one line on the tool work]**

[2-4 sentences: the ship-gate and your gate manifest. What it produced
on your real branch, what fought you, what changed because of it.]

**Unit 8: [one line on the tool work]**

[2-4 sentences: the PR-monitor and your playbook. What it read, what it
drafted, what you did with the draft.]

---

## Why I Chose This Issue

I began by searching the portal for Python backend issues, initially focusing on high-profile repositories like `apache/airflow`. When running `issue-scout` over those picks (#72265, #60172, and #66173), the tool rejected all three: one issue had 8 competing pull requests open simultaneously, while the other two had accumulated five closed, unmerged PRs each, signaling severe claimant contention and hidden architectural traps. Shifting focus to `home-assistant/core`, I graded three sibling device-tracker migration tasks under epic #50326 (#143022, #143039, and #143040). While the scout rejected the latter two due to stale and rejected PR history under `settled-spec`, it accepted #143022 with an `ambitious` difficulty note. The issue was completely unworked—no assignees, no linked PRs, and zero comments besides the initial docs bot.

This issue aligns directly with my Python backend profile and interest in modular device integrations. Although the issue description was only two lines, grading the three sibling issues together provided crucial insight: maintainers had rejected prior attempts on sibling integrations because contributors attempted a naive one-line class swap without introducing config entries. Because `ScannerEntity` strictly requires config entry support, this project gives me the opportunity to build the integration's first configuration flow (`config_flow.py`) and write its inaugural unit test suite with `pytest`. The scout proved its value by steering me away from the crowded race conditions on Airflow, while the `scope-feasible` patch correctly surfaced the open architectural scope as a difficulty note rather than rejecting the issue outright.

**Claim Comment:** https://github.com/home-assistant/core/issues/143022#issuecomment-3444983086

**Toolkit**

- Tool: issue-scout (search strategy, patched rubric with `scope-feasible` check and difficulty notes).
- What it produced: An `accept` verdict on `home-assistant/core#143022` with an `ambitious` difficulty note, and `reject` verdicts on sibling issues #143039 and #143040 due to closed unmerged PRs under `settled-spec`.
- Difficulties: Two-line task descriptions caused the preferred `actionable-description` check to fail, and the true requirement (config flow refactoring) was invisible in the issue body, existing only in sibling PR review discussions.
- What changed: Upgraded the scope check from `newcomer-scope` to `scope-feasible` to flag open design edges as difficulty notes rather than failing valid issues, and graded sibling issues concurrently to extract maintainer architectural expectations.

---

## Understanding the Issue

**Problem Description**

The `bt_home_hub_5` integration in Home Assistant currently uses the deprecated legacy `DeviceScanner` interface to track devices connected to BT Home Hub 5 routers. As part of the core architectural cleanup tracked under parent epic #50326, all legacy router scanners must migrate to the modern `ScannerEntity` model. However, `ScannerEntity` cannot be instantiated from legacy YAML configuration platforms; it requires the integration to be managed through Home Assistant's config entries architecture.

**Expected Behavior**

The integration should set up devices via a config flow (`config_flow.py`) where router connection parameters are validated and stored as a config entry. The device tracker platform should then register entities inheriting from `ScannerEntity` instead of implementing `DeviceScanner`, with comprehensive unit test coverage verifying config entry creation and entity updates.

**Current Behavior**

`homeassistant/components/bt_home_hub_5/device_tracker.py` still inherits from `DeviceScanner` and relies on legacy setup methods (`setup_scanner`). The integration lacks a `config_flow.py`, has no config entry handlers in `__init__.py`, and currently contains zero unit tests in `tests/components/bt_home_hub_5/`.

**Affected Components**

- `homeassistant/components/bt_home_hub_5/device_tracker.py`: Houses the current `DeviceScanner` implementation.
- `homeassistant/components/bt_home_hub_5/manifest.json`: Requires integration schema updates and config flow flags.
- `homeassistant/components/bt_home_hub_5/config_flow.py`: New file required to support config entries.
- `homeassistant/components/bt_home_hub_5/__init__.py`: Component entry point required to forward setup to platforms.
- `tests/components/bt_home_hub_5/`: New test directory required for `test_config_flow.py` and `test_device_tracker.py` using mocked `bthomehub5-devicelist` responses.

**Toolkit**

- Tool: Claude Code inspection and GitHub issue/PR cross-referencing.
- What it produced: Concrete architectural mapping of required files (`config_flow.py`, `device_tracker.py`, tests) and identified the absence of existing test coverage in `tests/components/bt_home_hub_5/`.
- Difficulties: The issue body gave no technical pointers; the dependency on `bthomehub5-devicelist` required checking upstream source to understand hardware expectations.
- What changed: Framed the implementation around a mocked router client in `pytest` rather than attempting physical hardware reproduction.

---

## Reproduction Process

**Environment Setup**

- Operating System: macOS
- Repository: Cloned fork of `home-assistant/core` checked out on the `dev` branch at commit `61215ee660b`.
- Inspection Mode: Static inspection of `dev @ 61215ee660b` (`git` and filesystem verification; no runtime needed to confirm the legacy class hierarchy and missing test suite).
- Challenges: Upstream `pyproject.toml` on `dev` recently bumped `requires-python` to `>=3.14.2`, which caused my initial `repro-check` run to fail when I listed a generic `Python 3.12 / 3.13` environment without a commit SHA. Recording the exact upstream commit (`61215ee660b`) and specifying static inspection resolved the environment mismatch.

**Steps to Reproduce**

1. Inspect the legacy device tracker implementation in `homeassistant/components/bt_home_hub_5/`:
   ```bash
   git grep "DeviceScanner" homeassistant/components/bt_home_hub_5/
   ```
2. Verify the presence of configuration flow and modern entity architecture files:
   ```bash
   ls homeassistant/components/bt_home_hub_5/
   ```
3. Check for existing unit test coverage for the integration:
   ```bash
   ls tests/components/bt_home_hub_5/
   ```

**Reproduction Evidence**

- **Commit showing reproduction:** `61215ee660b` (https://github.com/home-assistant/core/commit/61215ee660b)
- **Screenshots/logs:**
  ```text
  $ git grep "DeviceScanner" homeassistant/components/bt_home_hub_5/
  homeassistant/components/bt_home_hub_5/device_tracker.py:    DeviceScanner,
  homeassistant/components/bt_home_hub_5/device_tracker.py:) -> BTHomeHub5DeviceScanner | None:
  homeassistant/components/bt_home_hub_5/device_tracker.py:    scanner = BTHomeHub5DeviceScanner(config[DEVICE_TRACKER_DOMAIN])
  homeassistant/components/bt_home_hub_5/device_tracker.py:class BTHomeHub5DeviceScanner(DeviceScanner):

  $ ls homeassistant/components/bt_home_hub_5/
  __init__.py		device_tracker.py	manifest.json

  $ ls tests/components/bt_home_hub_5/
  ls: tests/components/bt_home_hub_5/: No such file or directory
  ```
- **My findings:** Static verification at commit `61215ee660b` confirms three facts: `BTHomeHub5DeviceScanner` directly subclasses the deprecated `DeviceScanner` on line 37 of `device_tracker.py`, the component directory contains only three files (`__init__.py`, `device_tracker.py`, `manifest.json`) with no `config_flow.py`, and `tests/components/bt_home_hub_5/` does not exist. The integration currently runs exclusively via the legacy platform path rather than `ScannerEntity`.

**Toolkit**

- Tool: `repro-check` skill (upgraded with `scope.md` and `voice-guide.md` to grade `claim-draft.md` and `repro-draft.md` jointly against issue `#143022`).
- What it produced: An initial `reject` verdict flagging `environment-recorded` and an unsourced design assertion, followed by a clean `accept` verdict across all 6 checks (`names-the-issue`, `promise-not-assertion`, `environment-recorded`, `steps-rerunnable`, `observed-matches-issue`, `repo-conventions`) after revision.
- Difficulties: My first draft listed `Python: 3.12 / 3.13` with no commit SHA or OS, which `repro-check` rejected because upstream `dev` now requires Python `>=3.14.2`. It also flagged my closing sentence ("requires both a modern config flow and test suite") as an unsourced design conclusion rather than direct command output.
- What changed: Replaced the generic Python version string with `Static inspection of dev @ 61215ee660b; no runtime needed` on `macOS`, and narrowed the closing finding to state only what the command output directly proves ("currently runs via the legacy platform path rather than `ScannerEntity`").

---

## Solution Approach

**Analysis**

[Your analysis of the root cause: what is causing the issue, traced
to the files and functions involved.]

**Proposed Solution**

[High-level description of your fix approach, and what you are
deliberately NOT changing.]

**Implementation Plan**

1. [Modify file X to do Y]
2. [Add or change Z]
3. [Update tests]

[Record deviations here as they happen: what changed mid-build and
why. A recorded deviation is honest work; silent drift is not.]

**Toolkit**

- Tool:
- What it produced:
- Difficulties:
- What changed:

---

## Testing Strategy

[What observable checks prove the fix: your reproduction steps re-run
against the built change (before and after), and the repo's own test
suite or checks with the outcome shown. "Tests pass" with nothing
observable named is not a strategy.]

**Toolkit**

- Tool:
- What it produced:
- Difficulties:
- What changed:

---

## Implementation Notes

[A running log, added to as you work. Date each entry. What you
built, decisions you made when you made them, wrong turns included.]

**[Date]**

[Entry]

**[Date]**

[Entry]

**Toolkit**

- Tool:
- What it produced:
- Difficulties:
- What changed:

---

## Pull Request

**PR Link:** [GitHub PR URL when opened]

**PR Description:** [what the PR says; much of the content above
adapts into it]

**Maintainer Feedback:**

- [Date]: [feedback received, or "none yet"; silence is a normal
  state and costs you nothing]
- [Date]: [how you addressed it]

**Status:** [Awaiting review / Iterating / Merged / Closed. The
status is a fact you record, never a grade: the PR is graded as
opened.]

**Toolkit**

- Tool:
- What it produced:
- Difficulties:
- What changed:

---

## Learnings and Reflections

**Technical Skills Gained**

[What you learned technically, named concretely.]

**Challenges Overcome**

[What was hard and how you solved it; point at the real events, the
Toolkit blocks above are full of them.]

**What I'd Do Differently Next Time**

[Reflection on your process.]

---

## Resources Used

[What actually helped, not a reading list. List things this story
already shows you using: the doc you followed to set the environment
up, the thread that explained the bug, the person who answered you.
One real entry beats ten plausible ones.]

- [Link to documentation that helped]
- [Discussion, issue, or post that helped]
