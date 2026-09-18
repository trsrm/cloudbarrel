# Current XT9 prototype

This document records the exported ASUS ZenWiFi XT9 backup system as observed during the CloudBarrel prototype. It is historical evidence, not a supported installation guide.

## Hardware and runtime

| Item | Exported state |
|---|---|
| Host | ASUS ZenWiFi XT9 on stock ASUSWRT |
| Kernel | Linux 4.19.183, ARMv7 |
| Runtime packages | Entware mounted at `/opt` |
| Transfer tool | `rclone` 1.75.1 static ARM build |
| Storage | One USB disk with separate system and backup filesystems |
| System filesystem | Small ext4 partition mounted as `SYSTEM` |
| Backup filesystem | ext4 partition mounted as `BACKUP` |

The original investigation also observed about 512 MB RAM and no swap. This resource limit strongly affects concurrency and monitoring cost.

## Disk layout

```text
USB disk
├── SYSTEM  — Entware, scripts, configuration, state, logs
└── BACKUP  — current copies and versioned History
```

Both filesystems contain role markers. The storage probe verifies mount path, ext4, read-write state, role identity, distinct devices, and a write/read/delete cycle.

## Backup workflow

The nightly pipeline has six stages:

1. Storage Preflight
2. Retention and log cleanup
3. Storage write probe
4. Google Drive versioned backup
5. iCloud Drive versioned backup
6. Post-backup free-space check

Google Drive and iCloud Drive use `current/` plus dated `history/`. Both use a last-success file-count baseline and abort if the new count falls below 50% of that baseline.

Both Sources had non-zero file-count baselines from successful Runs. The latest exported scheduled failure reported that the `SYSTEM` partition was not mounted after a reboot.

The last failure shows a startup-order risk: scheduled jobs can run before USB storage is ready after a reboot.

## Integrity

Weekly integrity covers Google Drive and iCloud Drive.

- Google Drive uses `rclone check` with dangling shortcuts skipped.
- iCloud Drive uses download comparison, one of 12 rotating hash shards, and package-aware verification for equivalent Apple package files.
- Each Source also performs a local Restore check on a file between 1 KiB and 10 MiB.

The exported state records a successful Integrity run and preserves which iCloud shard should run next.

## iCloud Photos Seed run

The Photos workflow is separate from the nightly pipeline. It discovers the Shared library, then runs:

1. Personal initial copy
2. Shared initial copy
3. Personal catch-up copy
4. Shared catch-up copy
5. Up to three final sync and check convergence passes

Durable markers prevent completed Stages from repeating. A supervisor checks process liveness, detects log inactivity after 30 minutes, restarts failed work, and sends Healthchecks events.

The exported state showed a running `shared-initial` Stage, no consecutive failures, and a durable marker for the completed Personal initial copy. The library size required a multi-day transfer.

This is only the state at export time. It does not prove that the Seed run later completed.

### Planned Photos Browse view

The agreed target layout also includes:

```text
icloud-photos/
├── personal/{current,history}/
├── shared/{current,history}/
└── browse/YYYY/MM/
```

Browse combines both Current copies through hard links and has no `all/` level. A prototype smoke test copied one Photos asset, preserved its date, placed its Browse link in the matching year and month, and confirmed the same inode with link count two. A read-only comparison of Personal and Shared paths reported zero overlap at that time.

The exported stack does not contain the production Browse indexer. The feature was designed and its core filesystem mechanism was verified, but it was not yet integrated after the Seed workflow.

## Scheduler

The exported cron state contains:

| Schedule | Work |
|---|---|
| Regular | Appliance heartbeat |
| Nightly | Versioned backup |
| Weekly | Integrity and Restore checks |
| Daily | iCloud authentication monitor |
| Frequent | Photos resource monitor |
| Frequent | Photos Seed supervisor |

The exported Entware startup script recreates most jobs but does not recreate the separate every-minute Photos resource monitor entry. This is configuration drift to resolve during migration, not behavior to copy.

## Monitoring

Healthchecks.io endpoints cover:

- appliance heartbeat;
- nightly backup;
- weekly integrity;
- iCloud authentication;
- iCloud Photos Seed run.

The heartbeat verifies mounts, role markers, `rclone`, configuration, kernel storage errors, and free space. Monitoring URLs and cloud credentials in the raw export are live secrets.

## Known risks

- The router performs both network and storage-compute roles.
- The userland showed repeated `grep` crashes during investigation.
- A software reset occurred during a heavy Photos operation. No OOM or kernel panic was proven.
- The former status script launched expensive recursive scans. It was replaced with a lightweight state and log reader.
- ASUS startup and USB mount ordering are fragile compared with native Linux mounts and `systemd` dependencies.
- SMART was not usable through the temporary USB device or bridge and remains deferred for proper backup hardware.
- The raw export includes live credentials and must never enter Git.

## Evidence location

The exact device export contains live credentials and private state. It is stored outside this repository and is not distributed with CloudBarrel.
