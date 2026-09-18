# Backup and restore

This document defines what CloudBarrel preserves and what a user can expect during recovery.

## Versioned local copy

For each Source, CloudBarrel maintains:

```text
source-name/
├── current/
└── history/
    └── <run-id>/
```

Current is the latest successfully reconciled copy. During a Run, `rclone sync --backup-dir` moves files that would be replaced or deleted into that Run's History directory.

This model preserves:

- the latest downloaded file;
- older versions observed by CloudBarrel Runs;
- files deleted from the Source during the History retention period;
- old paths for renamed or moved files.

It does not preserve every change made between Runs or the provider's internal revision database.

## Source behavior

### Google Drive

Cloud-native documents must be exported to ordinary local formats. A restored export is independent of the Google account but may not preserve every provider-specific feature or revision.

### iCloud Drive

Ordinary files restore directly from the local filesystem. Apple package objects may need package-aware integrity comparison because equivalent downloads can have different container bytes.

### iCloud Photos

Personal and Shared libraries remain separate authoritative copies. A required derived Browse view combines their Current files directly under `browse/YYYY/MM/` using hard links. Users can find a photo by date without knowing which Apple library owns it.

Browse is a convenience path, not an additional backup copy. Restore remains possible from `personal/current`, `shared/current`, and their History even if Browse is missing or damaged.

## Routine file restore

The minimum restore path is direct filesystem access:

1. Stop or pause the modifying Job for the affected Destination.
2. Find the item in Current or a dated History directory.
3. Copy it to a separate recovery location.
4. Open or hash the recovered copy before using it.

A file browser or SMB share may expose the Photos Browse view. A future web UI may make discovery easier. None of them may be required to read the authoritative backup.

## Host failure

The backup disk must remain readable from another supported Linux system. Recovery must not depend on the original router, Raspberry Pi, OMV installation, database, or CloudBarrel UI.

The future reference deployment should provide a command that rebuilds operational state from configuration and existing destinations without rewriting backup data.

## Cloud account loss

If the cloud account is unavailable:

- stop cloud reconciliation;
- preserve Current and History as read-only evidence;
- make a second copy before reorganizing large recoveries where possible;
- restore directly from local storage.

CloudBarrel cannot refresh metadata or redownload missing objects after account loss. This is why Integrity and Restore checks must run while the Source is still available.

## Restore proof

Each scheduled Integrity cycle must select a representative local file, copy it through the supported recovery path to temporary storage, and compare the result. This proves local readability but does not replace periodic human restore exercises.
