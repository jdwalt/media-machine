# Recovery procedure

This is the recovery sequence derived from project records. A complete isolated rebuild has not been performed. The public package excludes recovery secrets and runtime databases, so it cannot restore the server by itself.

The simplest recovery approach is a clean Ubuntu installation on a spare system disk, followed by configuration and application-state restoration. This can test completeness while preserving the working NVMe. Repairing an individual installation remains useful for a small, understood fault; an extended boot repair is not required to prove this documented build. This process is informed by the build of mini-media and the migration of media-machine from the original experiment to its current production state. 

## Rebuild order

1. Select a complete timestamped private configuration snapshot and verify its relative-path `SHA256SUMS` from inside the snapshot directory.
2. Install the recorded Ubuntu baseline and required host packages on an isolated spare system disk. Prevent IP conflicts and unintended connection to production storage.
3. Recreate required accounts, numeric user/group IDs, filesystem ownership, and permissions from private records.
4. Restore the physical member mounts, scratch mount, mergerfs pools, and media view. Substitute actual hardware identities and confirm each expected filesystem is mounted.
5. Restore the reviewed scripts, systemd units, Compose projects, application databases/configuration, and protected secret files. Do not restore obsolete Dell NIC or fan assumptions.
6. Verify and load the archived custom ARM image if its tag is absent. The public base-image Dockerfile, HandBrake installer and Rev2 layer provide the build recipe; the retained image archive preserves the already-built recovery artifact. Restore the recorded upstream image identities for other services.
7. Adapt Netplan and Pi-hole's external macvlan parent. Restore the host shim and validate DNS.
8. Start ARM/Jellyfin through their mount guard. Confirm both optical devices, Intel GPU access, application state, and representative playback.
9. Restore ADB authorization under the intended service account, reconnect the tablet, restore Fully Kiosk URLs/settings, and sign in to both web applications.
10. Restore Samba/SSH authorization from private configuration; verify the intended Storage user can read/write and an unauthorized identity is denied.
11. Restore monitoring, backup timers, and push credentials. Confirm actual results, not just that units are enabled.
12. Document elapsed recovery time, missing files, manual interventions, and the exact evidence produced.

## File restore

Restore data into a separate temporary directory before replacing production files. Compare hashes and, when appropriate, open the recovered file or validate the application's database. Check numeric ownership and modes before copying back.

For local replication, use the live backup path or a retained version under `.versions`. Recovery from back-off follows that separate project's restore procedure, then media-machine ownership and service-state validation apply here.

## Read-only operating checks

```bash
findmnt -T /srv/storage
findmnt -T /srv/backup-local
findmnt -T /srv/media
docker ps --format 'table {{.Names}}\t{{.Image}}\t{{.Status}}'
systemctl list-timers media-machine-config-backup.timer storage-replication.timer --no-pager
systemctl show media-machine-config-backup.service storage-replication.service -p Result -p ExecMainStatus
```

The returned outputs can contain deployment details. Review them before publishing. A directory existing is not sufficient evidence that its intended filesystem is mounted.
