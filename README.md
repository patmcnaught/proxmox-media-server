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
