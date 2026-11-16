# Wicofe Recovery Architecture 0.1

**Status:** Draft architecture. This document defines recovery responsibilities and safety requirements. It does not claim that a recovery image, repair utility, backup engine, or rollback implementation exists.

## 1. Purpose

Wicofe-Recovery provides a controlled environment for diagnosing, repairing, restoring and recovering Wicofe systems that cannot boot normally or whose storage/system state is damaged.

Recovery must operate independently of the damaged installation where possible, default to non-destructive inspection, and make the consequences of every mutation explicit.

## 2. Ownership boundaries

| Repository | Canonical responsibility |
|---|---|
| Wicofe-Recovery | Recovery environment, recovery workflow, diagnostics collection, restore orchestration and recovery result reporting |
| Wicofe-Update | Whole-system update state, activation protocol and update rollback decision |
| Wicofe-Installer | Initial installation and deployment |
| Wicofe-Storage | Device discovery, partition inspection and privileged storage operations |
| ONWFS | ONWFS format validation, consistency rules and format-specific repair tools |
| Wicofe-Kernel | Recovery kernel/runtime support and device interfaces |
| Wicofe-Image | Recovery image composition and release artifact metadata |
| WPM | Package database and individual package transaction recovery |
| Osynit | Normal-system service management; recovery startup must not depend on a healthy installed Osynit instance |
| Wicofe-Tests | Recovery, corruption, interruption, restore and boot-failure test suites |

Recovery coordinates the authoritative subsystem. It must not implement a competing filesystem checker, redefine update state, or silently overwrite the package database.

## 3. Recovery operating modes

Potential modes:
- **Standalone recovery boot:** boot a separate recovery image without mounting the installed root read-write.
- **Read-only diagnosis:** inspect devices, partitions, filesystems, boot metadata, logs and update state.
- **Filesystem maintenance:** invoke the canonical filesystem checker/repair tool with explicit mode and target.
- **System restore:** restore a verified backup or known-good system artifact.
- **Update recovery:** ask Wicofe-Update to inspect incomplete activation and execute its documented recovery protocol.
- **Package transaction recovery:** inspect and recover an interrupted WPM transaction using WPM-owned metadata and tooling.

Each mode requires implementation and test evidence before being advertised. Network access must be optional for core offline recovery tasks.

## 4. Trust boundary and environment isolation

The recovery environment must not trust binaries, libraries, service state, configuration or logs from a damaged installation merely because they are present on disk.

Boot recovery from a separately verified image or other explicitly trusted source. Verify recovery artifact integrity and authenticity before use. Keep recovery tools and their runtime dependencies isolated from the target root.

Mount the target read-only by default. Do not execute target-installed scripts, service hooks, package maintainer scripts or binaries during diagnosis. If a repair requires target metadata or code, parse it as untrusted input within bounded, least-privilege tooling.

## 5. Discovery and assessment

The initial workflow should:
1. Enumerate block devices and stable identities through Wicofe-Storage.
2. Inspect partition tables without modifying them.
3. Probe filesystem signatures and supported feature flags.
4. Identify candidate Wicofe installations and boot artifacts.
5. Read update/activation state without assuming it is internally consistent.
6. Collect kernel, boot and storage diagnostics.
7. Produce a structured assessment with evidence, uncertainty and recommended next actions.

Unknown partition layouts, unsupported filesystem features, ambiguous system roots and changed device identities must fail closed. A failed probe is not proof that a device is blank.

## 6. Repair policy

Separate operations into explicit risk classes:
- **Read-only:** inventory, metadata inspection, log collection and checksum verification.
- **Reversible preparation:** creating a backup copy or snapshot where supported and verified.
- **Mutating repair:** filesystem metadata repair, boot configuration changes, package database repair or system file restoration.
- **Destructive recovery:** repartitioning, formatting, wiping or restoring over existing data.

Every mutating operation requires a plan describing exact target identity, affected ranges/files, expected changes, backup state, interruption behavior and verification procedure. Destructive operations require explicit confirmation tied to the exact plan.

Never automatically repair a filesystem because it failed to mount. Preserve an original image or metadata backup where feasible, but do not claim backup safety until integrity and restore tests pass.

## 7. Filesystem repair integration

Filesystem-specific repair belongs to the filesystem owner:
- ONWFS owns ONWFS parser, checker, recovery semantics and repair tooling.
- Other filesystem formats require their own authoritative tools and reviewed licenses.
- Wicofe-Recovery supplies orchestration, target safety, progress, result normalization and recovery UX.

