# Contribution: Migrate bt_home_hub_5 from DeviceScanner to ScannerEntity

Path: `beat-2-contribution/contribution-story.md`

**Contribution Number:** 1
**Student:** Stephen Agyemang
**GitHub Username:** Stephen-Agyemang
**Issue:** https://github.com/home-assistant/core/issues/143022
**Pull Request:** https://github.com/home-assistant/core/pull/189792

---

## Toolkit

This section tracks the weekly development of my agent toolkit across units 5 through 8, documenting what was built, what broke during real evaluation runs, and what changed as a result.

**Unit 5: Upgraded issue-select to issue-scout with CLI search strategy and GitHub-wide scope**

Built an automated issue discovery workflow by upgrading issue-select into issue-scout, linking gh search queries directly to my Python and Java backend profile sentences in scope.md. During initial scans, ranking by fewest comments surfaced zero-comment repositories created the same day with no community adoption (failing repo-in-use). In addition, a naive substring filter on "ui" inadvertently dropped issues labeled "enquiry". 

To fix the filter, I patched it to enforce exact label matching, verifying that "enquiry" issues were preserved while pure UI labels were excluded. Furthermore, evaluation across `apache/airflow` candidates revealed that every issue with 3 or more comments already had active competing pull requests. I instituted a strict 3-comment cutoff rule in the scout rubric to detect candidate contention before multi-contributor race conditions form.

**Unit 6: Installed voice-guard hook and configured repository memory for home-assistant**

Integrated a PreToolUse voice-guard hook into settings.json to enforce concrete claims and prevent unvalidated comment submissions. During the wiring check, the hook successfully intercepted a body-less issue comment command, verifying the guard's gatekeeper behavior. In core, I configured CLAUDE.local.md with targeted pytest commands for the bt_home_hub_5 component and added it to .git/info/exclude so my local tooling configuration stays out of upstream commits.

**Unit 7: Validated bt_home_hub_5 migration and PR compliance via ship-gate and ship-manifest**

Configured `ship-manifest.md` to run Home Assistant's mandatory local verification checks (pytest with branch coverage, ruff formatting, and ruff linting) directly using `.venv/bin/` paths. During initial runs, `ship-gate` stopped on case-sensitive title formatting in the manifest, unmocked socket calls during test teardown, and discrepancies between `plan.md` and the PR draft. To pass the gate, I patched `coordinator.get_devicelist` across test fixtures to prevent unmocked socket leaks, added explicit assertions for the repair issue registry and `IndexError` handling, and documented my deviations from the plan in both `plan.md` and `pr_draft.md` while preserving all Home Assistant PR template checklist items verbatim.

**Unit 8: Installed the PR monitor and wrote a playbook from Home Assistant's measured review times**

The PR monitor read PR #189792's checks, review decision, and 10-item thread, and I wrote my playbook from repo data: `CONTRIBUTING.md` states no review wait, and five similar outside-contributor PRs (#187258, #186739, #175799, #181450, #188963) got a first human response in 1 to 7 days, so I set a 10-day first-review threshold (no nudge before 2026-11-18). Its `--state` report labelled my PR FEEDBACK-IN, but all 10 thread items were bots, and the "review" it reacted to was Copilot's "unable to review … insufficient remaining quota" message. Because of that, I added an "only humans count" rule to the playbook, and rewrote it once when my first version contradicted itself about the CLA bot; with that rule, the full run drafted a wait note instead of a reply, and its notes flagged the FEEDBACK-IN label as wrong. I posted nothing and checked the draft's two open questions myself: the CLA is signed (`cla-bot` passing) and the PR is no longer a draft.

---

## Why I Chose This Issue

