# Migration to Raspberry Pi

This plan moves backup compute from the XT9 router to a Raspberry Pi 4 or another Debian host while preserving the existing disk and backup data.

## Target

Use Raspberry Pi OS Lite or another plain Debian-compatible system as the first reference deployment. Use a separate boot device for the operating system and attach the existing ext4 backup disk over USB 3.

The first migration should not add OpenMediaVault, Docker, a web UI, or a new daemon. It should replace only router-specific infrastructure:

| XT9 | Linux replacement |
|---|---|
| Entware | `apt` packages |
| bind-mounted `/opt` | normal filesystem paths |
| ASUS `cru` | `systemd` timers |
| shell PID supervision | `systemd` service state |
| custom boot hook | mount and service dependencies |
| firmware curl/wget workarounds | distribution TLS tools |

## Preconditions

- Confirm the final iCloud Photos Seed state. Do not assume the exported `RUNNING` state completed.
- Stop all modifying XT9 jobs.
- Save a fresh secret-safe manifest and a private configuration backup.
- Record filesystem UUIDs, labels, and role markers.
- Confirm that the new USB enclosure exposes stable storage and, where possible, SMART data.

## Transition

1. Stop the XT9 scheduler and active workers.
2. Confirm no `rclone` or backup script still writes to the disk.
3. Run `sync` and unmount both filesystems cleanly.
4. Boot the Pi from a separate microSD or SSD.
5. Attach the USB disk and mount it by UUID.
6. Mount existing data without changing directory names.
7. Install the required distribution packages and `rclone`.
8. Move credentials into root-readable configuration outside Git.
9. Adapt paths and replace cron with services and timers.
10. Run Preflight and read-only inspection.
11. Run each Source in dry-run mode.
12. Run one controlled production backup.
13. Run Integrity and Restore checks.
14. Build the combined Photos Browse view and verify that sample links share inodes with both Current copies.
15. Enable timers only after manual runs pass.

Existing data should not be downloaded again. `rclone` should compare the destination and transfer only missing or changed items. A dry run is required before production.

## Service ordering

Services must require the backup filesystem mount. A timer firing during boot must wait for the mount or fail visibly without modifying state.

Long Photos work should run as a supervised service with restart policy and durable Stage markers. Status should come from stored state and the journal, not a filesystem scan.

## Validation

The migration is complete only when:

- the host reboots with storage mounted at the expected paths;
- timers do not start before storage is ready;
- Google Drive and iCloud Drive dry runs show expected changes;
- one production Run completes and updates its baseline;
- Healthchecks receives success and a controlled test failure;
- Integrity and Restore checks pass;
- `browse/YYYY/MM/` contains current Personal and Shared Photos through hard links without an `all/` level;
- a file can be recovered from Current and one from History using another machine or recovery path.

## OpenMediaVault decision

OpenMediaVault can be added after the plain Linux workflow is stable. It is valuable for disks, SMART, SMB, users, logs, and a polished NAS control plane, but it must not be required for backup correctness.
