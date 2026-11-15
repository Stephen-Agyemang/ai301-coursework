# Migrating BT Home Hub 5 to ScannerEntity in Home Assistant Core

## Overview
This project migrated the `bt_home_hub_5` integration in Home Assistant Core from the deprecated `DeviceScanner` platform to the modern `ScannerEntity` model backed by configuration entries and a data update coordinator. Faced with the deprecation of legacy device scanners under core epic #50326, I designed and implemented a complete configuration flow, integration entry point, background update coordinator, repair issue handlers, and a 29-test unit suite achieving 100% statement and branch coverage. I submitted the implementation in Pull Request #189792 (https://github.com/home-assistant/core/pull/189792) alongside documentation PR #50303; although CI was green (27 passed, 15 skipped, 0 failing), the PR was closed by a maintainer because physical router testing was required for final approval.

## What happened

1. **2026-10-26 to 2026-10-30: Initial Assessment and Scaffolding**:
   Through static inspection of the `home-assistant/core` `dev` branch, I verified that `homeassistant/components/bt_home_hub_5/device_tracker.py` subclassed the deprecated `DeviceScanner` and lacked both a `config_flow.py` and unit tests. I updated `manifest.json` to enable `"config_flow": true`, set `"integration_type": "hub"`, and declare the appropriate network presence categories.

2. **2026-10-30: Config Flow, Duplicate Prevention, and Coordinator Polling**:
   I implemented `config_flow.py` supporting both UI setup (`async_step_user`) and automatic legacy YAML import (`async_step_import`), preventing duplicate entries via `self._async_abort_entries_match({CONF_HOST: host})` rather than an IP-based unique ID. Because the underlying `bthomehub5-devicelist` library performs blocking HTTP requests, I wrapped polling inside `coordinator.py` using `hass.async_add_executor_job` and converted `RequestException` failures into `UpdateFailed`.

3. **2026-10-30: Patching Network Calls in Tests**:
   During initial test runs, unmocked socket calls leaked into teardown and triggered `HASocketBlockedError`. Below is an excerpt from `tests/components/bt_home_hub_5/test_device_tracker.py` showing how router responses were patched to verify dynamic entity registration without physical hardware:
   ```python
   with patch(
       "homeassistant.components.bt_home_hub_5.coordinator.get_devicelist",
       return_value=DEVICE_1,
   ):
       await hass.config_entries.async_setup(entry.entry_id)
       await hass.async_block_till_done()
   ```

4. **2026-11-01: The Default Host Bug**:
   When users configured the integration via legacy YAML without specifying an explicit `host`, setup raised a `KeyError: 'host'` and failed to create a config entry. This occurred because I had stripped `PLATFORM_SCHEMA` from `device_tracker.py` during modernization—unlike `sky_hub`, which retained its schema to provide a fallback default host. Writing the unit test first exposed the failure (`1 failed, 15 passed`), and adding an extended schema with a default host resolved it, bringing the suite to `16 passed`.
   ```python
   async def test_yaml_setup_uses_default_host(hass: HomeAssistant) -> None:
       """Test YAML configuration without a host imports the default address."""
       ...
       entries = hass.config_entries.async_entries(DOMAIN)
       assert len(entries) == 1
       assert entries[0].data[CONF_HOST] == DEFAULT_HOST
   ```
   ```diff
   +PLATFORM_SCHEMA = DEVICE_TRACKER_PLATFORM_SCHEMA.extend(
   +    {probatio.Optional(CONF_HOST, default=DEFAULT_HOST): cv.string}
   +)
   ```

5. **2026-11-01: The Failed YAML Import Repair**:
   My initial implementation registered a single unconditional `deprecated_yaml` repair issue that falsely assured users their YAML configuration had been successfully imported, even when the router was unreachable and no config entry was created. To fix this misleading feedback, I restored the conditional error handling pattern in `device_tracker.py`: an ERROR-level `yaml_import_cannot_connect` repair is dispatched only when connection fails, while `deprecated_yaml` is restricted to successful imports. Running pytest showed `3 failed, 15 passed` prior to the fix and `18 passed` afterward.