I began by searching the portal for Python backend issues, initially focusing on high-profile repositories like `apache/airflow`. When running `issue-scout` over those picks (#72265, #60172, and #66173), the tool rejected all three: one issue had 8 competing pull requests open simultaneously, while the other two had accumulated five closed, unmerged PRs each, which suggested heavy competition and design problems that earlier attempts had run into. Shifting focus to `home-assistant/core`, I graded three sibling device-tracker migration tasks under epic #50326 (#143022, #143039, and #143040). While the scout rejected the latter two due to stale and rejected PR history under `settled-spec`, it accepted #143022 with an `ambitious` difficulty note. The issue was completely unworked—no assignees, no linked PRs, and zero comments besides the initial docs bot.

This issue aligns directly with my Python backend profile and interest in modular device integrations. Although the issue description was only two lines, grading the three sibling issues together showed me why earlier attempts failed: maintainers had rejected prior attempts on sibling integrations because contributors attempted a naive one-line class swap without introducing config entries. Because `ScannerEntity` strictly requires config entry support, this project gives me the opportunity to build the integration's first configuration flow (`config_flow.py`) and write its first unit tests with `pytest`. The scout helped by steering me away from the crowded race conditions on Airflow, while the `scope-feasible` patch correctly surfaced the open architectural scope as a difficulty note rather than rejecting the issue outright.

**Claim Comment:** https://github.com/home-assistant/core/issues/143022#issuecomment-6404966273

**Toolkit**

- Tool: issue-scout. I ran it on three `apache/airflow` picks (all rejected for competing or failed PRs), then graded three sibling Home Assistant issues together; it rejected #143039 and #143040 and accepted #143022 with an `ambitious` difficulty note.
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
- **My findings:** Static verification at commit `61215ee660b` confirms three facts: `BTHomeHub5DeviceScanner` directly subclasses the deprecated `DeviceScanner` in `device_tracker.py`, the component directory contains only three files (`__init__.py`, `device_tracker.py`, `manifest.json`) with no `config_flow.py`, and `tests/components/bt_home_hub_5/` does not exist. The integration currently runs exclusively via the legacy platform path rather than `ScannerEntity`.

**Toolkit**

- Tool: `repro-check` skill (upgraded with `scope.md` and `voice-guide.md` to grade `claim-draft.md` and `repro-draft.md` jointly against issue `#143022`).
- What it produced: An initial `reject` verdict flagging `environment-recorded` and an unsourced design assertion, followed by a clean `accept` verdict across all 6 checks (`names-the-issue`, `promise-not-assertion`, `environment-recorded`, `steps-rerunnable`, `observed-matches-issue`, `repo-conventions`) after revision.
- Difficulties: My first draft listed `Python: 3.12 / 3.13` with no commit SHA or OS, which `repro-check` rejected because upstream `dev` now requires Python `>=3.14.2`. It also flagged my closing sentence ("requires both a modern config flow and test suite") as an unsourced design conclusion rather than direct command output.
- What changed: Replaced the generic Python version string with `Static inspection of dev @ 61215ee660b; no runtime needed` on `macOS`, and narrowed the closing finding to state only what the command output directly proves ("currently runs via the legacy platform path rather than `ScannerEntity`").

---

## Solution Approach

**Analysis**

The `bt_home_hub_5` integration instantiates `BTHomeHub5DeviceScanner(DeviceScanner)` via legacy YAML platform setup (`get_scanner`). In accordance with core epic #50326, legacy router device trackers must migrate to `ScannerEntity`. Because `ScannerEntity` requires config entry management, the integration must introduce a configuration flow and coordinator while providing a backwards-compatible YAML import path.

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
4. **DataUpdateCoordinator**: Introduce a `DataUpdateCoordinator` in `coordinator.py` polling `get_devicelist` via the executor at `SCAN_INTERVAL`. If `get_devicelist` returns `None`, raises `RequestException`, or encounters an unexpected exception, raise `UpdateFailed`. On `UpdateFailed`, `CoordinatorEntity.available` returns `False`, setting entity states to `STATE_UNAVAILABLE`.
5. **ScannerEntity & Entity Unique IDs**: In `device_tracker.py`, implement `BTHomeHub5ScannerEntity(ScannerEntity, CoordinatorEntity)`. Set `_attr_unique_id = f"{entry.entry_id}_{mac}"` and `_attr_has_entity_name = True`. Register a coordinator listener that dynamically calls `async_add_entities` whenever previously unseen client MAC addresses appear in the coordinator's dictionary. An entity's `is_connected` property returns `True` if its MAC address is in `coordinator.data`, and `False` when absent. Provide `device_info` linking to network MAC connections.
6. **Manifest & Component Metadata**: Update `manifest.json` with `"config_flow": true`, `"integration_type": "hub"`, and categories.

Deliberately NOT changing:
- The third-party `bthomehub5-devicelist` library dependency.
- Device tracking parsing semantics ({mac: name}).
- Excluding the optional `legacy_known_devices` repair issue check to keep the migration scoped strictly to epic #50326. (Changed on 2026-11-06: I added this repair after all. See "2026-11-06: Pre-Review Comparison Against `sky_hub`" and the documented deviations under Pull Request.)
- No unrequested code owners or feature scope added.

**Implementation Plan**

1. **Translations & UI Schema (`strings.json`)**: Define data schema labels for `CONF_HOST`, abort and error message keys (`already_configured`, `cannot_connect`, `unknown`), and the `yaml_import_cannot_connect` repair issue string. Core's shared `deprecated_yaml` string is reused under `HOMEASSISTANT_DOMAIN`.
2. **Config Flow (`config_flow.py`)**: Implement `async_step_user` prompting for `CONF_HOST` (defaulting to `"192.168.1.254"`). Check `self._async_abort_entries_match({CONF_HOST: host})` to reject duplicates with `already_configured`. Validate connectivity using `await self.hass.async_add_executor_job(get_devicelist, host)`. If `devicelist is None` or `requests.exceptions.RequestException` is caught, return `errors={"base": "cannot_connect"}`. Catch `IndexError` and unexpected `Exception` (`# noqa: BLE001`), log with `_LOGGER.exception`, and return `errors={"base": "unknown"}`. On success, call `self.async_create_entry(title=f"BT Home Hub 5 ({host})", data={CONF_HOST: host})`. Implement `async_step_import` with identical duplicate check and connectivity validation, aborting with `cannot_connect` or `unknown` on failure, and calling `async_create_entry` on success.
3. **Integration Lifecycle & Coordinator (`__init__.py` / `coordinator.py`)**: Implement `DataUpdateCoordinator` polling `get_devicelist` via executor; raise `UpdateFailed` when `get_devicelist` returns `None`, raises `RequestException`, or raises parsing exceptions. Implement `async_setup_entry` to instantiate the coordinator, perform first refresh, assign to `entry.runtime_data`, and forward entry setup to `Platform.DEVICE_TRACKER`. Implement `async_unload_entry` using `await hass.config_entries.async_unload_platforms(entry, [Platform.DEVICE_TRACKER])`.
4. **Device Tracker & YAML Import Trigger (`device_tracker.py`)**: Retain `PLATFORM_SCHEMA`. Implement `async_setup_scanner` conforming to `legacy.py:297-301`: call `hass.config_entries.flow.async_init(DOMAIN, context={"source": SOURCE_IMPORT}, data={CONF_HOST: config[CONF_HOST]})`. On import abort (`cannot_connect` or `unknown`), call `ir.async_create_issue(...)`. On successful entry creation, call `ir.async_create_issue` for `deprecated_yaml` under `HOMEASSISTANT_DOMAIN`. Return `True`. Implement `BTHomeHub5ScannerEntity(ScannerEntity, CoordinatorEntity)` with `_attr_unique_id = f"{entry.entry_id}_{mac}"`, `_attr_has_entity_name = True`, and `device_info` mapping network MAC connections. Add dynamic entity discovery on coordinator updates for newly seen client MAC addresses. Expose `is_connected`, `mac_address`, and `hostname`.
5. **Manifest (`manifest.json`)**: Set `"config_flow": true`, `"integration_type": "hub"`, and categories `["presence_location.network_presence", "computers_network.routers_network"]`.
6. **Test Suite (`tests/components/bt_home_hub_5/`)**:
   - `test_config_flow.py`: Assert `async_step_user` with reachable host creates entry; assert duplicate host aborts with `already_configured`; assert `None` returns and `RequestException` surface `cannot_connect`; assert `IndexError` surfaces `unknown`; assert `async_step_import` with valid host creates entry, unreachable host aborts `cannot_connect`, duplicate host aborts `already_configured`.
   - `test_device_tracker.py`: Assert legacy YAML setup via `async_setup_scanner` creates entry and records `deprecated_yaml` in issue registry; using `@pytest.mark.usefixtures("entity_registry_enabled_by_default")`, assert `BTHomeHub5ScannerEntity` initializes `STATE_HOME`, transitions to `STATE_NOT_HOME` when MAC is absent, and handles coordinator updates via `freezer.tick(SCAN_INTERVAL)` and `async_fire_time_changed(hass)`.
7. **Repo Verification**: Run unit tests via `pytest tests/components/bt_home_hub_5/` and linting via `ruff check` and `hassfest`.

**Toolkit**

- Tool: `plan-check` skill (evaluating `plan.md` and `plan-comment.md` against issue #143022 and `sky_hub` PR #187258).
- What it produced: An `accept` verdict validating the complete migration architecture, error routing, repair issue handling, and test fixtures.
- Difficulties: Initial drafts failed on invalid assumptions regarding IP unique IDs (forbidden by HA core rules), missing synchronous executor wrapping, unhandled library `IndexError` exceptions on non-Hub pages, and incomplete YAML migration triggers.
- What changed: Aligned the design with `homeassistant/components/sky_hub`, added executor jobs for synchronous calls, prevented duplicates via `_async_abort_entries_match`, implemented `async_setup_scanner` with issue registry cleanup, and incorporated `@pytest.mark.usefixtures("entity_registry_enabled_by_default")` for decisive tests.

---

## Testing Strategy

To verify the migration without requiring physical router hardware, the testing strategy relies on `pytest` combined with Home Assistant's test fixtures and socket blocking verification.

### 1. Observable Reproduction Steps Re-run
Re-running the static inspection commands confirmed the elimination of the legacy pattern and the presence of the full modern integration architecture:
```bash
# Verify DeviceScanner is no longer implemented
$ git grep "class BTHomeHub5DeviceScanner" homeassistant/components/bt_home_hub_5/
# (Returns 0 matches; replaced by BTHomeHub5ScannerEntity)

# Verify component structure
$ ls homeassistant/components/bt_home_hub_5/
__init__.py        config_flow.py     const.py           coordinator.py     device_tracker.py  manifest.json      strings.json

# Verify test suite presence
$ ls tests/components/bt_home_hub_5/
__init__.py            test_config_flow.py    test_device_tracker.py
```

### 2. Comprehensive Unit Test & Coverage Execution
Executed the test suite with branch coverage on 2026-11-03, after rebuilding the virtual environment with `script/setup` (CPython 3.14.10, macOS arm64). Output copied from the terminal:
```text
$ uv run --no-sync pytest tests/components/bt_home_hub_5 --cov=homeassistant.components.bt_home_hub_5 --cov-branch --cov-report=term-missing

Test session starts (platform: darwin, Python 3.14.10, pytest 9.0.3, pytest-sugar 1.1.1)
collected 20 items

 tests/components/bt_home_hub_5/test_config_flow.py ✓✓✓✓✓✓✓✓✓                 45% ████▌
 tests/components/bt_home_hub_5/test_device_tracker.py ✓✓✓✓✓✓✓✓✓✓✓           100% ██████████

Name                                                       Stmts   Miss Branch BrPart  Cover   Missing
------------------------------------------------------------------------------------------------------
homeassistant/components/bt_home_hub_5/__init__.py            10      0      0      0   100%
homeassistant/components/bt_home_hub_5/config_flow.py         43      0      4      0   100%
homeassistant/components/bt_home_hub_5/const.py                6      0      0      0   100%
homeassistant/components/bt_home_hub_5/coordinator.py         26      0      2      0   100%
homeassistant/components/bt_home_hub_5/device_tracker.py      61      0      8      0   100%
------------------------------------------------------------------------------------------------------
TOTAL                                                        146      0     14      0   100%

Results (0.77s):
      20 passed
```

Earlier runs on 2026-10-30 and 2026-11-01 reported 15 tests and 99% coverage (two partial branches in `device_tracker.py`). The tests and fixes added since then, recorded in Implementation Notes, brought it to the 20 tests and 100% shown above.

### 3. Static Analysis and Linting
```text
$ python3 -m script.hassfest
...
Integrations: 1563
Invalid integrations: 0

$ git diff upstream/dev -- homeassistant/generated/integrations.json
@@ -1482,7 +1482,7 @@
         "presence_location.network_presence",
         "computers_network.routers_network"
       ],
-      "config_flow": false,
+      "config_flow": true,
       "iot_class": "local_polling"
     },

$ uv run --no-sync prek run --all-files
ruff check...............................................................Passed
ruff format..............................................................Passed
codespell................................................................Passed
...
mypy.....................................................................Failed
  homeassistant/util/hass_dict.pyi: error: Duplicate module named "homeassistant.util.hass_dict" (also at "homeassistant/util/hass_dict.py")
pylint...................................................................Failed
  tests/test_loader.py:137:4: E0611: No name 'test_package' in module 'custom_components' (no-name-in-module)
...
hassfest.................................................................Passed
```

The mypy and pylint failures are in files this branch does not touch (`git diff --name-only upstream/dev...HEAD` lists none of them). They appear because `--all-files` passes every file in the repo to each tool at once. Running both tools on only the changed files, which is what the hooks do on a normal commit:
```text
$ git diff --name-only upstream/dev -- '*.py' | grep ^homeassistant | xargs mypy
Success: no issues found in 6 source files

$ git diff --name-only upstream/dev -- '*.py' | xargs pylint --ignore-missing-annotations=y; echo "exit code: $?"
exit code: 0
```

**Toolkit**

- Tool: `pytest` with `pytest-cov` and Home Assistant's test fixtures, plus `ruff` through `prek`. I ran the `bt_home_hub_5` tests with `--cov-branch --cov-report=term-missing` and used the Missing column to find the untested branches in `device_tracker.py`, then ran `prek run --all-files` (`ruff check`, `ruff format`, `hassfest`) before pushing.
- What it produced: 20 passing unit tests spanning config flow, coordinator polling, entity states, two-hub unique IDs, and legacy YAML import, with 100% statement and branch coverage across all five component files (146 statements, 14 branches) as of 2026-11-03.
- Difficulties: The test teardown fixture `verify_cleanup` blocked unmocked network sockets (`HASocketBlockedError`) when `async_setup_scanner` and user config flows triggered background coordinator setups that invoked `get_devicelist`. Additionally, `check_translations` failed due to missing translation definitions for `issues.deprecated_yaml.title`.
- What changed: Patched `get_devicelist` across both `config_flow` and `coordinator` during config flow and YAML setup tests, properly structured issue keys in `strings.json`, and generated translation JSON via `script.hassfest`.

---

## Implementation Notes

**2026-10-30: Architecture Scaffolding and Component Setup**
- Checked out branch `migrate/143022-bt-home-hub-5-scanner-entity` from `dev`.
- Updated `manifest.json` to register `"config_flow": true`, set `"integration_type": "hub"`, and remove `"quality_scale": "legacy"`.
- Created `const.py` with `DOMAIN = "bt_home_hub_5"`, `DEFAULT_HOST = "192.168.1.254"`, and `SCAN_INTERVAL = timedelta(seconds=10)`.
- Created `coordinator.py` defining `BTHomeHub5DataUpdateCoordinator` wrapping synchronous `get_devicelist` calls inside `hass.async_add_executor_job` with error handling converting failures to `UpdateFailed`.
- Created `__init__.py` implementing `async_setup_entry` and `async_unload_entry`.

**2026-10-30: Config Flow and Schema Migration**
- Built `config_flow.py` supporting `async_step_user` and `async_step_import` with host connection validation and duplicate detection via `_async_abort_entries_match`.
- Initially used `import voluptuous as vol` and `from __future__ import annotations`. Ruff rejected both because `pyproject.toml` bans them under `[tool.ruff.lint.flake8-tidy-imports.banned-api]`: line 872 says `"voluptuous".msg = "use probatio instead"` and line 879 bans `"__future__.annotations"` because Home Assistant requires Python 3.14+. Refactored schemas to use `probatio.Schema` and `probatio.Required`.

**2026-10-30: Device Tracker Platform & Entity Implementation**
- Replaced legacy `DeviceScanner` in `device_tracker.py` with `BTHomeHub5ScannerEntity(CoordinatorEntity, ScannerEntity)`.
- Implemented dynamic entity addition using coordinator update listeners to register newly discovered MAC addresses without requiring integration reloads.
- Replaced `setup_scanner` with `async_setup_scanner` to intercept legacy YAML configurations, register a deprecation issue in the issue registry, and forward parameters to `SOURCE_IMPORT`.

**2026-10-30: Test Suite Authoring and Pre-PR Hardening**
- Authored `tests/components/bt_home_hub_5/test_config_flow.py` and `tests/components/bt_home_hub_5/test_device_tracker.py`.
- Encountered `HASocketBlockedError` during teardown of `test_async_setup_scanner_legacy` because the background import flow triggered an unmocked coordinator fetch. Added multi-patch context managers to intercept both config flow and coordinator network calls.
- Resolved missing translation failures by adding `deprecated_yaml` under the `issues` object in `strings.json` and running `script.translations develop`.
- Formatted all files with `ruff format` and verified clean lint checks with `ruff check`. Statement coverage was 100%, but branch coverage was 99% because two branches in `device_tracker.py` were only partly covered.

**2026-11-01: Upstream CI Fixes and Device Registry Standards**
- Worked through upstream CI failures. As of 2026-11-03, `gh pr checks 189792 -R home-assistant/core` on the last pushed commit shows `Check mypy`, `Check pylint`, `Check pylint on tests`, and `Run prek checks` passing, but `Check hassfest` and `Run tests Python 3.14.10 (bt_home_hub_5)` still failing (19 pass, 2 fail, 16 skipping).
- Added mandatory `device_info` with `CONNECTION_NETWORK_MAC` on `BTHomeHub5ScannerEntity` to satisfy Home Assistant core registry standards.
- Sorted `strings.json` keys alphabetically and regenerated `homeassistant/generated/config_flows.py` and `homeassistant/generated/integrations.json`. The committed `integrations.json` did not match what a full `hassfest` run generates, which is one reason `Check hassfest` kept failing. That file was fixed on 2026-11-03.
- Verified all 15 unit tests pass with 100% statement coverage and 99% branch coverage.

**2026-11-01: Removal Version, Device Naming, and YAML Default Host Regression**
- Moved `breaks_in_ha_version` from `2026.12.0` to `2027.6.0` after confirming that the contributor docs require at least six months of deprecation for YAML configuration.
- Set `name=name` in `DeviceInfo`, `_attr_name = None`, and `_attr_hostname = name` so entities resolve to `device_tracker.phone` instead of `device_tracker.bt_home_hub_5_192_168_1_254_phone`, and removed the custom `hostname` property.
- Traced a regression caused by dropping `PLATFORM_SCHEMA`: YAML with no `host` lost the `192.168.1.254` default, failed with `KeyError: 'host'`, and created no config entry. My `pr-precheck` had flagged the possible loss of the default host as a caveat before the PR opened.
- Restored `PLATFORM_SCHEMA` with `DEFAULT_HOST`, matching `sky_hub`, and added `test_yaml_setup_uses_default_host`, which failed before the fix (`1 failed, 15 passed`) and passed after it (`16 passed`, 100% statement and branch coverage).

**2026-11-01: Failed YAML Import Reporting**
- Traced a second regression from my deviation to a single unconditional `deprecated_yaml` issue: when the router could not be reached, no config entry was created, but the repair notice still said the configuration had been imported automatically.
- Returned to the `sky_hub` design from my plan: `async_setup_scanner` now awaits the import flow, raises an ERROR-severity `yaml_import_cannot_connect_{host}` issue when the flow aborts with `cannot_connect` or `unknown`, and otherwise clears that issue and raises `deprecated_yaml_bt_home_hub_5` under `HOMEASSISTANT_DOMAIN`.
- Replaced the custom `deprecated_yaml` string in `strings.json` with `yaml_import_cannot_connect`, reused core's shared `deprecated_yaml` string, and regenerated `translations/en.json` with `script.translations develop`.
- Added `test_yaml_import_failure_creates_issue`, parametrized for `cannot_connect` and `unknown`, and updated `test_async_setup_scanner_legacy`. Before the fix, `3 failed, 15 passed`; after it, `18 passed` with 100% statement and branch coverage.

**2026-11-01: Blank Device Name Fallback**
- Confirmed that a blank router name made `DeviceInfo` fall back to the config entry title, producing `device_tracker.bt_home_hub_5_192_168_1_254` instead of an ID for that device. I did not confirm whether `bthomehub5-devicelist` can return a blank name, so this change is defensive.
- Changed `BTHomeHub5ScannerEntity` to use `name or mac` for the device name and `name or None` for the hostname, matching `sky_hub`.
- Added `test_device_without_name_falls_back_to_mac`. Before the fix, `1 failed, 18 passed`; after it, `19 passed` with 100% statement and branch coverage.

**2026-11-03: Entity Unique IDs Per Hub**
- Dropped my deviation from the planned `{entry_id}_{mac}` unique ID. With `mac` alone, two configured hubs that saw the same device clashed: the second hub's entity was ignored and logged `Platform bt_home_hub_5 does not generate unique IDs. ID 00:11:22:33:44:55 already exists - ignoring device_tracker.phone`.
- Setting `_attr_unique_id = f"{coordinator.config_entry.entry_id}_{mac}"` alone did not fix it, because `ScannerEntity.unique_id` in `homeassistant/components/device_tracker/entity.py:574-576` returns `self.mac_address` and ignores `_attr_unique_id`. Added a `unique_id` property override that returns `_attr_unique_id`, matching `sky_hub`.
- Added `test_same_device_on_two_hubs`. Before the fix, `1 failed, 19 passed`; after it, `20 passed` with 100% statement and branch coverage.

**2026-11-03: Logging Unexpected Config Flow Errors**
- Found that both `async_step_user` and `async_step_import` caught unexpected exceptions with `except Exception:  # noqa: BLE001` and returned `unknown` without logging anything, even though my plan said to log them with `_LOGGER.exception`. I had not listed this as a deviation. The user-facing consequence was an "Unexpected error" message with nothing in the Home Assistant log to explain it.
- Added `_LOGGER.exception("Unexpected exception")` to both handlers and removed the `noqa`. This is the common pattern in other config flows (`git grep -h -A1 "except Exception:$" -- 'homeassistant/components/*/config_flow.py' | grep -c '_LOGGER.exception("Unexpected exception")'` returned `327`). Ruff's `BLE001` rule does not flag a blind `except` that logs with `.exception`.
- Updated `test_user_flow_unknown_exception` and `test_import_flow_unknown_error` to assert the log message with `caplog`. Before the fix, `2 failed, 18 passed`; after it, `20 passed` with 100% statement and branch coverage. `uvx ruff@0.17.3 check` printed `All checks passed!` and `format --check` printed `8 files already formatted`, because the project venv's ruff is 0.17.1 and refuses to run.

**2026-11-03: Local Toolchain Rebuild and Regenerated Files**
- My local tools did not match CI: the venv ran Python 3.14.7 instead of the required 3.14.10, ruff 0.17.1 refused to run against the required `>=0.17.3`, and `python3 -m script.hassfest` failed with `No module named 'hassfest'`. Upgraded `uv`, deleted `.venv`, and reran `script/setup`.
- The first `script/setup` failed to build `dtlssocket==0.1.19` (pulled in by `pytradfri[async]`, not my integration) with `sh: autoconf: command not found`. Installed it with `brew install autoconf` and reran setup.
- Ran the full `python3 -m script.hassfest`: `Integrations: 1563`, `Invalid integrations: 0`. `git diff upstream/dev -- homeassistant/generated/integrations.json` now shows only `"config_flow": false` changing to `true` for `bt_home_hub_5`.
- `uv run --no-sync prek run --all-files` passed every hook except `mypy` and `pylint`, which failed on files this branch does not touch. Running them on only my changed files passed (details in Testing Strategy).

**2026-11-03: Push, Merge Conflict, and Green CI**
- Committed the 2026-11-01 to 2026-11-03 fixes as `dfc7f96` ("Fix YAML import, entity IDs, and error logging in bt_home_hub_5") and pushed.
- GitHub then reported conflicts in `homeassistant/components/bt_home_hub_5/manifest.json` and `homeassistant/generated/integrations.json`. Upstream #189910 had added `"connectivity": ["network"]` to my manifest, next to the lines I changed. 328 upstream commits from 86 contributors had landed since my last sync on 2026-11-01.
- Before touching my branch, I tested the resolution in a separate worktree. Then I ran `git merge upstream/dev` (merge, not rebase, because the repo's `CLAUDE.md` forbids rewriting pushed commits), kept both `"config_flow": true` and `"connectivity": ["network"]` in the manifest, took upstream's `integrations.json` with `git checkout --theirs`, and regenerated it with `python3 -m script.hassfest`. `git diff upstream/dev -- homeassistant/generated/integrations.json` again showed only `"config_flow": false` changing to `true`.
- After the merge, pytest crashed with `ModuleNotFoundError: No module named 'pytest_homeassistant.bootstrap'`. Upstream #190049 had renamed `packages/pytest-home-assistant` to `packages/pytest-homeassistant`, and my venv's editable install still pointed at the old folder. The import name did not change, so my tests needed no edits. Rerunning `script/bootstrap` fixed the venv, and the tests passed (`20 passed`).
- Committed the merge as `a26f753` and pushed. `gh pr checks 189792 -R home-assistant/core --watch` reported `All checks were successful` with `0 cancelled, 0 failing, 22 successful, 15 skipped, and 0 pending checks`, including `CI/Check hassfest` and `CI/Run tests Python 3.14.10 (bt_home_hub_5)`.

**2026-11-06: Pre-Review Comparison Against `sky_hub`**
- Before marking the PR ready, I compared every file against the merged `sky_hub` migration (#187258) and the `bthomehub5-devicelist` source, and found three gaps.
- Leftover YAML trackers: after migration, `async_setup_scanner` no longer calls `see()`, so the old `known_devices.yaml` entities stop updating and are marked stale, which shows them as away (`homeassistant/components/device_tracker/legacy.py:967-968`). The `sky_hub` docs page also says leftover entries keep the same entity IDs, so Home Assistant drops the new entities at startup. Either way, a user's "phone left home" automation would silently stop working. I first wrote the repair text to say the old trackers "will show as away" and to update automations to the new entities, but after reading the `sky_hub` docs I switched to `sky_hub`'s wording: the entries "can conflict with the new entities and prevent them from updating", and the fix is to remove them and restart. `sky_hub` has handled this since #187258 with a `legacy_known_devices` repair issue. I ported that check to `__init__.py`: on setup, and whenever the router reports a new device, it loads `known_devices.yaml` and raises an ERROR repair listing every tracked entry whose MAC the router reports. The issue is tied to the config entry, so removing the entry removes it. Added `tests/components/bt_home_hub_5/test_init.py` with seven tests. Before the fix, `7 failed, 20 passed`; after it, `27 passed`.
- Unload: `async_unload_entry` ignored the `False` that `async_unload_platforms` returns on failure, so Home Assistant would report a failed unload as successful. Switched to `async_unload_platforms_or_raise` (added to core on 2026-10-13 in #185989, and what `sky_hub` uses). Added `test_unload_entry_platform_failure`. Before, `1 failed, 27 passed` (`assert not True`); after, `28 passed`.
- Coordinator errors: `_async_update_data` caught `except Exception` and turned it into a one-line `UpdateFailed`, which hid the traceback that `DataUpdateCoordinator` logs for unexpected errors (`homeassistant/helpers/update_coordinator.py:582-585`). The library only raises `requests` errors (it handles timeouts itself) or `IndexError` from `p.tables[9]` when the page is not a Home Hub page, so I narrowed the catch to `IndexError`. Added `test_coordinator_unexpected_error`. Before, `1 failed, 28 passed`; after, `29 passed`. Setup still retries on an unexpected error; the difference is that the log now shows the full traceback.
- Manual test without hardware: ran `python3 -m homeassistant -c ./config` on my branch. Adding BT Home Hub 5 from the UI showed the host form with `192.168.1.254` pre-filled, and submitting it showed "Failed to connect" after the library logged `Connection to the router timed out` (`connect timeout=5`). Then I added `device_tracker: - platform: bt_home_hub_5` with no `host` to `config/configuration.yaml` and restarted. Settings → Repairs showed "YAML configuration import failed" (Error, by BT Home Hub 5), no "YAML configuration is deprecated" warning, and no config entry was created.
- Final local results: `29 passed`, 100% statement and branch coverage (171 statements, 18 branches). `ruff check` printed `All checks passed!`, `ruff format --check` printed `9 files already formatted`, `mypy` printed `Success: no issues found in 5 source files`, `pylint` exited `0`, and `python3 -m script.hassfest` printed `Invalid integrations: 0`.

**2026-11-06: Plan Check and Documentation Update**
- Before opening the docs PR, I checked my plan against what I had built. My plan never mentioned docs, and it explicitly scoped out the `legacy_known_devices` repair that I added earlier the same day. I had explained that change in the entry above, but I had not logged it as a deviation, and the "Deviations" section of `plan.md` still listed four changes I had since undone. I rewrote that section to show the current deviations and the reverted ones separately, and added the missing ones under Pull Request.
- I decided to keep the repair and add a docs PR instead of stopping at the original scope. The developer checklist says "Documentation is developed for home-assistant.io", my core PR template has a docs checkbox for user-facing setup changes, and the live BT Home Hub 5 page still only explains YAML setup, which this PR deprecates.
- Forked `home-assistant/home-assistant.io` and branched `bt-home-hub-5-config-flow` from `next`. Rewrote `source/_integrations/bt_home_hub_5.markdown` following the merged `sky_hub` page and the repo's integration template: UI setup with a Host field, Supported functionality, Data updates (polled every 10 seconds), a Troubleshooting entry for "Remove leftover known devices for BT Home Hub 5", and Removing the integration. I checked each claim against my core code. I left out the `sky_hub` page's "disabled by default unless their MAC address matches a device already known" because I could not find that behavior in current core: `ScannerEntity` just sets `_attr_entity_registry_enabled_default = False`.
- The page's title and `ha_` keys are no longer written by hand. `plugins/integration_metadata.rb` merges them from the core manifest, so the front matter only keeps `description`, `ha_release`, and `ha_domain`.
- `npx textlint` and `npx remark --quiet --frail` both exited `0` on the page.
- Opened the docs PR against `next`: https://github.com/home-assistant/home-assistant.io/pull/50303. The bot added the `has-parent` label, linking it to core PR #189792. I added the link to my core PR description and ticked its documentation box.

**2026-11-08: Ready for Review and PR Monitor**
- Went through every changed file line by line, ticked "Any generated code has been carefully reviewed", and left "The code change is tested and works locally" unticked because I have no router. Marked the PR ready for review at 08:00 UTC. That cleared the CLA bot's old "Changes requested", and the PR now shows "Review required" with 27 checks passing and 15 skipped on `b43c87b8df5`.
- Installed the Unit 8 PR monitor. `--state` found my PR but labelled it FEEDBACK-IN, even though every one of the 10 thread items came from a bot. The "review" it reacted to was Copilot's quota message.
- Wrote my playbook from Home Assistant data. `CONTRIBUTING.md` states no review wait. Five similar outside-contributor PRs (#187258, #186739, #175799, #181450, #188963) got their first human response in 1 to 7 days, but #175799 then took 108 days to merge. Set 10 quiet days for a first review (from 2026-11-08) and 14 for a stalled one. Added a rule that only humans count toward quiet days, then fixed it when my first version contradicted itself about the CLA bot.
- The full run drafted a wait note: nothing to post, next check 2026-11-18. I posted nothing and confirmed its two open questions with `gh`: the `cla-bot` check passes and the PR is no longer a draft.
- Updated the PR description without removing any template text. Under the empty Breaking change heading, I added a note that this is not a breaking change, because YAML is imported automatically and its removal follows the deprecation period (2027.6.0). After the Testing paragraph, I added a collapsed "Before/after and test output" block: `git grep` finds `BTHomeHub5DeviceScanner(DeviceScanner)` on `dev` and nothing on my branch, `git ls-tree` shows the new config flow, coordinator, and test files, and a fresh `pytest` run on `b43c87b8df5` gives 29 passed with 100% statement and branch coverage (171 statements, 18 branches).

**Toolkit**

- Tool: `ruff`, `hassfest`, and Home Assistant's test runner. I ran `ruff` and the integration's tests before every commit, and ran `python3 -m script.hassfest` after editing `manifest.json` or `strings.json` to regenerate `integrations.json` and the translations, then checked the result with `git diff upstream/dev`.
- What it produced: Code that passes `ruff`, `mypy`, `pylint`, and `prek` in upstream CI. Locally, a full `hassfest` run and the tests pass on Python 3.14.10. Before the 2026-11-03 push, upstream `Check hassfest` and the `bt_home_hub_5` test job failed. The test job failed on `test_device_tracker_entities` (`assert None is not None` at `test_device_tracker.py:47`, because the expected entity did not exist), which the 2026-11-01 naming fix addresses. After pushing the fixes and merging `upstream/dev`, all 22 upstream checks that ran passed, and 15 were skipped.
- Difficulties: Environment mismatches with `uv` attempting to fetch Python interpreter versions, strict JSON key ordering in `strings.json`, and an outdated local venv (Python 3.14.7, ruff 0.17.1, no `hassfest` module) that hid problems CI caught.
- What changed: First worked around the venv by calling `.venv/bin/python3` directly. Because a full `hassfest` run could not work in that venv, nothing local caught the stale `integrations.json`. Then rebuilt the venv with `script/setup` so local checks match CI.

---

## Pull Request

**PR Link:** https://github.com/home-assistant/core/pull/189792

**Docs PR Link:** https://github.com/home-assistant/home-assistant.io/pull/50303

**PR Description** (the "Proposed change" section as posted on 2026-11-08; the rest of the PR template is filled in on GitHub):

This migrates `bt_home_hub_5` from the legacy `DeviceScanner` to a config entry with `ScannerEntity` entities. I followed the `sky_hub` migration in #187258, since both are router-based device trackers.

What changes:
- The integration can be set up from the UI. The config flow checks that the router answers before creating an entry, and aborts if that host is already set up.
- A `DataUpdateCoordinator` polls the router's device list every 10 seconds, running the blocking `get_devicelist` call in the executor.
- Each device the router reports becomes a `ScannerEntity` with unique ID `{entry_id}_{mac}`, so the same device seen by two hubs doesn't clash. New devices are added without a reload. Like other `ScannerEntity` integrations, these entities are disabled by default.

What existing YAML users see:
- Their `device_tracker` YAML is imported into a config entry automatically on startup. If `host` is missing, it falls back to `192.168.1.254`, the same default as before.
- If the import works, they get the standard "YAML configuration is deprecated" repair issue, with removal in 2027.6.0.
- If the router can't be reached during import (or the import hits an unexpected error), no entry is created and they get a "YAML configuration import failed" repair issue instead, so they aren't told the import worked when it didn't.
- Leftover `known_devices.yaml` entries can conflict with the new entities. If any tracked entry matches a device the router reports, they get a "Remove leftover known devices" repair issue listing those entries, the same check `sky_hub` uses.

Testing: I don't own a BT Home Hub 5, so this is tested with mocked `get_devicelist` responses only. 29 tests pass with 100% statement and branch coverage for the integration. I also ran it locally with no router on my network: adding it from the UI shows "Failed to connect", and a YAML `device_tracker` entry with no `host` shows the "YAML configuration import failed" repair without creating an entry. If an owner is able to try it on real hardware, that would be very welcome.

I used AI Tools as an assistant while writing and testing this. I reviewed the changes and can explain it thoroughly.

### Documented Deviations from Initial Plan:
- **Platform Unload**: Used the shared `PLATFORMS` constant from `const.py` instead of the planned `[Platform.DEVICE_TRACKER]` list. Reason: setup and unload read the same list, so they cannot drift apart. User-facing consequence: none, because `PLATFORMS` contains only `Platform.DEVICE_TRACKER`.
- **Leftover Known Devices Repair**: Added the `legacy_known_devices` repair issue that my plan had scoped out. Reason: after the migration, old `known_devices.yaml` entries for the same devices conflict with the new entities, and `sky_hub` handles this the same way. User-facing consequence: users with leftover entries see an error repair listing them, instead of trackers that quietly stop updating.
- **Coordinator Errors**: My plan turned every unexpected exception into `UpdateFailed`. The coordinator now only catches `IndexError` and leaves other errors to `DataUpdateCoordinator`. Reason: the broad catch hid the traceback. User-facing consequence: none for entity states; the log now shows the full traceback.
- **Unload**: My plan used `async_unload_platforms`; the code uses `async_unload_platforms_or_raise`. Reason: the old code ignored a `False` return. User-facing consequence: a failed unload is reported as failed instead of successful.
- **Documentation PR**: My plan did not include a home-assistant.io update. Reason: the developer checklist and the core PR template expect docs for user-facing setup changes, and `sky_hub` shipped one. User-facing consequence: the integration page describes UI setup instead of the YAML setup this PR deprecates.

**Maintainer Feedback:**

- 2026-11-03: No human maintainer feedback yet. On 2026-10-31 the `home-assistant` bot marked the PR "Changes requested" because I had not signed the CLA, and moved it to draft. The CLA is now signed (`cla-signed` label, `cla-bot` check passing). Copilot's automated reviewer posted six reviews between 2026-10-31 and 2026-11-03, but each one says "Copilot was unable to review this pull request because the user who requested the review has insufficient remaining quota," so there is no automated feedback to address.
- 2026-11-08: None yet from a human maintainer. Before marking the PR ready, I went through every changed file line by line so I can answer review questions, then ticked "Any generated code has been carefully reviewed". I left "The code change is tested and works locally" unticked because I have only tested without a router. Marking the PR ready cleared the CLA bot's old "Changes requested"; the PR now shows "Review required". The only new thread activity is another Copilot quota message, which needs no reply.

**Status:** Awaiting review (waiting on the maintainer). Ready for review since 2026-11-08, with upstream CI green (27 successful, 15 skipped, 0 failing on `b43c87b8df5`). Docs PR #50303 is open against `next` with its checks passing. Next: per my PR-monitor playbook, no nudge before 2026-11-18; answer any review in my own words and push requested changes as new commits (no rebasing or force-pushing).

**Toolkit**

- Tool: `ship-gate` validation hook and upstream GitHub Actions CI. Before pushing, I let `ship-gate` run the checks in `ship-manifest.md` (pytest with branch coverage, `ruff format`, `ruff check`) and compare `plan.md` with my PR draft; after pushing, I read the CI results on PR #189792 and fixed the failing `Check hassfest` and `bt_home_hub_5` test jobs.
- What it produced: Verified branch isolation, formatting, test execution, and manifest categories. Before the 2026-11-03 fixes, upstream CI showed 19 passed, 16 skipped, and 2 failed (`Check hassfest` and `Run tests Python 3.14.10 (bt_home_hub_5)`). After pushing the fixes and merging `upstream/dev`, it showed 22 successful, 15 skipped, and 0 failing.
- Difficulties: Ensuring all working files stayed out of git commits and aligning strict core `hassfest` manifest validation rules.
- What changed: Synchronized manifest categories, sorted translation keys, and added mandatory `device_info`.

---

## Learnings and Reflections

**Technical Skills Gained**
- A working understanding of how Home Assistant integrations are structured, including how `ConfigFlow`, `DataUpdateCoordinator`, and `ScannerEntity` split the work.
- Hands-on experience with Home Assistant's test tools: configuring `MockConfigEntry`, intercepting coordinator lifecycles, and managing test teardown assertions like `verify_cleanup` and `HASocketBlockedError`.
- Hands-on experience with modern Python 3.14 requirements, including schema definitions with `probatio` and strict enforcement against deprecated typing imports.

**Challenges Overcome**
- Navigating the lack of hardware by using mocked executor jobs and understanding third-party library boundaries.
- Resolving complex async test teardown failures where background tasks scheduled by `async_setup_scanner` leaked unmocked socket calls into adjacent test teardowns.
- Reconciling upstream toolchain constraints (`uv` vs `.venv/bin/python3`) in a fast-evolving Python 3.14 development environment.

**What I'd Do Differently Next Time**
- Inspect the repository's active development fixtures and translation compilers earlier in the process to anticipate translation key test requirements before debugging teardown failures.
- Before simplifying a step from my validated plan, I will read why the reference implementation does it. Three of my bugs (lost default host, misleading repair notice, clashing entity IDs) came from simplifying something `sky_hub` did for a reason. If I still deviate, write the deviation and what users will experience in the PR's deviation list at that moment, not at the end.
- Match my local toolchain to CI before the first commit: run `script/setup`, then check the Python version, the ruff version and that `python3 -m script.hassfest` runs. My 3.14.7 venv with ruff 0.17.1 and no `hassfest` let a stale `integrations.json` reach CI.
- Write tests from the failures a user would hit (missing `host`, unreachable router, the same device seen by two hubs, a failed unload) before looking at coverage. I had 100% statement and branch coverage and still shipped the lost default host and the clashing entity IDs, because those branches ran but no test checked the user-facing result.

---

## Resources Used

- [Home Assistant Contributor Documentation: Config Flow](https://contributors.home-assistant.io/docs/config_entries_config_flow_handler/)
- [Home Assistant Contributor Documentation: Entity Standards](https://contributors.home-assistant.io/docs/core/entity/)
- [Sky Hub ScannerEntity Migration PR #187258](https://github.com/home-assistant/core/pull/187258)
- [Home Assistant Core Epic #50326](https://github.com/home-assistant/core/issues/50326)
- [Home Assistant Contributor Documentation: Fetching Data](https://contributors.home-assistant.io/core/integrations/fetching-data/) (the `DataUpdateCoordinator` pattern)
- [Legacy Device Tracker Migration Guide PR #187793](https://github.com/home-assistant/core/pull/187793)
- [Home Assistant AI Policy](https://contributors.home-assistant.io/start/ai-policy/)
- [Home Assistant Documentation Style Guide](https://contributors.home-assistant.io/writing/reference/style/) (for the docs PR)
