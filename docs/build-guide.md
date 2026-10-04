# Build and deployment notes

These notes describe the recorded host and the steps needed to adapt its deployment files. Read the service, storage and access documents first. The public templates require local substitution and private state; they are not an unattended installer. No formatting or partitioning commands are provided because storage must be selected and verified on the target host.

## 1. Platform, accounts and storage

Install the recorded Ubuntu Server baseline, Docker Engine with Compose, mergerfs, rsync and the host utilities listed in `scripts/README.md`. Recreate an administrator role (`mediaadmin` in examples) and ARM's `arm` account. The collected deployment uses administrator UID/GID 1000 and ARM UID/GID 1001; substitute actual values together with directory ownership. Identify the GPU render group instead of assuming group 993.

To recreate the recorded layout, create mountpoints and substitute six ext4/NVMe UUID roles plus the EFI UUID in `config/fstab.example`. Preserve existing data. Mount the scratch disk at `/srv`, primary members at `/mnt/storage-p1` and `p2`, backup members at `/mnt/backup-p1` and `p2`, and mergerfs pools at `/srv/storage` and `/srv/backup-local`. The media bind mount exposes `/srv/storage/media` at `/srv/media`. Confirm actual member filesystems and pool mounts with `findmnt`, then create or restore movies, tv and music directories with the intended permissions.

## 2. LAN networking and DNS

The production LAN interface uses DHCP. A reservation on the router ties the host's LAN address to its NIC MAC address; the router supplies the gateway and both DNS resolver addresses. Adapt the interface name in the LAN Netplan template and create the corresponding reservation on your router. The separate direct 10GbE interface is optional and uses a static /30 link. Its `192.0.2.x` values are documentation placeholders, not a usable direct-link subnet or peer assignment.

The collected external Docker network `pihole_lan` uses macvlan on `enp5s0`, an IPv4 /24 subnet, gateway and /32 allocation range. Copy `config/pihole-network.conf.example` into a protected private `/etc/pihole-network.conf` and substitute your actual values. Run `sudo bash scripts/create-pihole-network` only when deploying a new network; it refuses documentation addresses and leaves an existing network unchanged. This helper reconstructs the collected network shape; the original creation command was not retained. Match the reserved Pi-hole address across Compose, the shim helper and the tablet proxy. Reserve a separate shim address in the LAN and preserve its host /32 route to Pi-hole.

Keep access within the LAN and apply the intended local routing/firewall policy. Compose's ARM and Jellyfin port definitions publish to the host's default interfaces. Portainer and Scrutiny explicitly bind the LAN address in the examples. 

Portainer was added after ARM and Jellyfin were deployed. Secondary Uptime Kuma is the only stack deployed through and fully managed by Portainer. The other Compose projects were deployed outside Portainer; Scrutiny's definition is under `/opt/scrutiny`. Portainer could also manage other stacks, but this build has not validated QSV device configuration through Portainer. 

## 3. Container definitions and private inputs

Place Pi-hole at `/opt/pihole`, Beszel at `/opt/beszel-agent`, Scrutiny at `/opt/scrutiny`, Portainer at `/opt/portainer`, secondary Kuma at `/opt/uptime-kuma-secondary`, Jellyfin under the administrator's home, and ARM at `/home/arm/arm-qsv-deployment`. Preserve relative-volume semantics when copying Compose files into these directories. Adapt the configuration-backup helper if you choose different paths.

Copy the relevant `secrets.env.example` to a protected private environment file, fill values and use Compose's `--env-file` option. Pi-hole requires its web administrator password. Beszel requires its key, token and actual hub URL. Never commit the filled files. Restore or initialize application accounts through the application's normal setup process; the repository includes no account databases.

Copy `config/arm/arm.yaml.example` into protected `/srv/arm/config/arm.yaml`, fill required metadata credentials, and retain the upstream version's supporting configuration files. Review raw/transcode/completed paths and selected HandBrake arguments. Restore private ARM state separately when preserving existing jobs/users.

Build the QSV base image, then the Rev2 identification layer from the repository root:

```bash
docker build -t arm-qsv:2.24.3-hb1.9.2 image/arm-qsv-base
docker build -t arm-qsv:2.24.3-hb1.9.2-ident-rev2-20260818 image/arm-rev2
```

The base recipe was recovered from the retained September 27 configuration snapshot. It downloads HandBrake 1.9.2 source/signature, verifies the signature using the included upstream signing-key fingerprint, installs Rust build tools, and compiles HandBrake with QSV support. Its upstream base image, apt packages, Rust nightly channel and cargo-c release are not pinned by immutable versions; this records the working recipe rather than guaranteeing a byte-identical future image. The private deployment also holds the built image and a checked recovery archive. The Rev2 directory contains a diff against upstream ARM 2.24.3 and retains its MIT notice. Its public CRC lookup URL was restored from that upstream source after the collector masked URLs.

Check optical and GPU device mappings before creating containers. ARM's recorded privileged mode exposes more devices than its explicit list. The Jellyfin definition expects a locally available `jellyfin/jellyfin:12.0` image because it sets `pull_policy: never`; supply the recorded local image or deliberately choose and validate another image.

Create ARM and Jellyfin containers without starting them (`docker compose ... create`) after their mounts and writable state directories are ready. Other Compose projects use their recorded restart policies. The secondary Kuma definition was recovered from Portainer’s volume. It may be deployed through Portainer or standalone Compose while retaining the same container and named-volume identities. Use one manager for the stack.

## 4. Host scripts and units

Install `arm-tablet-usb` under `/usr/local/bin` and the other scripts under `/usr/local/sbin` as root-owned executable files. Copy the reviewed units to `/etc/systemd/system`, adapting administrator paths and local addresses first, then run `systemctl daemon-reload`.

Prepare private `/etc/arm-tablet-usb.conf` readable by the tablet service account and root-only mode 0600 `/etc/media-machine-diagnostics.conf` and `/etc/uptime-kuma-task-monitor.conf` from the matching examples. Authorize the administrator's ADB host key on the tablet, enable USB debugging, and configure Fully Kiosk with tablet-localhost URLs 8080 and 8081. Sign in to ARM/Pi-hole on the tablet. Keep the ADB key and browser profile private.

Start the shim and proxy after networking and Pi-hole are ready. Start ARM/Jellyfin through `media-containers.service`. Enable the relevant units and three timers only after checking their configuration and application paths. The configuration backup temporarily stops its selected running containers, so its first live invocation is an operating action, not a harmless syntax check.

## 5. Sharing, monitoring and validation

Adapt Samba's permitted user and filesystem permissions, initialize that Samba identity privately, and validate the file with `testparm` before loading it. Keep the collected SSH policy distinct from source-only machine authorizations. Preserve known-host verification for machine transfers. Back-off manages its own pull jobs; media-machine only needs the appropriate SSH key authorization and source filesystem access. No automated mini-media transfer service is part of this build.

Initialize Portainer and local monitoring accounts privately. Add service checks to primary/secondary Kuma and fill only the appropriate push URLs in the root config. Confirm the timers at 00:15 Sunday, 01:00 daily and 06:00 daily Pacific. Run diagnostic tools locally and review their private output. Validate DNS, both optical drives, QSV, representative Jellyfin playback, tablet reconnection, SMB allowed/denied access, local replication and configuration restoration. Record host execution results separately from the publication package's static checks.
