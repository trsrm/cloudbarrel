# Photos Browse view

The Photos Browse view gives users one date-based tree for the Current files from both Personal and Shared iCloud Photos libraries.

## Layout

```text
icloud-photos/
├── personal/
│   ├── current/
│   └── history/
├── shared/
│   ├── current/
│   └── history/
└── browse/
    ├── 2024/
    │   ├── 01/
    │   └── 02/
    ├── 2025/
    └── 2026/
```

There is no `browse/all/`, `browse/personal/`, or `browse/shared/`. Users see one timeline, like the combined view in Apple Photos.

## Storage behavior

Every Browse entry is a hard link to a file under `personal/current/` or `shared/current/`. Both names point to the same inode and data blocks, so the view does not duplicate media storage.

Browse requires the Current copies and Browse directory to share a filesystem with hard-link support. It never links to History.

## Date and filename rules

The lightweight first implementation derives `YYYY/MM` from the local timestamp preserved by the Photos transfer. If broader testing shows that this timestamp does not represent capture time for a file type, add a targeted EXIF or QuickTime metadata fallback.

Personal and Shared filenames may collide in the same month. The indexer must use deterministic alternate names and must never replace one different asset with another.

## Lifecycle

Reconcile Browse only after a successful Photos Run:

1. Read the two Current trees.
2. Calculate each `YYYY/MM` destination.
3. Create missing hard links.
4. Remove stale Browse links for files no longer in Current or moved between libraries.
5. Publish completion only after reconciliation succeeds.

Browse is derived state. It can be removed and rebuilt from the two Current copies. It must never become input to backup, Integrity, retention, or restore logic.

## Prototype evidence

The XT9 smoke test selected one real Photos asset, downloaded it with its timestamp, created the expected year/month link, and confirmed that canonical and Browse paths had the same inode and link count two. A separate read-only comparison reported no overlapping Personal and Shared paths at that time.

The production Browse indexer was not present in the exported XT9 stack. Implementing and verifying it remains required work.

## Deferred presentation

SMB or a local file browser is sufficient for the first release. Thumbnails, a static gallery, a web timeline, search, maps, and face recognition remain separate future work.
