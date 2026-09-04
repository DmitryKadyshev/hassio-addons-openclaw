# Changelog

All notable changes to the OpenClaw Assistant Home Assistant Add-on will be documented in this file.

## [0.5.92] - 2026-09-04

### Changed
- Fix Docker BuildKit `InvalidDefaultArgInFrom` by providing a valid default Home Assistant Debian base image.
- Publish pre-built multi-architecture images to GHCR through the Gitflow pipeline.
- Align supported architectures with Node.js 24 binary availability: `amd64` and `aarch64`.

## [0.5.90] - 2026-09-02

### Added
- **Resource profiles** (`resource_profile`): the add-on now gives the gateway an explicit Node.js heap budget instead of letting it size against total host memory. `auto` (default) picks `low` / `balanced` / `high` from CPU architecture and RAM, and never writes to `openclaw.json`. Selecting `low` explicitly also applies conservative OpenClaw defaults (currently `browser.enabled: false`) for keys you have not set yourself. The resolved profile and heap limit are logged at startup and shown on the landing page.
- **Home Assistant health sensors** (`ha_health_sensors`, `ha_health_interval`): optionally publish `sensor.openclaw_gateway`, `sensor.openclaw_version`, `sensor.openclaw_gateway_memory`, `sensor.openclaw_disk_used` and `sensor.openclaw_certificate_expiry` so you can alert on gateway health, disk usage and certificate expiry from automations. Requires `homeassistant_token`. New `oc-health` helper (`show` / `once` / `loop`) for previewing and debugging.
- **Config snapshots and rollback** (`config_backup_keep`): `openclaw.json` is now snapshotted to `/config/.openclaw/backups` before the add-on's first configuration write of each start, so an unwanted change can be undone. New `oc-config` helper: `list`, `diff`, `restore`, `snapshot`. Restoring always backs up the config it replaces first. Identical configs are not re-snapshotted, and only the newest `config_backup_keep` (default 10) are kept.
- New `ha_base_url` option to point the health sensors at a Home Assistant on a non-default port or host (empty = auto-detect).
- `oc-health check`: diagnoses credentials and API connectivity in one command — which endpoint was chosen, whether a token is present, whether the Supervisor host resolves, and the result of a live probe.
- Supervisor watchdog on the ingress port, so Home Assistant restarts the add-on if the ingress proxy stops answering. Can be turned off with the Watchdog toggle on the add-on page.
- **Smaller Home Assistant backups**: the add-on now declares `backup_exclude`, so regenerable caches and tooling (`.linuxbrew`, `.node_global`, `.npm`, `.cache`, `__pycache__`, stale `*.jsonl.lock` files) are skipped when Home Assistant backs the add-on up. Excluded directories are pruned without being walked, so backups are both smaller and faster. All user state — `openclaw.json`, config snapshots, skills, agent sessions, the `clawd` workspace, keys, secrets and certificates — is still backed up. Note that a restore replaces `/config` wholesale, so excluded tooling must be reinstalled rather than restored.
