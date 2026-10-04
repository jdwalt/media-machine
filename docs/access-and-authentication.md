# Access and authentication

All service access is confined to the LAN. The public examples use `192.0.2.x` addresses, `mediaadmin` as an administrator role alias, and empty private credential fields. Port publishing in Compose alone does not establish the network boundary; LAN routing and the surrounding network configuration supply that boundary. No outside-LAN service exposure is part of this build.

| Actor | Interface | Authentication | Access |
|---|---|---|---|
| Four household viewers | Jellyfin TCP 8096 | Four individual Jellyfin accounts | Assigned media playback |
| Pi playback client | Kodi with Jellyfin plugin | Quick Connect authorized using an existing Jellyfin account | Playback under that account’s permissions |
| ARM operator | ARM TCP 8080 | ARM application login; `DISABLE_LOGIN=false` | Jobs, identification and ingestion controls |
| LAN DNS client | Pi-hole TCP/UDP 53 | No per-query user login | DNS resolution/filtering |
| DNS administrator | Pi-hole web UI | Private application password | DNS administration |
| Host administrator | SSH / SSHFS | Host account; password authentication enabled in collected drop-in | Account filesystem permissions and separately granted administration |
| One Windows user | Samba Storage, TCP 445 | Samba authentication as the permitted host-account role | Read/write `/srv/storage`; guest access disabled for this share |
| USB kiosk | ADB and local web sessions | Authorized ADB host key plus retained ARM/Pi-hole logins | Forwarded interfaces on tablet localhost 8080/8081 |
| Mini-media | Manual file transfers, currently on hold | Operator-managed transfer access; no active automated job | Independent ingestion/playback services and local library; metadata and users remain host-local |
| Back-off | SSH-authenticated pull | Source-side SSH key authorization on media-machine | Access granted by the authenticated account’s filesystem permissions |
| Beszel agent | Outbound hub connection | Private `KEY` and `TOKEN` | Host, container and filesystem telemetry |
| Task reporter | Uptime Kuma push endpoint | Token-bearing monitor URL in root-only config | Report the designated monitor state |
| Portainer administrator | LAN HTTPS TCP 9443 | Private application account/session | Docker host administration through its socket |

## Filesystem and share permissions

The Storage share allows authenticated read/write access to `/srv/storage` for the permitted user, with guest access disabled. New directories retain group ownership through setgid. Exact masks and share settings are included in `config/samba/smb.conf.example`.

Samba credentials and Jellyfin users are separate identities. SSHFS acts with the authenticated Linux account's filesystem permissions. Share settings describe service authorization, while actual ownership, modes, groups and ACLs determine filesystem access. Private account databases, password hashes, authorized keys and application databases are not published.

The SSH template retains the collected password-enabled host policy. Back-off uses separate SSH key authorization for its incoming pull session; the authenticated account's filesystem permissions determine which source data it can read.

## Machine relationships

Mini-media is a physical counterpart that reproduces media-machine’s service capabilities on Ubuntu Desktop, including its own ARM ingestion and Jellyfin playback. It is scaled to one optical drive and one storage drive, with no internal backup. Its build helped prove out the approach during media-machine’s evolution. Its data, application metadata and users are maintained separately from media-machine. An earlier bidirectional transfer service brought additions into both libraries while respecting local deletion. Transfers have since returned to manual operation and are currently on hold; no automated bidirectional service is active. 

Back-off is an off-site backup host that initiates and manages its own pull jobs. Media-machine provides only the source data and SSH key authorization that permits the incoming session to authenticate. Deployed keys and fingerprints are excluded from public source. Host-key verification establishes server identity separately from client authentication. Back-off’s notifications to Uptime Kuma on PMVEN1 are entirely separate from media-machine’s local replication and configuration-backup reporting.

A Pi 3 B+ runs Kodi with the Jellyfin plugin. Its Quick Connect code authorizes the client through one of the four existing Jellyfin login accounts. It does not add a fifth account or provide anonymous playback. Client credentials and session state remain private. Separate build documentation for the Pi playback client and back-off is planned.

## Private configuration

| Input | Private placement | Public schema |
|---|---|---|
| ARM metadata and optional integration keys | Protected `/srv/arm/config/arm.yaml` | `config/arm/arm.yaml.example`, credential values empty |
| Pi-hole web password | Protected Compose environment | `compose/pihole/secrets.env.example` |
| Beszel enrollment key/token and hub URL | Protected Compose environment | `compose/beszel/secrets.env.example` |
| Kuma push URLs | Root-owned mode 0600 `/etc/uptime-kuma-task-monitor.conf` | Matching `.conf.example` in `config/` |
| Tablet identity | Service-account-readable `/etc/arm-tablet-usb.conf` | `config/arm-tablet-usb.conf.example` |
| Expected diagnostic disk serials | Root-owned mode 0600 `/etc/media-machine-diagnostics.conf` | Matching `.conf.example` in `config/` |
| Linux, Samba and Jellyfin accounts | Private OS/application stores | Roles and service rules only |
| ADB keys and browser sessions | Private administrator profile and tablet state | Authorization procedure only |

The environment/config separation in these public templates is a publication adaptation. The export embedded some values directly in deployed definitions. Filled secret files, resolved Compose output, full Docker inspections, local snapshots and configuration backups remain private operating records and are excluded from this repository.
