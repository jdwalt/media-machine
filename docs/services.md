# Services, placement, and startup

The October 3 host export supplies the definitions below. Numeric IDs and device paths describe the recorded machine; a new installation must match its own account and device inventory. `mediaadmin` replaces the deployed administrator account in public paths.

| Workload | Public source | Persistent state and access |
|---|---|---|
| ARM | `compose/arm/compose.json` | `/srv/arm/config`, `/home/arm/db`, logs and scratch; TCP 8080 |
| Jellyfin | `compose/jellyfin/compose.json` | Administrator home config/cache; read-only `/srv/media` at `/media`; TCP 8096, UDP 7359 |
| Pi-hole | `compose/pihole/compose.yaml` | `etc-pihole` next to Compose; external `pihole_lan` macvlan; DNS and web administration |
| Beszel | `compose/beszel/compose.yaml` | Agent state; outbound hub connection; loopback socket proxy |
| Scrutiny | `compose/scrutiny/compose.yaml` | Config and InfluxDB directories; LAN-bound TCP 8082; daily 02:00 collector |
| Portainer | `compose/portainer/compose.yaml` | Named `portainer_data` volume; LAN-bound HTTPS TCP 9443; Docker socket |
| Secondary Uptime Kuma | `compose/kuma/compose.yaml`, collected from Portainer | `louislam/uptime-kuma:2`; named `uptime_kuma_secondary_data` at `/app/data`; LAN-bound TCP 3001 |
| Samba | `config/samba/smb.conf.example` | Authenticated read/write Storage share at `/srv/storage` |
| SSH / SSHFS | `config/ssh/` | Host-account access; password authentication enabled in collected cloud-init drop-in |

## ARM and Jellyfin

ARM uses the local image `arm-qsv:2.24.3-hb1.9.2-ident-rev2-20260818`. The Rev2 Dockerfile layers the identification patch onto `arm-qsv:2.24.3-hb1.9.2`. The base recipe in `image/arm-qsv-base/` starts from ARM 2.24.3, installs the Intel media driver and tools, and builds HandBrake 1.9.2 with QSV, VCE and libdovi support. Its build checks the HandBrake version and Intel driver file. ARM is privileged in the collected Compose definition. Its explicit device mappings include `/dev/sr0`, `/dev/sg0`, and `/dev/dri/renderD128`; privileged access is broader than that list. The hardware has two optical drives. Their device identities must be checked on a rebuild.

ARM maps `/srv/arm` to `/home/arm/media`, overlays the completed media path with `/srv/media`, and separately mounts music, logs, config and `/home/arm`. UID/GID 1001 are supplied to the application. Current settings use MakeMKV extraction and `qsv_h264` HandBrake transcoding with English subtitle selection, a 60-second manual identification window, and application login enabled.

Jellyfin's collected image reference is `jellyfin/jellyfin:12.0`, with `pull_policy: never`. This records a local deployment reference, not a verified upstream release or available registry tag. It runs as 1000:1000 with supplementary GPU group 993, exposes the render node, sets `no-new-privileges`, and mounts media read-only. Cache and configuration remain writable. Preserve the private image inventory when recovering a locally tagged image.

Both containers use restart policy `no`. `media-containers.service` requires Docker and `srv-media.mount`, checks the mount plus movies/tv directories, and starts the already-created containers. Its `BindsTo=srv-media.mount` relationship stops the unit when that mount becomes inactive. It does not create containers or replace checks of the physical storage members.

## DNS and tablet

Pi-hole uses a separate LAN macvlan address on external network `pihole_lan`. The collected network has parent `enp5s0`, an IPv4 /24 subnet, gateway and /32 IP allocation range. `create-pihole-network` reproduces that structure with private deployment values. The host helper creates `pihole-shim` on `enp5s0`, gives it a /32 address, and routes the Pi-hole address through it. DNS clients use ordinary DNS queries; web administration uses an application password supplied privately to Compose. NTP functions are disabled in this definition.

`pihole-tablet-proxy.service` binds socat to `127.0.0.1:8081` and forwards to Pi-hole HTTP port 80. `arm-tablet-usb` reconnects ADB reverse mappings for 8080 and 8081 every five seconds, then launches Fully Kiosk after a new connection. The service runs as the account owning the authorized ADB key. The ASUS ZenPad uses Android 7 and recorded Fully Kiosk 1.61.2 with radios disabled and retained browser logins.

## Monitoring

Beszel uses host networking with `DISABLE_SSH=true`. Its `KEY`, `TOKEN`, and hub URL are private deployment inputs. The socket proxy publishes only `127.0.0.1:2375`, sets `CONTAINERS=1` and `POST=0`, mounts the Docker socket read-only, and uses a read-only filesystem with `/run` tmpfs. These are the collected proxy controls; access to Docker information still requires trust in the monitoring components.

Scrutiny runs v0.9.3 omnibus with `SYS_RAWIO` and `SYS_ADMIN`, a read-only udev mount, and explicit access to five HDD nodes and the NVMe controller. Portainer has direct Docker socket access and therefore administrative control over this host. Their local browser access and application credentials remain within the LAN.

The primary Uptime Kuma and Beszel hub live on another LAN system. The local Kuma container is `uptime-kuma-kumamon`. The reporting helper reads token-bearing URLs from `/etc/uptime-kuma-task-monitor.conf` and reports local replication health daily at 06:00 Pacific, adding the weekly configuration result on Sunday.

## Related systems

Mini-media is an HP Z240 motherboard in a Z230 chassis. Transfers are manual and currently on hold; no automated bidirectional synchronization is active. Application metadata and users remain separate. The Pi 3 B+ runs Kodi with the Jellyfin plugin, authorized through Quick Connect using an existing Jellyfin account. Back-off is a separate Z230 project that initiates SSH-authenticated source pulls. Its jobs and Uptime Kuma notifications on PMVEN1 are managed entirely by back-off and are outside this repository. The [access matrix](access-and-authentication.md) documents these integration boundaries.
