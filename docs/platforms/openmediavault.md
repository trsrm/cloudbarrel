# OpenMediaVault

OpenMediaVault is the preferred future NAS integration because it already owns storage administration, SMART, SMB, users, logs, notifications, and system services.

## Integration shape

An OMV plugin may provide:

- a dashboard summary;
- Job and Run pages;
- start, stop, and retry actions;
- settings backed by the CloudBarrel core;
- host-native log links and notifications.

The plugin must call the portable core interface. It must not reproduce source commands, retention rules, or Integrity logic.

## Ownership

OMV owns disks, mounts, shares, users, permissions, and host updates. CloudBarrel owns cloud backup behavior. This prevents two control planes from rewriting the same system configuration.

## Timing

Build this integration after the plain Linux deployment and core interface are stable. Do not make OMV a prerequisite for the first supported release.
