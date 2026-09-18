# Plain Linux

Plain Debian, Ubuntu, and Raspberry Pi OS form the CloudBarrel reference deployment.

## Responsibilities

- native package installation;
- filesystems mounted by UUID;
- `systemd` services and timers;
- protected configuration and credentials;
- local logs and durable Run state;
- optional SMB and SMART tools managed by the host.

This deployment defines supported backup behavior. Other integrations must produce the same Jobs, Runs, History, Integrity, and Restore results.

## Initial implementation

The first Linux migration should run the proven shell and `rclone` workflow with minimal path changes. A daemon is justified only after the migration reveals repeated orchestration that scripts and `systemd` cannot express safely.

## Media services

Transmission, ReadyMedia, Jellyfin, and Samba can run beside CloudBarrel. Keep downloads and media outside the backup namespace, and avoid competing heavy disk work during backup and Integrity windows.
