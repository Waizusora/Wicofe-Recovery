# Recovery Documentation Index

This directory documents Wicofe's recovery environment and system repair workflows.

## Architecture
- [Recovery environment architecture 0.1](RECOVERY_ENVIRONMENT_ARCHITECTURE_0.1.md) — draft design for isolated recovery boot, read-only diagnosis, repair policy, backup/restore, update handoff and interruption safety.

## Scope
- Bootable recovery environment.
- System diagnostics and log collection.
- Restore, rollback, and recovery orchestration.
- Filesystem inspection/repair integration.
- Recovery scenarios and validation procedures.

## Ownership
[ONWFS](https://github.com/Waizusora/ONWFS) owns ONWFS consistency and filesystem-specific repair semantics. [Wicofe-Update](https://github.com/Waizusora/Wicofe-Update) owns update state and rollback protocol. [Wicofe-Installer](https://github.com/Waizusora/Wicofe-Installer) owns initial installation. [Wicofe-Storage](https://github.com/Waizusora/Wicofe-Storage) owns storage discovery and operations. This repository coordinates recovery and does not redefine component formats or state machines.

## Safety
Default to read-only diagnostics. Require explicit confirmation for mutations, preserve data where possible, verify backup/restore integrity, and report partial or indeterminate outcomes honestly. ONWFS 0.2.x is still a draft; no ONWFS repair capability is implied.

- [Central documentation index](https://github.com/Waizusora/Wicofe-Docs/blob/main/INDEX.md)
- [Repository documentation map](https://github.com/Waizusora/Wicofe-Docs/blob/main/REPOSITORY_DOCUMENTATION_MAP.md)
