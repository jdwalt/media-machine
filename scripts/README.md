# Service and diagnostic scripts

The ten deployed helpers come from the October 3 redacted host export. The additional network-creation helper is a public reconstruction from the collected network structure. They are public adaptations, not a byte-for-byte recovery backup. Install only after adapting the build guide and private settings. No production script was run or changed during publication review.

| Script | Purpose | Host destination |
|---|---|---|
| `arm-tablet-usb` | Reconnect tablet ADB reverse ports and launch Fully Kiosk | `/usr/local/bin/` |
| `create-pihole-network` | Create the external macvlan network with private LAN parameters; manual deployment step | Run from the repository |
| `pihole-host-shim` | Create/remove host-to-Pi-hole macvlan shim and route | `/usr/local/sbin/` |
| `replicate-storage-local` | Mount-checked, locked, nondeleting local rsync with changed-file versions | `/usr/local/sbin/` |
| `media-machine-config-backup` | Quiesce selected containers, capture private state, checksum and promote snapshot | `/usr/local/sbin/` |
| `report-backup-health-to-kuma` | Check scheduled job completion and report through private push URLs | `/usr/local/sbin/` |
| `media-status` | Hardware/storage/service diagnostic snapshots | `/usr/local/sbin/` |
| `media-diff` | Compare two latest host snapshots | `/usr/local/sbin/` |
| `media-diffcontrol` | Save/show/compare a reviewed diagnostic baseline | `/usr/local/sbin/` |
| `arm-status` | ARM/Jellyfin diagnostics and snapshots | `/usr/local/sbin/` |
| `arm-diff` | Compare two latest application snapshots | `/usr/local/sbin/` |

Public adaptations repair collector-redacted expressions, load tablet/disk identities from private config, and save new diagnostic files mode 0600. The obsolete Dell fan telemetry section is omitted for this ASUS build. The application diagnostic retains operational error scanning; no historical fault report is included. Generated diagnostics can contain private addresses, filenames, serials and logs even when common token strings are masked. They are local operating records, not public examples.

Dependencies include Bash, coreutils, util-linux, rsync, Docker, curl, ADB, iproute2, socat, pciutils, smartmontools, lsscsi, ethtool, lm-sensors and ffprobe from FFmpeg as applicable. The scripts expect the mount/account layout in the build guide.
