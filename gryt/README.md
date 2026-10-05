# Gryt

[Gryt](https://gryt.chat) is an open source voice, video and text chat platform.

Connect with the [Gryt desktop app](https://github.com/Gryt-chat/gryt/releases) or the hosted web client at
[app.gryt.chat](https://app.gryt.chat). The first user to join a new server becomes its owner; afterwards the
server is invite-only (create invites under **Server settings → Invites**).

## Requirements

- **amd64 only** — the Gryt Linux bundle is published for x64 only.
- **3 port allocations** on the server (see below).

## Server Ports

| Allocation | Default | Proto | Variable | What it is |
|------------|---------|-------|----------|------------|
| default    | 5000    | TCP   | — (allocation port is used automatically) | Signaling server: chat, logins, everything except voice |
| extra #1   | 5005    | TCP   | `SFU_PORT` | SFU WebSocket (voice signaling) |
| extra #2   | 3478    | UDP   | `ICE_UDP_MUX_PORT` | WebRTC media — the actual voice/video |

After adding the two extra allocations to the server, set `SFU_PORT` and `ICE_UDP_MUX_PORT` to their port
numbers in the Startup tab.

## Required for voice over the internet

Set **SFU Public Host** (`SFU_PUBLIC_HOST`) to the address clients use to reach the SFU, e.g.
`203.0.113.10:5005` (or `wss://sfu.example.com` behind a TLS reverse proxy). Without it, chat works but
voice does not.

`ICE_ADVERTISE_IP` can usually stay empty — the SFU discovers its public IP via STUN. Set it explicitly when
the server is behind NAT and STUN picks the wrong address.

## Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `GRYT_VERSION` | `latest` | Version to install (e.g. `1.6.5`). Reinstall the server to apply. |
| `SERVER_NAME` | `My Gryt Server` | Display name shown to users |
| `SERVER_DESCRIPTION` | `A Gryt voice chat server` | Server description |
| `JWT_SECRET` | *(empty)* | Session token secret. Empty = auto-generated and persisted on first start |
| `SFU_PORT` | `5005` | TCP port of the SFU websocket — must match extra allocation #1 |
| `SFU_PUBLIC_HOST` | *(empty)* | Public SFU address given to clients (see above) |
| `ICE_UDP_MUX_PORT` | `3478` | UDP port all WebRTC media flows over — must match extra allocation #2 |
| `ICE_ADVERTISE_IP` | *(empty)* | Public IP(s) advertised in ICE candidates; may be required behind when voice does not work |
| `MAX_PEERS` | `200` | Max peers connected to the SFU at once |
| `STUN_SERVERS` | Google STUN | Comma-separated STUN URIs |
| `CORS_ORIGIN` | `http://127.0.0.1:15738,https://app.gryt.chat` | Allowed origins (desktop app + hosted web client) |
| `GRYT_IDENTITY_TIERS` | `account` | Add `local` to let people join without a Gryt account |
| `EXTERNAL_HOST` | *(empty)* | URL clients use to reach this server, e.g. `http://203.0.113.10:5000` |
| `IMAGE_WORKER_ENABLED` | `true` | Thumbnails, recompression and voice tile colors |
| `DISABLE_STUN` | `false` | Disable server-side STUN. Only with a direct, port-preserving path to the internet; set `ICE_ADVERTISE_IP` instead |

## Voice troubleshooting

If chat works but voice does not, check all three allocations:

- `SFU_PORT` must be reachable over TCP.
- `ICE_UDP_MUX_PORT` must be reachable over UDP.
- Both values must match the corresponding allocations.

If the desktop client cannot reach the voice server, or users connect without audio, set `ICE_ADVERTISE_IP` to the public IP through which the UDP media port is reachable. Do not use `127.0.0.1`, `0.0.0.0`, or a private container/LAN address. `DISABLE_STUN` should normally remain `false`.

## Updating

Set `GRYT_VERSION` and reinstall the server (or leave it on `latest` and reinstall). The `data/` directory
(SQLite database + uploads) and the auto-generated JWT secret (`.jwt_secret`) are kept, because the install
script only replaces the program files.

## Data

All server data lives in `data/` (`gryt.db` + `uploads/`). Back up the server by copying that directory
while the server is stopped.
