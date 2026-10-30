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

The `bt_home_hub_5` integration instantiates `BTHomeHub5DeviceScanner(DeviceScanner)` on line 32 via legacy YAML platform setup (`get_scanner`). In accordance with core epic #50326, legacy router device trackers must migrate to `ScannerEntity`. Because `ScannerEntity` requires config entry management, the integration must introduce a configuration flow and coordinator while providing a backwards-compatible YAML import path.

According to `homeassistant/components/device_tracker/legacy.py:297-301`, legacy YAML platforms must define one of `async_get_scanner`, `get_scanner`, `async_setup_scanner`, or `setup_scanner`. Removing `get_scanner` without providing an import hook causes `legacy.py` to raise `"Invalid legacy device_tracker platform"` and silently drops existing YAML configurations. 

Following the precedent established in `homeassistant/components/sky_hub/device_tracker.py:31-68` and PR #187258:
- The integration retains `PLATFORM_SCHEMA` in `device_tracker.py` and replaces `get_scanner` with `async_setup_scanner(hass, config, see, discovery_info=None) -> bool`.
- It initiates import via `hass.config_entries.flow.async_init(DOMAIN, context={"source": SOURCE_IMPORT}, data={CONF_HOST: config[CONF_HOST]})`.
- If the import flow aborts (with `cannot_connect` or `unknown`), it registers an ERROR-severity issue `yaml_import_cannot_connect` under `DOMAIN` with placeholder `{"host": config[CONF_HOST]}`.
- If the import succeeds, it deletes any prior `yaml_import_cannot_connect` issue for that host and registers a WARNING-severity `deprecated_yaml_{DOMAIN}` issue under `HOMEASSISTANT_DOMAIN` with placeholders `{"domain": DOMAIN, "integration_title": "BT Home Hub 5"}`.
- In all cases, it returns `True` to satisfy `legacy.py`.

Because Home Assistant policy strictly forbids using mutable hostnames or IP addresses as unique IDs ("Never: an IP address, a device name, a URL, or a hostname"), config entries do not set a unique ID. Instead, duplicate entries are prevented via `self._async_abort_entries_match({CONF_HOST: host})` in both user and import steps, matching `sky_hub`.

Because the underlying library is synchronous and can return `None`, raise `RequestException`, or raise `IndexError` on non-Hub payloads, network calls must run via the executor and handle both connection and unexpected payload errors cleanly.

**Proposed Solution**

Migrate `bt_home_hub_5` from `DeviceScanner` to `ScannerEntity` backed by config entries, following the `sky_hub` architecture:
1. **Config Flow & Duplicate Prevention**: Implement `ConfigFlow` in `config_flow.py` handling `async_step_user` and `async_step_import`. Prevent duplicate configurations using `self._async_abort_entries_match({CONF_HOST: host})` without setting an IP-based unique ID. Add `strings.json` defining schema labels, abort keys (`already_configured`, `cannot_connect`, `unknown`), and the `yaml_import_cannot_connect` issue string.
2. **Legacy YAML Import Hook (`async_setup_scanner`)**: In `device_tracker.py`, keep `PLATFORM_SCHEMA` and implement `async_setup_scanner` passing `data={CONF_HOST: config[CONF_HOST]}` to `SOURCE_IMPORT`. On abort (`cannot_connect` or `unknown`), create `yaml_import_cannot_connect` via `ir.async_create_issue`. On success, clear any previous `yaml_import_cannot_connect` issue and create `deprecated_yaml_{DOMAIN}` under `HOMEASSISTANT_DOMAIN` with `translation_placeholders={"domain": DOMAIN, "integration_title": "BT Home Hub 5"}`. Return `True`.
3. **Connection Validation & Async Executor**: Validate the host by calling `get_devicelist` wrapped in `hass.async_add_executor_job`.
   - If `get_devicelist` returns `None` or raises `requests.exceptions.RequestException`, return/abort `cannot_connect`.
   - If `get_devicelist` raises `IndexError` or any other unexpected `Exception`, log via `_LOGGER.exception` and return/abort `unknown`.
