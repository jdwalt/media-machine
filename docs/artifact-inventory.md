

# Source inventory

This repository gathers the source for a working household server whose design developed through several builds. Completion reports explain the choices and tests; selected installed files show how the services are actually connected. The documentation was assembled after the build, with private state separated from reusable source.

| Period | Record or collection |
|---|---|
| August 2026 | Custom ARM Quick Sync and Rev2 identification work; the image tag records August 18 |
| September 9–13 | Initial local-replication qualification and a later successful scheduled run |
| September 20 | ASUS platform completion report and migration acceptance |
| September 27 | Retained configuration snapshot used to recover the QSV base build recipe |
| October 3, Pacific time | Owner review, selected installed-file export and follow-up build-definition collection; some archive names use October 4 UTC |

The October 3 export supplied 37 selected text files. A follow-up collection supplied the QSV base Dockerfile and HandBrake installer from the September 27 configuration snapshot and the live secondary Kuma Compose definition from Portainer’s volume.

Public source now includes ten deployed host helpers, one reconstructed network-creation helper, eleven systemd units/timers, seven Compose definitions, storage/network/SSH/Samba/ARM templates, the QSV base recipe and the Rev2 ARM image context. No identified service source remains awaiting collection. Private application state, credentials, diagnostics and recovery archives are excluded.

| Source | Collection and public representation |
|---|---|
| QSV base image | Retained `home/arm/arm-qsv-image` snapshot; `image/arm-qsv-base/` |
| Rev2 identification layer | Deployed patch directory; `image/arm-rev2/` |
| Secondary Kuma | Portainer volume `compose/1/docker-compose.yml`; `compose/kuma/compose.yaml` |
| Pi-hole macvlan | Selected runtime network fields; parameterized creation helper and private config example |
| Core services and monitoring | Collected Compose definitions with credential separation |
| Local backup and diagnostics | Collected host scripts and units with documented public adaptations |

The initial ARM deployment manifest describes an earlier transition and conflicts with current restart/image values. Current Compose and units take precedence, so that historical manifest is not shipped. Stock crontab entries are not project deployment logic. Duplicate Samba Storage sections are consolidated in the public template.

The Pi uses Kodi’s Jellyfin plugin and Quick Connect through an existing account. Mini-media transfers are manual and currently on hold. Back-off’s jobs and PMVEN1 Kuma notifications belong to its separate project; media-machine supplies data and SSH key authorization.

Public adaptations are described in [provenance](provenance.md) and `scripts/README.md`. `source-manifest.json` records repository-relative paths and hashes of the adapted sources, never deployed secret-bearing originals.
