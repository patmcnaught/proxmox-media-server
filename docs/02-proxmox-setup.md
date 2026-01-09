# Proxmox Setup

This document describes the Proxmox VE configuration used to host the media server VM, including GPU passthrough, IOMMU configuration, and VM design decisions.

The goal is a **stable hypervisor** that can survive VM rebuilds without rework.

---

## Host Overview

- Hypervisor: Proxmox VE
- Boot disk: NVMe SSD
- Storage backend: LVM-thin (`local-lvm`)
- GPU: NVIDIA (dedicated passthrough)
- VM OS: Ubuntu Server

Proxmox is treated strictly as an orchestration layer — not a place to store application data.

---

## BIOS / Firmware Prerequisites

Before installing Proxmox, ensure the following are enabled in BIOS:

- VT-d / IOMMU (Intel) or SVM / IOMMU (AMD)
- Above 4G Decoding
- PCIe ACS (if available)
- Disable CSM / Legacy boot (UEFI only)

Without these, GPU passthrough will not function reliably.

---

## Enable IOMMU in Proxmox

### Edit GRUB

On the Proxmox host:

```bash
nano /etc/default/grub
For Intel CPUs:

ini
Copy code
GRUB_CMDLINE_LINUX_DEFAULT="quiet intel_iommu=on iommu=pt"
For AMD CPUs:

ini
Copy code
GRUB_CMDLINE_LINUX_DEFAULT="quiet amd_iommu=on iommu=pt"
Apply changes:

bash
Copy code
update-grub
reboot
Verify IOMMU
After reboot:

bash
Copy code
dmesg | grep -e IOMMU -e DMAR
You should see messages indicating IOMMU is enabled.

Configure VFIO Modules
Create or edit:

bash
Copy code
nano /etc/modules
Ensure the following are present:

nginx
Copy code
vfio
vfio_iommu_type1
vfio_pci
vfio_virqfd
Isolate the GPU from Proxmox
Identify the GPU and its associated devices:

bash
Copy code
lspci -nn
Example output:

makefile
Copy code
01:00.0 VGA compatible controller [0300]: NVIDIA Corporation ...
01:00.1 Audio device [0403]: NVIDIA Corporation ...
Record the device IDs (e.g. 10de:2206).

Bind GPU to VFIO
Create:

bash
Copy code
nano /etc/modprobe.d/vfio.conf
Add:

nginx
Copy code
options vfio-pci ids=10de:XXXX,10de:YYYY
Replace with your GPU and audio device IDs.

Prevent Host Driver Loading
Create:

bash
Copy code
nano /etc/modprobe.d/blacklist-nvidia.conf
Add:

nginx
Copy code
blacklist nouveau
blacklist nvidia
Update initramfs:

bash
Copy code
update-initramfs -u
reboot
Verify GPU Is Bound to VFIO
After reboot:

bash
Copy code
lspci -nnk -d 10de:
You should see:

css
Copy code
Kernel driver in use: vfio-pci
If Proxmox is using the GPU, passthrough will not work.

VM Configuration
VM Settings (Key Points)
Machine type: q35

BIOS: OVMF (UEFI)

CPU: Host

Memory: Ballooning disabled

NUMA: Enabled (optional, but recommended)

Add GPU to VM
In Proxmox UI:

VM → Hardware

Add → PCI Device

Select GPU

Check:

✅ All Functions

✅ Primary GPU

✅ ROM-Bar (if needed)

Repeat for the GPU audio device if listed separately.

NVIDIA Drivers Inside the VM
Inside the Ubuntu VM:

bash
Copy code
sudo apt update
sudo apt install nvidia-driver-XXX
reboot
Verify:

bash
Copy code
nvidia-smi
You should see the GPU listed and idle.

Docker GPU Support
Install NVIDIA Container Toolkit:

bash
Copy code
distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg
curl -fsSL https://nvidia.github.io/libnvidia-container/$distribution/libnvidia-container.list | sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list
sudo apt update
sudo apt install -y nvidia-container-toolkit
sudo systemctl restart docker
Verify Docker sees the GPU:

bash
Copy code
docker run --rm --gpus all nvidia/cuda:12.0-base nvidia-smi
Snapshot Guidance (Important)
Snapshots are useful for:

Short-term testing

Configuration experiments

Snapshots are not backups.

Anything that matters must exist:

On passthrough disks

On NAS

Or in external backups

Final Takeaway
Proxmox should disappear once configured.

If GPU passthrough works and disks attach cleanly, the hypervisor is doing its job. Everything else belongs inside the VM.