4. **DataUpdateCoordinator**: Introduce a `DataUpdateCoordinator` in `coordinator.py` (or `__init__.py`) polling `get_devicelist` via the executor at `SCAN_INTERVAL`. If `get_devicelist` returns `None`, raises `RequestException`, or encounters an unexpected exception, raise `UpdateFailed`. On `UpdateFailed`, `CoordinatorEntity.available` returns `False`, setting entity states to `STATE_UNAVAILABLE`.
5. **ScannerEntity & Entity Unique IDs**: In `device_tracker.py`, implement `BTHomeHub5ScannerEntity(ScannerEntity, CoordinatorEntity)`. Set `_attr_unique_id = f"{entry.entry_id}_{mac}"` and `_attr_has_entity_name = True`. Register a coordinator listener that dynamically calls `async_add_entities` whenever previously unseen client MAC addresses appear in the coordinator's dictionary. An entity's `is_connected` property returns `True` if its MAC address is in `coordinator.data`, and `False` when absent. Under `entity.py:707-715`, newly tracked entities default to disabled unless another integration has already registered a device with that MAC in the device registry.
6. **Manifest & Component Metadata**: Update `manifest.json` with `"config_flow": true`, `"integration_type": "hub"`, and remove `"quality_scale": "legacy"`.

Deliberately NOT changing:
- The third-party `bthomehub5-devicelist` library dependency.
- Device tracking parsing semantics ({mac: name}).
- Excluding the optional `legacy_known_devices` repair issue check to keep the migration scoped strictly to epic #50326.
- No unrequested code owners or feature scope added.

**Implementation Plan**

