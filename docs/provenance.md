# Provenance and licensing

Jack Walter designed, assembled, configured, and tested media-machine. ChatGPT assisted with organizing his project records into public build documentation. Original repository material is licensed under Apache License 2.0. Upstream applications and adapted source retain their required licenses and notices.

This is an as-built account of a project developed over several months. It brings together completion reports, operating notes, owner clarifications and selected installed files after the system entered service. It describes the working build and the choices made along the way, rather than a step-by-step journal of its construction.

The *Media Machine ASUS Z490-A / HAF 922 Project Completion Report*, completed September 20, 2026, documents the current physical platform and migration. Focused local-backup, ingestion, file-access and tablet records supply implementation and validation details. The October 3 installed-file export supplies the scripts and deployment definitions. The follow-up collection recovered retained image-build recipes and the live secondary Kuma stack.

Public source distinguishes collected deployment files, credential redactions, portable examples and reconstructed steps. Actual credentials, account databases and recovery archives remain private. The [source inventory](artifact-inventory.md) records coverage and collection dates.

Back-off is a separate Z230 backup project, and the Pi 3 B+ is a Kodi/Jellyfin playback client. Separate public build documentation for those systems is planned. Their integration with media-machine is documented here.

[DriveQual](https://github.com/jdwalt/DriveQual) is the related storage-qualification project. This server repository is independent of it.

## October 3 source export

Current scripts and deployment definitions were collected as redacted text and reviewed by the owner. Public adaptations repair collector-masked program expressions, replace deployed administrator names with `mediaadmin`, load sensitive identities/credentials from private examples, remove obsolete Dell-only diagnostic telemetry, consolidate an identical Samba section, and update the ARM source-hash label. They were not installed back onto the production host.

The adapted `image/arm-rev2/identify.py` derives from Automatic Ripping Machine 2.24.3. The upstream source was fetched from https://github.com/automatic-ripping-machine/automatic-ripping-machine/blob/2.24.3/arm/ripper/identify.py for comparison and restoration of the collector-masked public CRC API URL. The diff retains the local Rev2 changes. ARM's upstream MIT notice is retained in `third-party/ARM-LICENSE`; the repository's Apache 2.0 license covers original repository material. Other applications keep their own upstream licenses.

Restoring the upstream CRC URL yields SHA-256 `f33fa61823fb7abfa05fad5b33355187aea34885d11a3bd3c70249e6ff69f14c`, matching the hash label in the collected deployed Dockerfile.

Owner clarification, October 3: the Pi uses Kodi’s Jellyfin plugin and Quick Connect with an existing account. Mini-media transfers are manual and currently on hold, with no active automated bidirectional service. Back-off manages its pull jobs and PMVEN1 Kuma notifications independently; media-machine supplies data and SSH key authorization.

## Recovered build definitions

The QSV base Dockerfile and HandBrake 1.9.2 installer were recovered from the September 27 configuration snapshot. The installer includes its own Expat/MIT permission notice and attribution to the tianon HandBrake Dockerfile; that notice remains intact. The collector-masked Rustup URL was restored to `https://sh.rustup.rs`, documented by the official [Rust installation page](https://rust-lang.org/tools/install/). This is a public-source reconstruction, not a claim of byte-for-byte recovery of the original installer.

The live secondary Kuma Compose file was read from Portainer’s data volume. Its container-internal path `/data/compose/1/docker-compose.yml` maps to the host volume’s `compose/1/docker-compose.yml`. Its LAN address is replaced with the common documentation host address.

The macvlan creation helper is new repository code based on the collected driver, parent and IPAM field structure. It is not an exported historical host script. Private addresses are supplied locally.
