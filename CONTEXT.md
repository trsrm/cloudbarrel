# CloudBarrel domain

CloudBarrel keeps independent, versioned local backups of data held by consumer cloud services. This glossary defines the terms used across product, code, and documentation.

## Language

**Source**:
A configured cloud data set that CloudBarrel reads, such as one Google Drive or one iCloud Photos library.
_Avoid_: Remote, provider, account

**Supported Source**:
A Source whose backup, failure, notification, Integrity, and Restore paths pass the reference deployment acceptance checks.
_Avoid_: Available Source, built-in Source

**Destination**:
The local filesystem location owned by a Job.
_Avoid_: Target, backup folder

**Job**:
A durable backup definition that connects one Source to one Destination and defines schedule, history, integrity, and notification policies.
_Avoid_: Task, cron job, script

**Run**:
One execution of a Job, with durable state, stages, timestamps, progress, and a result.
_Avoid_: Attempt, process, sync

**Action required**:
A visible condition in which automation cannot continue safely until the user completes one specific recovery action.
_Avoid_: Generic error, warning

**Stage**:
A named step within a Run, such as preflight, transfer, verification, or retention.
_Avoid_: Phase, status

**Current copy**:
The latest successfully reconciled local representation of a Source.
_Avoid_: Mirror, live backup, current snapshot

**History**:
Older local file versions and deleted files moved out of the Current copy during a Run.
_Avoid_: Archive, snapshots, trash

**Retention policy**:
The rules that remove History by age or storage pressure while preserving a protected recent period.
_Avoid_: Cleanup policy, rotation

**Preflight**:
The read-only checks that must pass before a Run may change the Current copy or History.
_Avoid_: Health check, validation

**Integrity check**:
A comparison that tests whether Source content and the Current copy are equivalent under source-specific rules.
_Avoid_: Sync check, verification

**Restore check**:
A test that reads backup data through a recovery path and proves that at least one representative item can be recovered intact.
_Avoid_: Smoke test, copy test

**Seed run**:
The first potentially long Run that creates a complete local copy of a Source and can resume after interruption.
_Avoid_: Initial sync, download

**Browse view**:
A rebuildable `browse/YYYY/MM/` tree of hard links to the Current copies of Personal and Shared iCloud Photos libraries.
_Avoid_: Gallery, photo backup, All folder

**Platform integration**:
Packaging or UI that connects CloudBarrel to a host environment without owning backup behavior.
_Avoid_: Core plugin, platform backend

**Reference deployment**:
The plain Linux installation used to define and verify supported CloudBarrel behavior.
_Avoid_: Default platform, primary UI

**Constrained deployment**:
A best-effort installation on hardware where memory, CPU, operating system, or storage tooling limits supported behavior.
_Avoid_: Router edition, lite core
