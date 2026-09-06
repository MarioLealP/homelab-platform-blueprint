# ADR 0005 · Backups the node cannot destroy

**Status:** accepted

## Context

Ransomware and "rm on the wrong host" both have the same signature: the
production host, using its own credentials, deletes or encrypts the backups
along with everything else. If the node can write freely to its backup
destination, the backups are only as safe as the node.

## Decision

The backup destination is **append-only from the node's perspective**. The
node's credential can create new snapshots and read its own, but cannot delete
or overwrite existing ones. Pruning old snapshots is done by a separate identity
(a scheduled job at the destination, or the operator), not by the node.

## Consequences

**Good**

- A fully compromised node cannot retroactively destroy backup history.
- "Delete everything" scripts, buggy retention config, and malware all hit the
  same wall.

**Bad / accepted**

- Retention/pruning is a second moving part with its own credential and
  schedule.
- Storage at the destination grows until the pruner runs; it must be monitored.
- Restores are unaffected — read access is retained.

## Related

- `docs/06-data-and-backup.md` — the backup and restore-testing design.
- Restore tests run against these snapshots on a schedule; a failed test pages
  the operator.
