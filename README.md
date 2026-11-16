# Wicofe-Recovery

Recovery and repair environment for Wicofe OS.

## Scope

- Bootable recovery environment.
- Filesystem inspection and repair workflows.
- System diagnostics and log collection.
- Restore and rollback integration.
- Recovery documentation and test scenarios.

## Safety requirements

Recovery operations must distinguish read-only diagnostics from destructive repair. Require explicit confirmation for data-changing actions, preserve logs where possible, and validate backup/restore integrity. Recovery should default to read-only inspection and must not execute binaries or scripts from a damaged target installation.

## Boundaries

Wicofe-Installer owns installation; Wicofe-Update owns update orchestration and update-state transitions; ONWFS owns filesystem-specific consistency and repair rules; Wicofe-Storage owns storage discovery and operations. Recovery coordinates these components without redefining their formats or state machines.

## Status

Initial repository baseline. No recovery image, repair utility, backup engine, or restore implementation is claimed.

## License

Original project code targets MPL-2.0; external utilities and dependencies require license review.

## Documentation

See [docs/INDEX.md](docs/INDEX.md) for the [recovery environment architecture](docs/RECOVERY_ENVIRONMENT_ARCHITECTURE_0.1.md) and technical documentation. The [central repository documentation map](https://github.com/Waizusora/Wicofe-Docs/blob/main/REPOSITORY_DOCUMENTATION_MAP.md) identifies canonical ownership and cross-repository boundaries.
