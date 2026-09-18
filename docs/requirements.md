# Requirements

This document defines the initial CloudBarrel product requirements for contributors and reviewers.

## Goals

CloudBarrel must:

1. Create independent local copies of configured cloud Sources.
2. Preserve files deleted or replaced in the cloud for a configurable period.
3. Detect unsafe source, storage, scheduler, and authentication failures.
4. Prove that backup data can be read and restored.
5. Run on plain Linux without a NAS-specific control plane.
6. Expose the same behavior through optional platform integrations.

## Open-source guarantees

- This repository contains the complete open-source CloudBarrel product.
- Backup correctness, History, Integrity, Restore, and access to stored files must not depend on an external application.
- External local applications may use the CloudBarrel interface.
- CloudBarrel must remain operable on infrastructure controlled by the user and must not require a vendor cloud service.
- Unavailable external software must never block direct access to backup data.

## Reference sources

- Google Drive
- iCloud Drive
- iCloud Photos Personal Library
- iCloud Photos Shared Library

Each Source is configured independently. A failure in one Source must be visible and must not silently mark another Source as successful.

Presence in the reference setup does not make a Source publicly supported. A Source becomes a Supported Source only after its backup, failure, notification, Integrity, and Restore paths pass the reference deployment acceptance checks.

## Job and Run behavior

A Job must define a Source, Destination, schedule, Retention policy, Integrity policy, and notification policy.

Each Run must record:

- immutable Run ID;
- state and current Stage;
- start, update, and finish times;
- transferred files and bytes when available without a new scan;
- result and normalized error;
- path to bounded diagnostic output.

Minimum Run states are `queued`, `running`, `succeeded`, `failed`, and `cancelled`. A restartable Seed run may also expose `retry_wait`.

Only one modifying Run may own a Destination at a time.

A failed Run may put its Job into `action_required` when automatic retry is unsafe or cannot succeed. That status must identify one specific reason and one safe next action. It is not a successful Run state.

Status must report these dimensions separately:

- latest Run result;
- latest known Source completeness;
- latest Integrity result;
- latest Restore result;
- current Action required, if any.

No combined green status may imply that all five are current when one is unknown, stale, or failed.

## Backup behavior

- The Current copy represents the latest successful reconciliation.
- Replaced and deleted local files move to dated History before removal from Current.
- A failed transfer must not be recorded as a successful baseline.
- CloudBarrel must not assume that all Sources provide stable sizes, hashes, or revision histories.
- Source-specific equivalence rules must be explicit and testable.

## Preflight

Before a modifying Run, CloudBarrel must verify:

- the expected Destination filesystem is mounted read-write;
- the destination identity matches configuration;
- a small write/read/delete probe succeeds when writes are expected;
- required tools and credentials are present;
- the Source listing succeeds;
- the listing is not unexpectedly empty or anomalously smaller than the last successful baseline;
- free space is above the configured safety floor.

Preflight failure must leave Current and History unchanged.

## Retention

- Age-based retention applies only to History.
- Storage-pressure cleanup removes the oldest eligible History until the target free-space level is reached.
- A configurable recent period remains protected from automatic cleanup.
- Current is never a retention candidate.
- Cleanup validates every deletion path before removing data.

## Photos Browse view

- Personal and Shared Photos remain separate authoritative Sources.
- One combined Browse view exposes both Current copies as `browse/YYYY/MM/`.
- Browse must not contain an extra `all/`, `personal/`, or `shared/` directory level.
- Every Browse entry is a hard link to a file in a Current copy. It must not duplicate media blocks.
- Browse includes Current files only. It does not expose History.
- Reconciliation runs only after a successful Photos Run and removes stale links for deleted files or moves between Personal and Shared.
- Filename collisions must be preserved with deterministic alternate names; the indexer must never overwrite an existing different file.
- Browse is derived state and can be deleted and rebuilt from the two Current copies.
- The Destination filesystem must support hard links for this feature.

## Integrity and restore

- Integrity checks compare Source content with Current under source-specific rules.
- Large checks may rotate through deterministic shards.
- Missing files and read errors always fail the check.
- Provider-specific representation differences may pass only after a stricter semantic equivalence check.
- Each integrity cycle includes a local Restore check through a documented recovery path.

## Monitoring

- Scheduled work sends start, success, and failure signals where useful.
- External dead-man monitoring detects a device or scheduler that stops reporting.
- Failure notifications identify the Job, Run, Stage, result, and safe next action.
- Authentication expiry or failure creates Action required with the reconnect procedure.
- Status reads stored state and recent bounded logs; it must not traverse the backup tree.

## Security

- Credentials never enter Git, normal logs, failure notifications, or public diagnostics.
- Platform integrations use least privilege.
- The reference deployment must document file ownership and permissions.
- Public examples use placeholders and synthetic identifiers.
- Authentication and recovery stay local to the user's infrastructure.

## Resource behavior

- Transfer concurrency is configurable per deployment.
- A constrained deployment may disable expensive Integrity checks and web UI.
- Backup work must not endanger the primary network router.
- Status and monitoring operations remain cheap while a large Run is active.

## Non-goals

- Implementing cloud transfer protocols already supported by `rclone`
- Managing disks, RAID, SMB users, torrents, or media libraries
- Capturing changes that occur entirely between scheduled Runs
- Reproducing Google or Apple provider-side revision databases
- Promising identical support across Linux servers and routers
