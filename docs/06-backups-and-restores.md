# Backups & Restores

This document describes how application data for the media stack is backed up and restored. The strategy prioritizes **recoverability**, **simplicity**, and **VM disposability**.

The guiding assumption is that:
> The VM can be rebuilt at any time. The data cannot.

---

## What Needs to Be Backed Up

Only **application state** needs protection.

### Included in Backups

| Path | Reason |
|----|------|
| `/mnt/plexdata` | Plex metadata, watch history, settings |
| `/mnt/projects` | App configs for all containers |
| Selected Jellyfin paths | Config and cache (if desired) |

---

### Explicitly Excluded

| Path | Reason |
|----|------|
| `/mnt/media` | Lives on NAS (separate backup policy) |
| `/mnt/downloads` | Ephemeral |
| `/mnt/plextranscode` | Temporary data |
| VM disk | Fully disposable |

---

## Backup Tool Choice: Restic

Restic was chosen because it provides:

- Encrypted backups by default
- Deduplication
- Snapshot-style history
- Easy restores to arbitrary paths
- Support for local or remote repositories

Backups are performed **inside the VM**, not at the Proxmox layer.

---

## Restic Deployment Model

Restic runs as a **Docker container** with access to:

- Application data paths
- A backup repository (local or remote)
- A credentials file or environment variables

This keeps backup logic versioned alongside the stack.

---

## Backup Scope

Example backup targets:

```text
/mnt/plexdata
/mnt/projects
These paths are sufficient to restore the full application layer.

Backup Frequency
Recommended schedule:

Daily incremental backups

Manual backup before major changes

Manual backup before upgrades

Restic’s deduplication makes frequent backups inexpensive.

Restore Scenarios
Scenario 1: App Misconfiguration
Stop affected container

Restore specific app config directory

Restart container

No other services are impacted.

Scenario 2: VM Loss / Rebuild
Reinstall Proxmox (if needed)

Recreate Ubuntu VM

Reattach raw SSD

Mount partitions

Reinstall Docker

Deploy containers (empty configs)

Restore:

/mnt/plexdata

/mnt/projects

Start Docker stack

The system returns to its previous state.

Scenario 3: Partial Restore
Restic allows restoring:

Single files

Single directories

Point-in-time snapshots

This is useful for:

Accidental config changes

App-level corruption

Testing rollback scenarios

Why Not Proxmox Backups?
Proxmox backups and snapshots are intentionally not relied upon because:

They encourage VM-level coupling

They hide application state

They complicate selective restores

They give a false sense of safety

Proxmox snapshots are treated as temporary safety nets, not backups.

Validation Checklist
Backup repository initialized

Backups complete without errors

Restore test performed at least once

Backup credentials stored securely

No sensitive data committed to GitHub

Final Takeaway
Backups are only useful if restores are boring.

This strategy ensures:

VMs are disposable

Data is portable

Recovery is predictable

Mistakes are survivable