6. **2026-11-03: CI From Red to Green**:
   Upstream CI checks initially reported two failures: `Check hassfest` and `Run tests Python 3.14.10 (bt_home_hub_5)`. These were hidden locally because my development venv was running an older Python version (`3.14.7`) and outdated `ruff` (`0.17.1`) without local hassfest validation. Rebuilding the virtual environment using `script/setup`, regenerating the stale `integrations.json`, and cleanly merging upstream/dev to resolve a manifest conflict introduced by upstream PR #189910 restored all GitHub Actions checks to an `All checks were successful` status with zero failures.

7. **2026-11-06: Comparing Against `sky_hub`**:
   Performing a file-by-file audit against the merged `sky_hub` reference migration (#187258) revealed that lingering `known_devices.yaml` entries could conflict with the newly registered `ScannerEntity` instances. I addressed this by incorporating a `legacy_known_devices` repair notification in `__init__.py` to warn users, adopting unload helper patterns (`async_unload_platforms_or_raise`), and replacing the catch-all `except Exception` in the coordinator with `except IndexError`. These enhancements cleanly expanded test coverage and increased passing unit tests from `20 passed` to `29 passed`.

8. **2026-11-06: 100% Branch Coverage**:
   Running `pytest` with `--cov-branch --cov-report=term-missing` across all 29 tests confirmed complete statement and branch coverage across all five integration modules:
   ```text
   Name                                                       Stmts   Miss Branch BrPart  Cover   Missing
   ------------------------------------------------------------------------------------------------------
   homeassistant/components/bt_home_hub_5/__init__.py            34      0      4      0   100%
   homeassistant/components/bt_home_hub_5/config_flow.py         43      0      4      0   100%
   homeassistant/components/bt_home_hub_5/const.py                7      0      0      0   100%
   homeassistant/components/bt_home_hub_5/coordinator.py         26      0      2      0   100%
   homeassistant/components/bt_home_hub_5/device_tracker.py      61      0      8      0   100%
   ------------------------------------------------------------------------------------------------------
   TOTAL                                                        171      0     18      0   100%
   29 passed in 1.05s
   ```

9. **2026-11-08 to 2026-11-10: How It Ended**:
   After resolving upstream manifest merge conflicts and passing all 27 GitHub Actions CI checks that ran (`hassfest`, `ruff`, `mypy`, `pylint`, and `pytest` on Python 3.14.10), I marked PR #189792 ready for review and asked the community for real-hardware testing. Maintainer `joostlek` subsequently closed the pull request, noting that maintainers had concluded that hardware-backed migrations require physical device access by the author.
   Just 88 seconds later (16:02:57 UTC on 2026-11-10), the same maintainer codified this policy update directly on core epic #50326:
   > To work on one of these migrations, you need access to the device. We only accept migration pull requests from contributors who can test the change on their own hardware. Pull requests tested only with mocked responses will be closed.

## Limitations and what's next
While the integration passed all automated test suites and CI gates, it was limited by the lack of physical hardware testing during review. Hardware verification was merely recommended in the PR template ("The code change is tested and works locally") rather than enforced as a written requirement when I built this. Consequently, I left the testing checklist box unchecked on purpose and stated transparently in the pull request description:
> I don't own a BT Home Hub 5, so this is tested with mocked `get_devicelist` responses only.

When closing PR #189792, maintainer `joostlek` stated:

> "We have discussed this type of migration PRs before and we came to the conclusion that if the author doesn't have access to such device, we're going to close them.
>
> Thanks for being willing to contribute"

Because the pull request thread was locked, I posted this final update on issue #143022 to leave a clear record for other contributors:
> Update: my PR #189792 was closed because I don't have access to a BT Home Hub 5, which is now required for these migrations (see the note on #50326). Thanks to @joostlek for explaining. I'm stepping back from this issue so someone who owns the router can pick it up, and my branch stays on my fork if it helps. I'd still like to help with these migrations on a device I can test.

Moving forward, I would factor hardware dependency policies into issue selection earlier or focus on software-only core components where complete validation can be achieved in a sandboxed environment. The patterns learned here, such as building config flows, coordinators, and 100% branch-covered test suites, provide a strong foundation for future core contributions.
My concrete next step is to target software-only integrations, or devices I own or that are common in the US. Throughout this contribution, I adhered to Home Assistant project standards: complying with the six-month deprecation timeline for YAML configurations, following the official AI contributor guidelines, preserving the PR template structure without deleting mandatory sections, and aligning my architectural approach with approved core reference PRs like `sky_hub`.