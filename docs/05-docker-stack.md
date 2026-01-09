# Docker Stack

This document describes how the media server applications are deployed using Docker. The stack is intentionally simple, readable, and optimized for long-term maintenance rather than clever abstractions.

The goal is that every container:
- Is easy to understand
- Uses predictable paths
- Can be backed up and restored cleanly

---

## Core Principles

- Docker Compose for orchestration
- One stack, one `docker-compose.yml`
- Bind mounts only (no Docker volumes)
- Non-root containers
- Explicit networking
- Explicit paths

This stack prioritizes **clarity over cleverness**.

---

## Applications in the Stack

| Application | Purpose |
|------------|--------|
| Plex | Primary media server |
| Jellyfin | Secondary / alternative media server |
| SABnzbd | Usenet downloader |
| Radarr | Movie management |
| Sonarr | TV management |
| Prowlarr | Indexer management |
| Overseerr | Media request management |
| Restic | App data backups |

---

## Directory Mapping Strategy

All container configuration lives under `/mnt/projects` or `/mnt/plexdata`.

### App Config Layout

/mnt/projects/
├── radarr/
│ └── config/
├── sonarr/
│ └── config/
├── prowlarr/
│ └── config/
├── overseerr/
│ └── config/
├── sabnzbd/
│ └── config/
└── jellyfin/
├── config/
└── cache/

Plex is treated separately due to its size and IO profile.

---

## Plex Container

### Bind Mounts

| Host Path | Container Path |
|----------|---------------|
| `/mnt/plexdata` | `/config` |
| `/mnt/plextranscode` | `/transcode` |
| `/mnt/media` | `/media` |

### Notes

- GPU acceleration (NVENC) enabled
- Custom server access URL explicitly set
- Apple TV discovery tested and working

---

## Jellyfin Container

### Bind Mounts

| Host Path | Container Path |
|----------|---------------|
| `/mnt/projects/jellyfin/config` | `/config` |
| `/mnt/projects/jellyfin/cache` | `/cache` |
| `/mnt/plextranscode/jellyfin` | `/transcode` |
| `/mnt/media` | `/media` |

### Notes

- Hardware acceleration enabled
- Treated as stateless and rebuildable
- Clean reset preferred over auth repair

---

## Downloader: SABnzbd

### Image Choice

binhex/arch-sabnzbd


This image was chosen after issues with IPv6-only binding in other images.

### Bind Mounts

| Host Path | Container Path |
|----------|---------------|
| `/mnt/projects/sabnzbd/config` | `/config` |
| `/mnt/downloads` | `/downloads` |

### Notes

- Explicit IPv4 binding
- API connectivity verified with *arr apps
- Post-processing and imports confirmed

---

## *arr Stack (Radarr, Sonarr, Prowlarr)

### Shared Mounts

| Host Path | Container Path |
|----------|---------------|
| `/mnt/projects/<app>/config` | `/config` |
| `/mnt/media` | `/media` |
| `/mnt/downloads` | `/downloads` |

### Notes

- Consistent paths across all apps
- Simplifies import logic
- Symlink workaround used for Radarr UI path selection

---

## Overseerr

### Bind Mounts

| Host Path | Container Path |
|----------|---------------|
| `/mnt/projects/overseerr/config` | `/config` |

### Notes

- Talks only to Radarr/Sonarr
- No direct access to media or downloads required

---

## Networking Model

- Single Docker bridge network
- Containers refer to each other by service name
- No host networking (except where explicitly required)

This avoids:
- Port collisions
- Hardcoded IP dependencies
- Fragile LAN assumptions

---

## Environment Variables

All containers use:
- `PUID=1000`
- `PGID=1000`
- Shared timezone

Variables are defined in `.env` and documented in `.env.example`.

---

## Why One Compose File?

- Easier to reason about
- Easier to start/stop the entire stack
- Easier to back up configs
- Easier to document

This is not a microservices problem.

---

## Startup Order (Conceptual)

1. NAS mounts available
2. Docker daemon starts
3. Downloader (SABnzbd)
4. *arr apps
5. Plex / Jellyfin
6. Overseerr

Compose handles ordering implicitly; manual sequencing is rarely needed.

---

## Final Takeaway

This Docker stack is boring by design.

Boring stacks:
- Break less
- Recover faster
- Age better

If the filesystem layout is correct, the Docker stack becomes easy.
