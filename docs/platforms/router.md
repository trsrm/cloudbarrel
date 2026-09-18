# Constrained routers

Routers such as ASUSWRT or OpenWrt devices are best-effort CloudBarrel deployments, not the reference platform.

## Expected limits

- low memory and CPU;
- old kernels and vendor userlands;
- unusual boot and mount ordering;
- limited process supervision;
- unreliable or missing SMART passthrough;
- greater risk that backup work affects network availability.

## Supported shape

A constrained deployment may use shell scripts, `rclone`, the host scheduler, atomic state files, low concurrency, and external dead-man monitoring.

It may omit:

- a web interface;
- expensive full Integrity checks;
- concurrent Jobs;
- local indexing and thumbnails;
- media-server workloads.

## Rule

Router workarounds remain in packaging or integration code. They must not weaken Preflight, credential safety, History protection, or failure visibility in the portable core.

The XT9 prototype is valuable as a constrained-device test bed, but its implementation is evidence to simplify, not a template to copy unchanged.
