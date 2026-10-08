# Backup and Recovery

## Strategy

Two independent mechanisms protect the environment:

1. **Proxmox `vzdump`** — whole-container snapshots to the Proxmox backup storage.
2. **Backrest (restic)** — file-level backups of selected data to two separate disks.

Backups are written to **two different physical disks** so that a single disk failure does not destroy all copies.

## What Is Backed Up

| Source | Method | Target |
|---|---|---|
| Containers 100/101/102 | `vzdump` snapshot | Proxmox backup storage |
| Containers 103/104/105 | `vzdump` snapshot | Proxmox backup storage |
| Docker projects directory | restic (via Backrest) | Disk #1 and Disk #2 |
| Application configs | restic (via Backrest) | Disk #1 and Disk #2 |

> Container snapshots protect the system; restic protects the data and configuration that live on the shared storage.

## Schedule

| Job | Frequency | Target |
|---|---|---|
| `vzdump` — core containers | Weekly (Sunday, 04:00) | Proxmox backup storage |
| `vzdump` — infra containers | Daily (01:30) | Proxmox backup storage |
| restic backup "home-server" | Daily 02:00 | Repository "server-raid" |
| restic backup "home-server-hdd320" | Daily 02:30 | Repository "server-hdd320" |
| restic prune | Weekly (03:00) | Both repositories |
| restic integrity check | Monthly (04:00) | Both repositories |
| garbage collection | Daily (13:02) | — |

- The schedule lives **inside Backrest**, not in cron, so it recovers automatically with the container.

TODO: confirm the exact daily/weekly cadence is still current.

## Retention

- restic pruning removes old snapshots on a schedule.
- `check` runs monthly to verify repository integrity.

TODO: document the exact retention policy (daily/weekly/monthly counts).

## Recovery Procedures

### Recover a container (vzdump)

1. Identify the desired snapshot in the Proxmox backup storage.
2. Restore to the original container ID, or restore to a new ID to inspect first.
3. Boot the restored container and verify services.

### Recover files (restic)

```bash
export RESTIC_REPOSITORY=/path/to/repo
restic snapshots
restic restore <snapshot-id> --target /tmp/restore
```

- The repository password is stored in a secret file, not in this repository.
- Never restore directly over a live path without inspecting the snapshot first.

### Recover the whole host (high level)

1. Install Proxmox; set hostname and management IP.
2. Create storage (LVM-thin) and import the ZFS mirror.
3. Recreate containers with the same IDs, resources, and mounts.
4. Add persistent mounts for the data disks.
5. Restore the automation container from its archive.
6. Bring up container stacks with `docker compose up -d`.
7. Restore host services: port proxy, tunnel, cron, OOM protection.
8. Verify backups by performing a test restore.

> Recreate containers **before** restoring data, so mount points exist.

## Verification

- Monthly `check` on both restic repositories.
- TODO: document a periodic full restore test and its result.

## Known Gaps / TODO

- TODO: Add off-site backup copy (3-2-1).
- TODO: Encrypt backup repositories at rest.
- TODO: Monitor backup job failures with an alert.