ONWFS 0.2.x is a draft format proposal. Until its on-disk format and tools are implemented and validated, recovery must not claim ONWFS repair capability.

A filesystem tool result must distinguish clean, errors-found, repaired, unrepairable, unsupported-feature, interrupted and I/O-failure states. A nonzero exit code alone is not a sufficient diagnosis; preserve structured tool output and version identity.

## 8. Update and boot recovery

Wicofe-Update owns the update state machine and activation semantics. Recovery should:
- identify the current and candidate release;
- inspect durable update intent and completion records;
- verify referenced artifacts and compatibility;
- ask the updater for a supported resume, rollback or manual-recovery action;
- record which system state is active and which state is retained;
- verify boot configuration and required artifacts after the operation.

Do not independently switch A/B slots, rewrite boot control state or roll back a release unless a published contract explicitly assigns that action to the recovery component. The update model is not yet selected; no atomic rollback guarantee is implied.

## 9. Backup and restore

A backup manifest should identify:
- backup schema/version and creation time;
- source system/release and architecture;
- included paths, volumes and metadata classes;
- excluded paths and volatile data;
- file/data hashes and integrity tree;
- encryption and key-recovery requirements, if supported;
- tool versions and provenance;
- restore compatibility and required free space.

Backup completion must not be reported until all required data and manifest metadata are verified. Restore must validate the backup, target identity, compatibility, capacity and overwrite scope before writing.

System rollback and user-data restore are separate operations. Restoring old system binaries does not necessarily make newer user data or application databases compatible. Define migration and restore policy for persistent data explicitly.

## 10. Diagnostics and privacy

Provide a diagnostic bundle with:
- recovery image and tool versions;
- architecture and boot mode;
- device/partition inventory with privacy-aware identifiers;
- filesystem probe/check results;
- boot and update state summaries;
- relevant logs and operation IDs;
- timestamps, error stages and verification outcomes.

Exclude passwords, tokens, private keys, encryption secrets, personal file contents and unnecessary hardware identifiers. Review diagnostic bundles before export or network upload. Network transfer must be explicit and authenticated.

## 11. Power-loss and interruption safety

Every mutation must define durable checkpoints and restart behavior. On recovery restart, inspect actual device state rather than trusting a progress marker alone.

Test interruption:
- during filesystem checking and repair;
- while writing boot configuration;
- during backup creation and verification;
- during restore at every write boundary;
- during package transaction recovery;
- during updater resume/rollback handoff.

Report complete, failed-before-write, partially modified and indeterminate outcomes distinctly. Do not automatically repeat a destructive action when its previous completion state is uncertain.

## 12. Security model

- Use least privilege and narrow privileged interfaces.
- Revalidate stable device identity before each destructive workflow.
- Bound parser memory, recursion, archive size and decompression.
- Reject path traversal, unsafe symlinks and malformed metadata.
- Do not execute untrusted target-root code.
- Verify recovery image and restore artifact authenticity.
- Keep secrets out of logs and diagnostic bundles.
- Require authorization for destructive repair, restore and downgrade.
- Record operation plans, confirmations and results for audit.

## 13. Testing strategy

- Boot recovery without a valid installed root.
- Recovery image integrity and signature failure.
- Malformed partition tables and filesystem metadata.
- Unsupported and unknown filesystem feature handling.
- Read-only mount enforcement.
- Device identity replacement between plan and execution.
- Corrupt, truncated and incompatible backup rejection.
- Backup/restore integrity and metadata preservation.
- Power loss at each repair/restore checkpoint.
- Interrupted update-state inspection and updater handoff.
- Package transaction recovery and database consistency.
- Secret redaction and diagnostic privacy.
- x86-64 and AArch64 boot tests for every claimed profile.

All destructive tests must use disposable images or dedicated test hardware.

## 14. Open decisions

- Recovery boot protocol and image profile.
- Trusted recovery signing roots and update mechanism.
- Minimal recovery userspace and service model.
- Stable device identity contract.
- Filesystem checker/repair command interfaces.
- Backup format, encryption and key recovery.
- Supported restore granularity and data migration.
- Update rollback ownership and boot-control interface.
- Diagnostic retention/export policy.
- Offline networking and remote recovery scope.

## 15. Completion criteria

A production recovery environment requires a separately verifiable boot artifact, read-only-by-default discovery, authoritative filesystem tooling, explicit destructive plans, tested backup/restore integrity, update handoff contracts, privacy-safe diagnostics and fault-injection evidence. Until these exist, this remains a draft architecture.
