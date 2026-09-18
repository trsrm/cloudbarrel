# Reliability model

This document describes the failures CloudBarrel must expect and the safeguards that make those failures visible and recoverable.

## Failure principles

1. A silent failure is worse than a visible failed Run.
2. A successful transfer is not proof of a usable backup.
3. Read-only checks must finish before destructive reconciliation begins.
4. Stored state must survive a process crash or host reboot.
5. Monitoring must be cheaper than the work it observes.

## Failure matrix

| Failure | Required response |
|---|---|
| Destination missing or mounted incorrectly | Fail Preflight before any write |
| Destination becomes read-only | Fail the active Stage and alert |
| Cloud authentication expires | Preserve data, fail visibly, include reconnect steps |
| Cloud listing fails | Abort before reconciliation |
| Cloud listing unexpectedly collapses | Abort and require review or baseline update |
| Transfer process exits | Record Stage failure and bounded diagnostics |
| Long Run stalls | Supervisor stops the child safely and retries within policy |
| Host reboots | Resume a Seed run from durable markers; never infer success from stale PID state |
| Disk space becomes low | Prune eligible History, preserve protected History and Current |
| Scheduler stops | External dead-man check alerts |
| Backup content differs | Fail Integrity unless an explicit source-specific equivalence rule passes |
| Backup cannot be read locally | Fail Restore check and alert |

## Baselines

CloudBarrel may use the last successful remote file count as an anomaly baseline.

- Create or update the baseline only after a fully successful Run.
- Treat zero as unsafe unless the Source is explicitly expected to be empty.
- Make the permitted drop configurable.
- A baseline is a guardrail, not proof of integrity.

The XT9 prototype uses a 50% floor for Google Drive and iCloud Drive. This is a safe prototype default, not a universal product default.

## Retention under pressure

The prototype keeps History for 180 days, protects the newest 30 days, begins emergency cleanup below 15% free space, and aims for 25% free space. Product defaults require validation on larger disks.

Cleanup must:

- validate that every candidate is inside a configured History directory;
- ignore unknown directory names;
- support dry run;
- remove oldest eligible Runs first;
- stop if the target free-space level cannot be reached safely.

## Run state

Write state atomically. A PID is evidence only while the process is alive. After reboot, stale state must become a failed or retryable Run, never a successful Run.

Seed runs need durable Stage completion markers so completed large transfers are not repeated after a restart.

## Status cost

The XT9 prototype rebooted shortly after an old status script launched full `rclone size` traversals during an active Photos Seed run. Logs did not prove the exact reset mechanism, but the operation was unnecessary and unsafe on a 512 MB router.

Status must read:

- stored Run and Stage state;
- process liveness;
- current filesystem free space;
- a bounded tail of the active log.

Status must not launch a cloud listing, hash pass, or recursive destination scan.

## Verification layers

1. Transfer process succeeds.
2. Run state and baseline commit atomically.
3. Periodic Integrity check compares Source and Current.
4. Restore check copies and compares local data.
5. Human recovery instructions explain how to use the backup without the original host.

No single layer replaces the others.

## Photos Browse safety

The Browse view is rebuilt or reconciled only after a successful Photos Run. A failed backup must leave the previous Browse view intact or mark it stale; it must not publish a partially reconciled timeline as current.

The indexer must verify that each destination is inside `browse/YYYY/MM/`, use hard links on the same filesystem, preserve filename collisions, and never modify Personal or Shared Current files. Deleting Browse must not delete media content because the authoritative Current links remain.
