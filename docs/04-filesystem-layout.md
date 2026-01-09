# Filesystem Layout

This document describes how storage is structured inside the Ubuntu VM running the media stack. The layout is intentionally simple, explicit, and optimized for Docker bind mounts, backups, and rebuilds.

The guiding principle is **clear separation of concerns**.

---

## High-Level Storage Strategy

| Storage Type | Purpose |
|-------------|--------|
| NVMe (Proxmox host) | Proxmox OS + VM disk |
| SATA SSD (raw passthrough) | App configs, metadata, transcoding |
| NAS (NFS) | Media libraries and downloads |

Media files never live on the VM itself. The VM is disposable; the data is not.

---

## SSD Layout (Inside the VM)

The secondary SATA SSD is passed through raw to the VM and partitioned for distinct workloads.

### Mount Points

| Mount Point | Purpose | Notes |
|------------|--------|------|
| `/mnt/plexdata` | Plex metadata & database | High IOPS, persistent |
| `/mnt/plextranscode` | Transcoding / temp IO | High write churn |
| `/mnt/projects` | App configs & projects | All non-Plex apps |

Each partition is mounted via **UUID** in `/etc/fstab` with `noatime`.

---

### Why Separate These?

- Plex metadata benefits from fast random IO
- Transcoding should never compete with metadata
- App configs should be easy to back up independently
- One noisy workload should not degrade the others

---

## NAS Mounts (NFS)

Media and downloads live on a Synology NAS and are mounted into the VM using NFSv4.

### Mount Table

| Mount Point | Source | Purpose |
|------------|-------|--------|
| `/mnt/media/Movies` | NAS | Movie library |
| `/mnt/media/TV` | NAS | TV library |
| `/mnt/downloads` | NAS | SABnzbd download target |

These mounts are static (no automount) and use `_netdev` to avoid boot blocking.

---

## Why NFS (Not SMB)?

- Better Linux-native behavior
- Cleaner permissions handling
- More predictable performance for media workloads

---

## Docker Bind Mount Strategy

All containers use **bind mounts only** — no Docker-managed volumes.

### Plex Example

| Host Path | Container Path |
|----------|---------------|
| `/mnt/plexdata` | `/config` |
| `/mnt/plextranscode` | `/transcode` |
| `/mnt/media` | `/media` |

---

### *arr Apps (Radarr, Sonarr, etc.)

| Host Path | Container Path |
|----------|---------------|
| `/mnt/projects/<app>/config` | `/config` |
| `/mnt/media` | `/media` |
| `/mnt/downloads` | `/downloads` |

---

## Permissions Model

- All containers run as non-root
- Unified user:
  - `PUID=1000`
  - `PGID=1000`
- Ownership enforced on host directories

This avoids:
- Root-owned files
- Permission drift
- Inconsistent behavior between containers

---

## Why This Layout Works Well

- Easy to back up (`rsync`, `restic`, tar)
- Easy to restore on a fresh VM
- Easy to reason about in Docker configs
- Easy to document and share

If the VM dies, the filesystem layout makes rebuilding boring — which is the goal.

---

## Rebuild Checklist (Filesystem)

1. Attach raw SSD to VM
2. Partition and format SSD
3. Mount partitions to:
   - `/mnt/plexdata`
   - `/mnt/plextranscode`
   - `/mnt/projects`
4. Add UUID entries to `/etc/fstab`
5. Mount NAS shares to:
   - `/mnt/media`
   - `/mnt/downloads`
6. Verify ownership and permissions
7. Start Docker stack

If these steps work, everything else is configuration.
