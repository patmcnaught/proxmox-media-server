## Proxmox Media Server — A Reproducible, GPU-Accelerated Media Stack

This repository documents a **fully working Proxmox-based media server** built with long-term maintainability in mind.

It is not a one-click installer.

It *is* a real-world, battle-tested playbook that shows how to build a media stack that can be **rebuilt, recovered, and understood** months or years later.

### What This Covers
- Proxmox VE as a clean hypervisor
- Ubuntu Server VM (disposable by design)
- NVIDIA GPU passthrough for hardware transcoding
- Raw SSD passthrough for app data and metadata
- NAS-backed media via NFS
- Docker Compose–managed media stack:
  - Plex
  - Jellyfin
  - Radarr / Sonarr / Prowlarr
  - SABnzbd
  - Overseerr
- Bind mounts only (no opaque Docker volumes)
- Restic-based application backups
- Apple TV Plex discovery fixes
- Real failure modes and lessons learned

### What Makes This Repo Different
Most homelab guides show the *happy path*.

This repo documents:
- What broke
- Why it broke
- How it was fixed
- How to avoid repeating it

Every design decision is intentional:
- VMs are disposable
- Data is portable
- Recovery is boring (by design)

If you’ve ever rebuilt a media server and thought *“I wish past-me had written this down”* — this repo is for you.

---

### Who This Is For
- Homelabbers using Proxmox
- Anyone running Plex or Jellyfin in Docker
- People who care about backups and rebuilds
- Folks who prefer clarity over cleverness

---

### Repo Status
✔️ Actively used  
✔️ Fully functional  
✔️ Documented end-to-end  
✔️ Designed to be cloned, adapted, and reused  

Feel free to fork it, template it, or use it as a reference.

-----------------------------------------------------------------

# Proxmox Media Server

A documented, reproducible media server built on **Proxmox VE**, using **Docker**, **GPU passthrough**, **NAS-backed media**, and **SSD-backed application data**.

This repository is a **playbook**, not a one-click installer. It documents a working, real-world setup with lessons learned along the way.

---

## Goals

- Stable, reproducible Proxmox-based media server
- Clean separation of OS, app data, transcode space, and media
- GPU-accelerated transcoding (NVENC)
- Apple TV–friendly Plex discovery
- Docker-first application management
- Bind mounts (no opaque Docker volumes)
- Practical backup and restore strategy

---

## High-Level Architecture

[ Proxmox VE ]
|
├── Ubuntu Server VM
│ ├── Docker Engine
│ ├── Media Apps (Plex, Jellyfin, *arr stack)
│ └── Restic (backups)
│
├── NVMe (OS + VM disk)
└── SATA SSD (raw passthrough)
├── /mnt/plexdata
├── /mnt/plextranscode
└── /mnt/projects

Media files live on a **Synology NAS** and are mounted into the VM via **NFS**.

---

## What This Repo Contains

- 📘 Step-by-step documentation under `/docs`
- 🐳 A sanitized Docker Compose stack under `/docker`
- 🧠 Real-world troubleshooting notes and lessons learned
- 🔁 A structure designed for rebuilds and future upgrades

---

## What This Repo Is NOT

- ❌ A turnkey installer
- ❌ A “clone and run” solution
- ❌ Optimized for beginners with no Linux experience

Some comfort with Linux, Docker, and home networking is assumed.

---

## Repository Structure

proxmox-media-server/
├── README.md
├── docker/
│ ├── docker-compose.yml
│ ├── .env.example
│ └── README.md
├── docs/
│ ├── 01-hardware-and-assumptions.md
│ ├── 02-proxmox-setup.md
│ ├── 03-disk-passthrough.md
│ ├── 04-filesystem-layout.md
│ ├── 05-docker-stack.md
│ ├── 06-backups-and-restores.md
│ └── 07-troubleshooting-and-lessons.md
└── scripts/

---

## Hardware Assumptions (Example)

- Proxmox host with VT-d / IOMMU support
- Dedicated GPU for passthrough (NVIDIA)
- NVMe boot drive
- Secondary SSD for app data and transcoding
- NAS or external storage for media

---

## Status

✔️ Fully functional  
✔️ Plex + Apple TV discovery working  
✔️ GPU transcoding confirmed  
✔️ Docker stack stable  

This documentation will evolve as the stack does.

---

## License

TBD
