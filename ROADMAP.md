# Roadmap

This roadmap orders work by risk for the CloudBarrel project. A phase ends when its stop condition is met; later phases must not be pulled forward for appearance.

## Phase 0 — preserve the prototype

- Keep the raw XT9 export under ignored local references.
- Record current behavior, known failures, and open questions.
- Remove secrets from every public example.

**Stop condition:** another contributor can explain the current system and its risks without reading the original chat.

## Phase 1 — plain Linux migration

- Move the existing USB disk to a Raspberry Pi 4 or another Debian host without recopying valid data.
- Replace Entware startup and ASUS cron with native mounts, services, and timers.
- Keep the proven transfer semantics unchanged during migration.
- Run dry runs before production writes.

**Stop condition:** Google Drive and iCloud Drive complete scheduled versioned Runs, alerts work, and the existing backup remains recoverable.

## Phase 2 — complete reliability loop

- Finish a restartable Personal and Shared iCloud Photos Seed run.
- Add normal recurring Photos Runs after the Seed run.
- Build and reconcile the combined `browse/YYYY/MM/` hard-link view from both Photos Current copies.
- Prove retention under age and low-space conditions.
- Prove cloud Integrity checks and local Restore checks.
- Document credential recovery and disaster recovery.
- Expose Run result, completeness, Integrity, Restore, and Action required as separate status dimensions.

**Stop condition:** each supported Source has a tested backup, failure, notification, integrity, and restore path.

## Phase 3 — extract the portable core

- Measure the remaining duplication and operational gaps in the Linux deployment.
- Define the smallest stable interface for Jobs, Runs, status, and control.
- Include structured Action required events and safe next actions in that interface.
- Choose the implementation only after the interface is proven. Go and SQLite are leading candidates, not commitments.
- Keep `rclone` as the transfer engine.

**Stop condition:** platform code can start, observe, and stop Runs without knowing source-specific orchestration details.

## Phase 4 — usable distribution

- Package the reference deployment for Debian-compatible systems.
- Add a generic web interface only if CLI and stored state are insufficient for normal operation.
- Publish secret-safe example configuration and migration guidance.
- Select an open-source license.
- Publish versioned releases to a small group of known users.
- Record each case that requires a terminal, manual repair, or maintainer help.

**Stop condition:** a new user can install, configure one Supported Source, receive an actionable failure alert, and restore a file from documented steps; repeated manual interventions are recorded for later design decisions.

## Phase 5 — platform integrations

1. OpenMediaVault dashboard and service pages
2. Docker and CasaOS application metadata
3. Constrained router packaging where the hardware can support it

**Stop condition:** every integration delegates backup behavior to the same core interface.

## Deferred

- Cockpit integration
- Thumbnails or photo gallery
- Turnkey Transmission, DLNA, or Jellyfin bundles
- Multi-disk replication and RAID guidance
- Provider connectors beyond the initial three
