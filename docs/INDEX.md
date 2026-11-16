# Recovery Documentation Index

This directory documents Wicofe's recovery environment and system repair workflows.

## Scope
- Bootable recovery environment.
- System diagnostics and log collection.
- Restore, rollback, and recovery orchestration.
- Filesystem inspection/repair integration.
- Recovery scenarios and validation procedures.

## Ownership
[ONWFS](https://github.com/Waizusora/ONWFS) owns ONWFS consistency and filesystem-specific repair semantics. [Wicofe-Update](https://github.com/Waizusora/Wicofe-Update) owns update rollback/orchestration. [Wicofe-Installer](https://github.com/Waizusora/Wicofe-Installer) owns installation. This repository coordinates recovery and does not redefine component formats.

## Safety
Separate read-only diagnostics from destructive repair. Require explicit confirmation, preserve original data where possible, and verify backup/restore integrity.

- [Central documentation index](https://github.com/Waizusora/Wicofe-Docs/blob/main/INDEX.md)
- [Repository documentation map](https://github.com/Waizusora/Wicofe-Docs/blob/main/REPOSITORY_DOCUMENTATION_MAP.md)
