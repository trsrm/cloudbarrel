# Architecture

This document defines the high-level CloudBarrel modules and the seams between portable backup behavior and host integrations.

## System shape

```text
              ┌──────────────────────────────┐
              │ Platform integration         │
              │ CLI · systemd · OMV · CasaOS │
              └──────────────┬───────────────┘
                             │ control/status
              ┌──────────────▼───────────────┐
              │ CloudBarrel core             │
              │ Jobs · Runs · policies       │
              │ supervision · notifications  │
              └────────┬───────────┬─────────┘
                       │           │
               transfer│           │state
              ┌────────▼───┐   ┌───▼─────────┐
              │ rclone     │   │ local store │
              │ process    │   │ candidate:  │
              │ adapter    │   │ SQLite      │
              └────────┬───┘   └─────────────┘
                       │
              ┌────────▼─────────────────────┐
              │ Consumer clouds              │
              └──────────────────────────────┘
```

SQLite is a candidate for a future daemon. The current prototype uses atomic state files. No storage decision is accepted until the Linux migration proves the required interface.

## Core module

The core owns the difficult behavior behind one interface:

- configure and schedule Jobs;
- start, stop, retry, and observe Runs;
- execute Preflight and staged workflows;
- supervise `rclone` and normalize its progress and errors;
- maintain Current, History, baselines, and retention;
- run Integrity and Restore checks;
- record durable state;
- publish notifications.

Callers should not need to understand provider commands, retry rules, filesystem layout, or log parsing.

The first interface may be a CLI plus durable state. An HTTP interface is justified when a real UI integration needs it.

## Transfer seam

`rclone` is a true external dependency. CloudBarrel owns orchestration and treats the process invocation as an adapter:

- production runs use the installed `rclone` binary;
- tests use controlled process results or a small fake adapter;
- source-specific flags remain inside the core implementation;
- raw `rclone` output is normalized before platform integrations consume it.

CloudBarrel does not add a second abstraction over each `rclone` command until tests or another transfer adapter require it.

## Platform seam

A Platform integration may:

- install and update CloudBarrel;
- supply configuration and secrets;
- start or stop Jobs through the core interface;
- display stored Run state and bounded logs;
- connect host-native notifications or dashboards.

It must not implement backup semantics.

## External application seam

CloudBarrel is complete on its own. External local applications may consume its interface, but they remain callers at the same seam as other integrations. They do not own backup semantics and cannot be required to read or restore stored data.

### Plain Linux

The reference adapter uses native filesystems, `systemd` services and timers, and normal package management. It defines supported behavior.

### OpenMediaVault

An OMV plugin may add dashboard widgets, Jobs, Runs, logs, and settings. OMV continues to own disks, mounts, SMART, SMB, users, and host administration.

### CasaOS

A CasaOS package supplies Compose and application metadata and opens the generic CloudBarrel UI. CasaOS is a distribution surface, not the backup control plane.

### Constrained routers

Router packaging may use shell scripts and the host scheduler. It may reduce concurrency, Integrity work, and UI features. Router-specific workarounds stay outside the core.

## Filesystem model

```text
storage/
└── backup/
    ├── google-drive/
    │   ├── current/
    │   └── history/<run-id>/
    ├── icloud-drive/
    │   ├── current/
    │   └── history/<run-id>/
    └── icloud-photos/
        ├── personal/{current,history}/
        ├── shared/{current,history}/
        └── browse/
            └── YYYY/MM/  — hard links from both Current copies
```

The Browse view has no `all/` level and does not separate Personal from Shared. It is a derived, rebuildable view, not an authoritative backup. It is reconciled only after a successful Photos Run and never links to History.

The operating system, unrelated media files, and downloads should live outside this namespace.

## Ownership outside CloudBarrel

The host platform owns:

- disk discovery, formatting, mounts, RAID, and SMART;
- users, groups, file sharing, and network configuration;
- Transmission, DLNA, Jellyfin, and other applications;
- operating-system updates and security policy.

CloudBarrel may check host conditions required for a Run, but it must not become a second host administration system.
