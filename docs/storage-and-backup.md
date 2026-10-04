# Storage and backup

## Data layout

The NVMe holds the operating system, Docker engine, and deployment files under `/home` and `/opt`. A separate approximately 808GB WDC disk is mounted at `/srv` for working data. Nested mounts expose the primary and backup pools within that hierarchy.

| Data class | Working location | Protection |
|---|---|---|
| Completed media | `/srv/storage/media`, presented through `/srv/media` | Local replication; source for back-off; separate mini-media copy with manual transfers on hold |
| Family files | `/srv/storage/personal` | Local replication; source for back-off |
| Configuration snapshots and image archive | `/srv/storage/homelab/media-machine` | Local replication; source for back-off |
| ARM raw/transcode/review | `/srv/arm` | Temporary work intentionally excluded from primary protection |
| Application runtime state | Under `/home`, `/opt`, and `/srv/arm/config` | Selected weekly capture described below |

Two ext4 members form the primary mergerfs view and two independent ext4 members form the local-backup view. Each pool is about 8TB decimal before filesystem overhead, displayed at roughly 7.2TiB. Pooling is not parity or mirroring. The collected fstab uses `category.create=mfs`, `minfreespace=100G`, `cache.files=off`, `func.getattr=newest`, and `allow_other`. Each pool depends on its two member mounts. `/srv/storage/media` is bind-mounted at `/srv/media`. The public fstab gives each filesystem a distinct replacement UUID role.

## Daily local replication

`replicate-storage-local` copies the whole `/srv/storage/` tree into `/srv/backup-local/`, using `storage-replication.service` and a daily 01:00 America/Los_Angeles timer. It uses a lock to prevent overlapping runs.

- New and changed files enter the live backup tree.
- Source deletions are not propagated.
- Displaced versions of changed destination files are kept below `.versions/<timestamp>/`.
- New top-level primary folders are automatically included.

The initial September 9 run examined about 685GB, transferred about 485GB, and was followed by a dry run reporting zero changes. The September 18 completion report also records a successful September 13 scheduled run at 08:00:09–08:00:47 UTC, with exit code 0 and no warnings in the queried journal. These dates identify captured validation evidence, not the most recent live run. The October 3 export contains the deployed script and timer definitions rather than a current execution log. An owner-supplied October 3 status check reports `Result=success` and `ExecMainStatus=0`; the timer last triggered October 3 at 08:00:13 UTC (01:00:13 Pacific), with its next trigger scheduled for October 4. Execution start/end timestamps were empty, so that output establishes the timer activation and reported status without dating a completed run.

Renames and reorganizations can leave old paths in this backup. The helper requests hard-link, ACL, extended-attribute and numeric-ID preservation through rsync. It does not delete source-removed files or prune nonempty version history; empty version directories are removed after a run.

## Weekly configuration capture

`media-machine-config-backup` runs Sunday at 00:15 America/Los_Angeles. It writes a timestamped snapshot under `/srv/storage/homelab/media-machine/backups/`, promotes completed work, and advances `LATEST`.

The recorded helper locks against overlap, stops the selected running containers while copying their state, restores their running state on exit, uses an incomplete directory until completion, generates `SHA256SUMS`, and uses hard links for unchanged snapshot files.

The collected helper stops the running instances of Pi-hole, ARM, Jellyfin, Portainer and secondary Kuma, then captures their selected state. Sources include `/opt/pihole`, `/opt/portainer`, `/opt/uptime-kuma-secondary`, the Portainer/Kuma named-volume data, ARM config/logs/home and Jellyfin config. ARM media/music/log duplicates are excluded from its home copy. Host capture includes SSH configuration, local scripts, units, Netplan, cron files, package inventory and full Docker inspection. Beszel and Scrutiny application-state directories are not in this helper’s selected source list.

The active custom ARM image has a separately verified compressed archive. It is a recovery asset, not public GitHub source. This repository contains the reviewed build context and patch instead.

The unit requires the primary pool mount. The helper preflights its containers and source directories, acquires a lock, restarts previously running containers through an EXIT trap, writes `incomplete-<timestamp>` until completion, and advances `LATEST` only after checksums are generated. No snapshot-pruning command is present. These private snapshots contain credentials and runtime state and must stay outside the public repository. Checksums verify saved bytes; a rebuild validates recovery behavior.

## External consumers and peers

Media-machine provides source data to back-off, a separate Z230 backup project. This repository documents the source paths and SSH key authorization on media-machine. Back-off manages its own storage, schedules, monitoring, restore procedures and power controls. Separate public build documentation for that project is planned.

Mini-media holds a separate data copy. Transfers are manual and currently on hold; no automated bidirectional synchronization is active. Application metadata and user accounts remain host-local.

The primary and local-backup pools share this host's chassis, power, and operating system. Their recorded local replication behavior is documented here; external backup operation is a separate project responsibility.
