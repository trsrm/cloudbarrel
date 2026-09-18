# CloudBarrel project context

Use this document to continue the project in a new ChatGPT, Codex, or human working session. It summarizes durable decisions and verified lessons. Dated device state belongs in [docs/current-xt9-setup.md](docs/current-xt9-setup.md).

## Objective

Build an open-source, self-hosted backup platform that keeps independent local copies of consumer cloud data on ordinary disks and home servers. The initial sources are Google Drive, iCloud Drive, and both Personal and Shared iCloud Photos libraries.

The primary threat is loss of cloud access through account lockout, deletion, provider error, or user error. A backup must remain usable without access to the original cloud account.

The product promise is: CloudBarrel runs automatically and tells the user when a specific action is required. It must reduce attention spent on setup, authentication, diagnosis, retries, and restore without hiding uncertainty behind one generic success state.

## Product decision

CloudBarrel is a portable backup core with thin platform integrations.

- Plain Debian, Ubuntu, or Raspberry Pi OS is the reference deployment.
- OpenMediaVault may provide the deepest NAS UI integration.
- CasaOS and generic Docker may provide simple application distribution.
- Router environments are constrained, best-effort deployments.
- The core must not know how an OMV or CasaOS UI works.

This repository is all of CloudBarrel, and it is complete on its own. It includes backup correctness, History, Integrity, Restore, CLI control, machine-readable status, and platform integrations. External local applications may consume the CloudBarrel interface, but they must not be required to read or restore an archive.

The working name is **CloudBarrel**. The tagline is **Bring your cloud data home.**

## Proven backup semantics

Each Source has a `current/` copy and dated `history/` entries.

- New files enter the Current copy.
- Replaced or deleted files move to History before Current is changed.
- A cloud rename or move appears as an old path in History and a new path in Current.
- History stores versions observed by scheduled Runs, not every provider-side revision.
- Cleanup may delete old History. It must never delete the Current copy.

The XT9 prototype implements this with `rclone sync --backup-dir ... --delete-after` after strict preflight checks.

## Reliability requirements learned from the prototype

- Confirm the expected filesystems are mounted read-write and identify the correct volumes before any write.
- Perform a small write/read/delete probe before a backup pipeline.
- Abort when a cloud listing fails, returns zero unexpectedly, or drops below a safe fraction of the last successful count.
- Update baselines only after a fully successful Run.
- Use low transfer concurrency on constrained hardware.
- Store durable Run and Stage state outside process memory.
- Make long Seed runs restartable with durable completion markers.
- Supervise stalled or crashed work and limit consecutive automatic failures.
- Read progress from stored state and recent logs. Never rescan a large destination merely to display status.
- Keep a protected recent History window even under disk pressure.
- Verify backups against the cloud and perform a local Restore check.
- Send actionable failure notifications, including the failed Stage and recovery command when authentication expires.
- Keep Run result, source completeness, Integrity result, Restore result, and Action required status separate.
- Use external dead-man monitoring so a silent scheduler or dead device is detected.

## Source-specific lessons

### Google Drive

- Google-native documents must be exported into independent local formats.
- Dangling shortcuts should not fail the backup.
- A file-count baseline is a better deletion guard than total byte size because cloud-native objects can be sizeless.

### iCloud Drive

- Authentication sessions expire and require a simple, documented reconnect flow.
- Some Apple package files can download as different container bytes while containing equivalent internal files. Integrity verification needs a package-aware fallback rather than accepting every byte difference or ignoring all differences.
- Full download verification can be expensive. The prototype rotates checks across 12 shards.

### iCloud Photos

- Personal and Shared libraries are distinct Sources and remain separate authoritative copies.
- The first copy can run for days. It needs durable markers, retries, a watchdog, progress state, and final convergence checks.
- The required Browse view combines both Current copies directly under `browse/YYYY/MM/`. There is no `all/`, `personal/`, or `shared/` level inside Browse.
- Browse entries are hard links, not copies. They use no additional media blocks and must point only to files in the two Current copies.
- The Browse view is derived state. Reconcile it after each successful Photos Run, and allow it to be deleted and rebuilt without affecting the backup.
- The prototype smoke test proved the mechanism for a sampled file: the local timestamp selected the correct year and month, both paths had the same inode, and the link count became two.
- Photo indexing, thumbnails, and a custom gallery are deferred. Existing media software should be evaluated before building them.

## Host and media-server scope

The original prototype grew from a router backup node toward a Raspberry Pi home server. SMB, SMART monitoring, Transmission, DLNA, Jellyfin, storage administration, users, and permissions are valuable, but they belong to the operating system or NAS platform.

CloudBarrel owns cloud backup behavior. It may ship deployment guidance that works beside media services, but it should not become another NAS operating system.

## Current implementation evidence

A private local XT9 export was used as implementation evidence. It contains working scripts, configuration, state, and live credentials, and is stored outside this repository. It is not public source code.

The prototype established useful behavior but also exposed router limits: low memory, no swap, fragile userland tools, nonstandard startup ordering, and a router reboot associated with an expensive status command. The exact reboot mechanism was not proven. The status command was replaced with a lightweight state reader.

## Immediate direction

1. Preserve and understand the XT9 behavior.
2. Complete or safely stop the active Photos Seed run before moving the disk.
3. Recreate the behavior on plain Linux with native mounts and `systemd` supervision.
4. Add and verify the combined Photos Browse view after the Photos Current copies converge.
5. Verify current/history semantics, retention, integrity, alerts, and restore.
6. Only then decide whether the portable core should remain scripts or become a daemon. Go plus SQLite and an embedded web UI is a candidate, not an accepted decision.
7. Add OMV and CasaOS integrations only after the core interface is proven.
8. Publish versioned releases to a small group of known users and record every case that requires a terminal or maintainer help.

## Non-goals for the first release

- A new cloud transfer engine
- A NAS operating system
- A media library manager
- Full provider revision history
- Equal feature support on low-memory routers
- A custom web UI before CLI and run-state behavior are stable
