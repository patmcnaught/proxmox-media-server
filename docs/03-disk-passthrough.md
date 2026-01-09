# Disk Passthrough (Raw SSD)

This document describes how a secondary internal SSD is passed through *raw* from Proxmox to the Ubuntu VM and used for application data and transcoding.

Raw disk passthrough is used instead of virtual disks to:
- Avoid abstraction layers
- Improve performance
- Make disk identity stable across reboots
- Simplify recovery and rebuilds

---

## Disk Overview

- Disk type: SATA SSD
- Size: 1TB
- Purpose: Application data, metadata, and transcoding
- Ownership: VM-exclusive (not shared with Proxmox)

---

## Why Raw Disk Passthrough?

Using raw passthrough instead of a virtual disk provides:

- Direct disk access (no QCOW2 overhead)
- Stable disk identity
- Cleaner failure modes
- Easier filesystem-level recovery

If the VM is destroyed, the disk can be reattached to a new VM intact.

---

## Identify the Disk on Proxmox

On the Proxmox host, list disks by stable ID:

```bash
ls -l /dev/disk/by-id/
Identify the SSD by model and serial number. Example:

text
Copy code
ata-WDC_WDS100T2B0B-00YS70_1818A5800554
⚠️ Always use /dev/disk/by-id/ paths
Never use /dev/sdX, which can change between boots.

Attach Disk to the VM (Proxmox UI)
Open the Proxmox web UI

Select the target VM

Go to Hardware

Click Add → Hard Disk

Set:

Bus/Device: VirtIO Block

Storage: Do not select storage

Disk: Use the full /dev/disk/by-id/... path

Confirm and apply

The disk is now passed through raw to the VM.

Verify Disk Inside the VM
Inside the Ubuntu VM:

bash
Copy code
lsblk
Confirm the new disk appears without partitions.

Example:

text
Copy code
sdb    1T
Partitioning Strategy
The disk is partitioned into multiple logical regions to isolate workloads:

Partition	Mount Point	Purpose
Part 1	/mnt/plexdata	Plex metadata
Part 2	/mnt/plextranscode	Transcoding / temp IO
Part 3	/mnt/projects	App configs

This separation prevents noisy workloads from impacting critical metadata.

Format Filesystems
Each partition is formatted individually (example using ext4):

bash
Copy code
mkfs.ext4 /dev/sdb1
mkfs.ext4 /dev/sdb2
mkfs.ext4 /dev/sdb3
Filesystem choice is conservative and prioritizes reliability.

Mount Using UUIDs
Retrieve UUIDs:

bash
Copy code
blkid
Add entries to /etc/fstab:

ini
Copy code
UUID=<uuid-plexdata>     /mnt/plexdata        ext4  defaults,noatime  0 2
UUID=<uuid-transcode>    /mnt/plextranscode   ext4  defaults,noatime  0 2
UUID=<uuid-projects>     /mnt/projects        ext4  defaults,noatime  0 2
Create mount points if needed:

bash
Copy code
mkdir -p /mnt/plexdata /mnt/plextranscode /mnt/projects
Mount all:

bash
Copy code
mount -a
Validation Checklist
Disk appears in VM as expected

All partitions mount correctly

Mounts persist across reboot

No references to /dev/sdX

Ownership set for Docker user (PUID=1000)

Common Mistakes to Avoid
❌ Using /dev/sdX instead of /dev/disk/by-id

❌ Storing media files on the VM disk

❌ Mixing transcode and metadata workloads

❌ Forgetting _netdev (for NAS mounts, not SSD)

Final Takeaway
Raw disk passthrough turns the VM into a consumer of storage rather than an owner of it.

This makes the VM disposable, the data durable, and rebuilds predictable.

yaml
Copy code

---

# 🟢 STEP 3: Commit the File

Scroll down → **Commit changes**

Done.

---

## Where This Leaves You

You’ve now fully documented:
- ✔️ Filesystem layout
- ✔️ Docker stack
- ✔️ Raw disk passthrough (the scary part)

You’re building this in the **exact right order**.

---

## Next (Per Recommendation)

👉 **1️⃣ `02-proxmox-setup.md`**  
This will cover:
- IOMMU
- VFIO
- GPU passthrough
- NVIDIA drivers
orially

When ready, just say:

> “Let’s do 02.”

You’re on rails now — keep going.
