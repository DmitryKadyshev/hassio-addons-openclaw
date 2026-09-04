# Changelog

All notable changes to the OpenClaw Assistant Home Assistant Add-on will be documented in this file.

## [0.5.91] - 2026-09-04

### Added
- Add Gitflow-aware GitHub Actions for PR validation and multi-architecture container publishing.
- Publish `amd64`, `aarch64`/`arm64`, and `armv7` images to GitHub Container Registry and assemble a multi-arch manifest.
- Add `develop`, `release/*`, and `main` publishing flows with development, release-candidate, and versioned image tags.

### Changed
- Bump the Home Assistant add-on version to `0.5.91`.

## [0.5.90] - 2026-09-02

### Added
- **Resource profiles** (`resource_profile`): the add-on now gives the gateway an explicit Node.js heap budget instead of letting it size against total host memory. `auto` (default) picks `low` / `balanced` / `high` from CPU architecture and RAM, and never writes to `openclaw.json`. Selecting `low` explicitly also applies conservative OpenClaw defaults (currently `browser.enabled: false`) for keys you have not set yourself. The resolved profile and heap limit are logged at startup and shown on the landing page.
- **Home Assistant health sensors** (`ha_health_sensors`, `ha_health_interval`): optionally publish `sensor.openclaw_gateway`, `sensor.openclaw_version`, `sensor.openclaw_gateway_memory`, `sensor.openclaw_disk_used` and `sensor.openclaw_certificate_expiry` so you can alert on gateway health, disk usage and certificate expiry from automations. Requires `homeassistant_token`. New `oc-health` helper (`show` / `once` / `loop`) for previewing and debugging.
- **Config snapshots and rollback** (`config_backup_keep`): `openclaw.json` is now snapshotted to `/config/.openclaw/backups` before the add-on's first configuration write of each start, so an unwanted change can be undone. New `oc-config` helper: `list`, `diff`, `restore`, `snapshot`. Restoring always backs up the config it replaces first. Identical configs are not re-snapshotted, and only the newest `config_backup_keep` (default 10) are kept.
- New `ha_base_url` option to point the health sensors at a Home Assistant on a non-default port or host (empty = auto-detect).
- `oc-health check`: diagnoses credentials and API connectivity in one command.
- Supervisor watchdog on the ingress port, so Home Assistant restarts the add-on if the ingress proxy stops answering.

### Changed
- **Documentation reviewed against OpenClaw `2026.8.2`.** Added a *Device pairing (first connection)* walkthrough and corrected Control UI HTTPS/pairing guidance.
- Startup warnings about legacy `/config/.node_global` and `/config/.linuxbrew` directories now reflect their backup exclusion behavior.

### Fixed
- Removed the retired `gateway.controlUi.dangerouslyDisableDeviceAuth` configuration key and fixed `lan_https` proxy attribution for OpenClaw `2026.8.2`.
- Fixed Home Assistant health sensor endpoint selection, retry behavior, and failure logging.
- Fixed gateway restart-loop recovery and restart backoff after OpenClaw upgrades.

## [0.5.89] - 2026-09-02

### Changed
- Bump OpenClaw to `2026.8.2`.

## [0.5.88] - 2026-08-25

### Fixed
- Preserve explicit `false` values for boolean add-on options instead of replacing them with `true` defaults during startup.

## [0.5.87] - 2026-08-10

### Fixed
- Correct the bundled OpenClaw npm package version in the Docker image build for add-on `0.5.86`.

## [0.5.85] - 2026-07-21

### Changed
- Bump OpenClaw to `2026.7.1-2`.

## [0.5.84] - 2026-07-17

### Changed
- Bundle `mcporter@0.12.3` in the add-on image.

## [0.5.82] - 2026-07-15

### Fixed
- Repair add-on startup automatically when the bundled OpenClaw CLI is older than the persisted config format.
- Regenerate malformed `lan_https` CA/server certificates with proper X.509 extensions.

## [0.5.81] - 2026-07-14

### Changed
- Bump OpenClaw to `2026.7.1`.

## [0.5.80] - 2026-06-26

### Changed
- Bump OpenClaw to `2026.6.10`.

## [0.5.78] - 2026-06-16

### Changed
- Bump OpenClaw through the `2026.5.28` and `2026.6.6` upstream releases.

## [0.5.76] - 2026-05-29

### Changed
- Bump OpenClaw to `2026.5.27`.

## [0.5.75] - 2026-05-28

### Changed
- Add backup-friendly persistence defaults for optional toolchains.

## [0.5.74] - 2026-05-27

### Fixed
- Bundle `node-llama-cpp` and add `cmake` for source builds when needed.

## [0.5.73] - 2026-05-26

### Added
- Add the add-on-native `oc-gateway` helper for container-supervised runtime management.

## [0.5.72] - 2026-05-04

### Fixed
- Repair startup when a persisted OpenClaw config selects the unavailable Brave search provider.

## [0.5.71] - 2026-05-03

### Changed
- Bump OpenClaw through the `2026.4.29` and `2026.5.2` upstream releases.
