# CasaOS

CasaOS is a future distribution integration for users who prefer a home-server application store and Docker-based installation.

## Integration shape

The package should contain Compose configuration and CasaOS metadata. It should open the generic CloudBarrel web interface rather than implement a second UI.

Persistent volumes must separate:

- configuration and secrets;
- Run state;
- logs;
- backup destinations.

## Limits

CasaOS launches and presents the application. It does not define backup semantics or replace the core supervisor. Direct access to host storage, permissions, and device health requires explicit documentation.

## Timing

Package for CasaOS after native Linux behavior and generic container storage permissions are proven.
