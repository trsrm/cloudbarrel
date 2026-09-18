<p align="center">
  <img src="docs/assets/cloudbarrel-logo.png" alt="CloudBarrel" width="360">
</p>

# CloudBarrel

**Bring your cloud data home.**

CloudBarrel is a planned open-source, self-hosted backup tool for keeping independent, versioned local copies of data from consumer cloud services. It targets ordinary Linux machines and home servers, with optional integrations for OpenMediaVault, CasaOS, and constrained routers.

CloudBarrel is currently a documentation-first project. There is no installable release yet.

## Why it exists

Three failures that already happened, none of them unusual:

> My Threads account was locked by mistake. Support sent a data-export link that only works from inside the account I can no longer sign in to. The data is still there. I just cannot reach it.

> The Google Sheet where I track my finances silently reverted to a state from months earlier. No revision history, no explanation. Days of retyping.

> Several years of my childhood photos no longer exist anywhere. There was one copy, and one copy is all it takes.

All three have the same cause: one copy, held by someone else. Accounts get locked, sync bugs overwrite files, support does not reply. A few lost documents can be retyped. Several years of photos cannot.

Exporting by hand does not solve it. 700 GB of iCloud Photos is days of supervised downloads, so the task stays on the to-do list instead of getting done. CloudBarrel is set up once and then runs on its own.

Cloud storage is not an independent backup. An account can be locked, deleted, corrupted, or made unavailable. CloudBarrel is intended to copy data to storage that the user controls and to prove that the copy can be restored.

The reference setup includes:

- Google Drive
- iCloud Drive
- iCloud Photos, including Personal and Shared libraries

iCloud Photos also gets one combined `browse/YYYY/MM/` tree. It contains hard links to the current Personal and Shared files, so users can browse one timeline without duplicating media data.

The project will use proven transfer tools such as `rclone`. It will not implement another sync engine.

A Source becomes publicly supported only after its backup, failure, notification, Integrity, and Restore paths pass the reference deployment acceptance checks. Presence in the reference setup alone is not a support promise.

## Product shape

```text
Consumer clouds
      │
      ▼
CloudBarrel core
  jobs, runs, history, retention,
  integrity, restore checks, alerts
      │
      ▼
Local disk or NAS storage

Host integrations: plain Linux · OpenMediaVault · CasaOS · router-lite
```

The core must remain independent of any NAS dashboard. Platform integrations expose the same backup behavior through the host's normal controls.

This repository contains all of CloudBarrel, and it is complete on its own. External local applications may use the CloudBarrel interface, but they must not be required for backup correctness or for reading and restoring stored data.

## Status

The working reference implementation runs on an ASUS ZenWiFi XT9 with Entware, shell scripts, `rclone`, cron, Healthchecks.io, and an ext4 USB disk. It proved the backup semantics, but it also exposed the limits of running heavy backup work on a router.

The next practical milestone is to migrate the proven workflow to plain Debian or Raspberry Pi OS with native `systemd` supervision. Product code will follow only after that migration shows which interfaces are real.

## Documentation

- [Chat context](CHAT-CONTEXT.md) — self-contained project handoff for a new AI or human collaborator
- [Domain language](CONTEXT.md) — canonical project terms
- [Requirements](docs/requirements.md) — product scope and acceptance conditions
- [Architecture](docs/architecture.md) — core and integration seams
- [Reliability](docs/reliability.md) — failure model and safeguards
- [Backup and restore](docs/backup-and-restore.md) — data semantics and recovery expectations
- [Photos Browse view](docs/photos-browse.md) — combined date-based access to Personal and Shared photos
- [Current XT9 setup](docs/current-xt9-setup.md) — evidence from the working prototype
- [Migration to Raspberry Pi](docs/migration-to-pi.md) — safe transition plan
- [Roadmap](ROADMAP.md) — ordered milestones and stop conditions
- [Architecture decision](docs/adr/0001-portable-core.md) — why the core stays independent of host platforms
- [Product boundary decision](docs/adr/0002-open-source-product.md) — why CloudBarrel remains complete without external applications

## Platforms

| Platform | Intended support |
|---|---|
| [Debian, Ubuntu, Raspberry Pi OS](docs/platforms/plain-linux.md) | Reference deployment |
| [OpenMediaVault](docs/platforms/openmediavault.md) | Native NAS integration after the core is stable |
| [CasaOS and generic Docker](docs/platforms/casaos.md) | Accessible application packaging |
| [ASUSWRT, OpenWrt, and similar routers](docs/platforms/router.md) | Best-effort constrained deployment |

Media-server functions such as SMB, Transmission, DLNA, and Jellyfin belong to the host platform. CloudBarrel may document compatible deployment recipes, but they are not part of the backup core.

## Security

Never commit an `rclone.conf`, cloud session, Healthchecks URL, device export, or production state file. Local device snapshots belong under `references/private/`, which Git ignores.

## License

CloudBarrel is licensed under the [Mozilla Public License 2.0](LICENSE).