1. **Translations & UI Schema (`strings.json`)**: Define data schema labels for `CONF_HOST`, abort and error message keys (`already_configured`, `cannot_connect`, `unknown`), and the `yaml_import_cannot_connect` repair issue string. Core's shared `deprecated_yaml` string is reused under `HOMEASSISTANT_DOMAIN`.
2. **Config Flow (`config_flow.py`)**: Implement `async_step_user` prompting for `CONF_HOST` (defaulting to `"192.168.1.254"`). Check `self._async_abort_entries_match({CONF_HOST: host})` to reject duplicates with `already_configured`. Validate connectivity using `await self.hass.async_add_executor_job(get_devicelist, host)`. If `devicelist is None` or `requests.exceptions.RequestException` is caught, return `errors={"base": "cannot_connect"}`. Catch `IndexError` and unexpected `Exception`, log with `_LOGGER.exception`, and return `errors={"base": "unknown"}`. On success, call `self.async_create_entry(title=f"BT Home Hub 5 ({host})", data={CONF_HOST: host})`. Implement `async_step_import` with identical duplicate check and connectivity validation, aborting with `cannot_connect` or `unknown` on failure, and calling `async_create_entry` on success.
3. **Integration Lifecycle & Coordinator (`__init__.py` / `coordinator.py`)**: Implement `DataUpdateCoordinator` polling `get_devicelist` via executor; raise `UpdateFailed` when `get_devicelist` returns `None`, raises `RequestException`, or raises parsing exceptions. Implement `async_setup_entry` to instantiate the coordinator, perform first refresh, assign to `entry.runtime_data`, and forward entry setup to `Platform.DEVICE_TRACKER`. Implement `async_unload_entry` using `await hass.config_entries.async_unload_platforms_or_raise(entry, [Platform.DEVICE_TRACKER])` matching `sky_hub/__init__.py:94`.
4. **Device Tracker & YAML Import Trigger (`device_tracker.py`)**: Retain `PLATFORM_SCHEMA`. Implement `async_setup_scanner` conforming to `legacy.py:297-301`: call `hass.config_entries.flow.async_init(DOMAIN, context={"source": SOURCE_IMPORT}, data={CONF_HOST: config[CONF_HOST]})`. On import abort (`cannot_connect` or `unknown`), call `ir.async_create_issue(hass, DOMAIN, f"deprecated_yaml_cannot_connect_{config[CONF_HOST]}", is_fixable=False, issue_domain=DOMAIN, severity=ir.IssueSeverity.ERROR, translation_key="yaml_import_cannot_connect", translation_placeholders={"host": config[CONF_HOST]})`. On successful entry creation, call `ir.async_delete_issue(hass, DOMAIN, f"deprecated_yaml_cannot_connect_{config[CONF_HOST]}")` and `ir.async_create_issue(hass, HOMEASSISTANT_DOMAIN, f"deprecated_yaml_{DOMAIN}", is_fixable=False, issue_domain=DOMAIN, severity=ir.IssueSeverity.WARNING, translation_key="deprecated_yaml", translation_placeholders={"domain": DOMAIN, "integration_title": "BT Home Hub 5"})`. Return `True`. Implement `BTHomeHub5ScannerEntity(ScannerEntity, CoordinatorEntity)` with `_attr_unique_id = f"{entry.entry_id}_{mac}"` and `_attr_has_entity_name = True`. Add dynamic entity discovery on coordinator updates for newly seen client MAC addresses. Expose `is_connected`, `mac_address`, and `hostname`.
5. **Manifest (`manifest.json`)**: Set `"config_flow": true`, `"integration_type": "hub"`, and remove `"quality_scale": "legacy"`.
6. **Test Suite (`tests/components/bt_home_hub_5/`)**:
   - `test_config_flow.py`: Assert `async_step_user` with reachable host creates entry; assert duplicate host aborts with `already_configured`; assert `None` returns and `RequestException` surface `cannot_connect`; assert `IndexError` surfaces `unknown`; assert `async_step_import` with valid host creates entry, unreachable host aborts `cannot_connect`, duplicate host aborts `already_configured`.
   - `test_device_tracker.py`: Assert legacy YAML setup via `async_setup_scanner` creates entry and records `deprecated_yaml` in issue registry; assert unreachable host creates `yaml_import_cannot_connect`; using `@pytest.mark.usefixtures("entity_registry_enabled_by_default")`, assert `BTHomeHub5ScannerEntity` initializes `STATE_HOME`, transitions to `STATE_NOT_HOME` when MAC is absent, and transitions to `STATE_UNAVAILABLE` on coordinator `UpdateFailed`.
7. **Repo Verification**: Run unit tests via `pytest tests/components/bt_home_hub_5/` and linting via `uv run --no-sync prek run --all-files`.

**Toolkit**

- Tool: `plan-check` skill (evaluating `plan.md` and `plan-comment.md` against issue #143022 and `sky_hub` PR #187258).
- What it produced: An `accept` verdict validating the complete migration architecture, error routing, repair issue handling, and test fixtures.
- Difficulties: Initial drafts failed on invalid assumptions regarding IP unique IDs (forbidden by HA core rules), missing synchronous executor wrapping, unhandled library `IndexError` exceptions on non-Hub pages, and incomplete YAML migration triggers.
- What changed: Aligned the design with `homeassistant/components/sky_hub`, added executor jobs for synchronous calls, prevented duplicates via `_async_abort_entries_match`, implemented `async_setup_scanner` with issue registry cleanup, and incorporated `@pytest.mark.usefixtures("entity_registry_enabled_by_default")` for decisive tests.

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

- [Home Assistant Contributor Documentation: Config Flow](https://contributors.home-assistant.io/docs/config_entries_config_flow_handler/)
- [Home Assistant Contributor Documentation: Entity Standards](https://contributors.home-assistant.io/docs/core/entity/)
- [Sky Hub ScannerEntity Migration PR #187258](https://github.com/home-assistant/core/pull/187258)
- [Home Assistant Core Epic #50326](https://github.com/home-assistant/core/issues/50326)

