# Troubleshooting & Lessons Learned

This section documents real issues encountered while building this stack, along with their root causes and fixes. These are the kinds of problems that don’t show up in clean tutorials but matter in real deployments.

---

## Plex Server Not Discoverable on Apple TV

### Symptoms
- Plex server shows as online in the web UI
- Apple TV Plex app cannot see or connect to the server
- Server appears as “offline” or not listed at all

### Root Cause
Apple TV cached an old Plex server identity and could not resolve the correct address after the server was rebuilt.

Additionally, Plex was not advertising a reachable address for LAN clients.

### Fix
1. In Plex Web UI:
   - Settings → Network
   - Set **Custom server access URLs**:
     ```
     http://<VM-IP>:32400
     ```
2. Restart the Plex container
3. On Apple TV:
   - Remove the offline server entry
   - Relaunch the Plex app

### Lesson
Apple TV Plex discovery is fragile after rebuilds. Always explicitly set the server access URL when running Plex behind Docker or after reinstallation.

---

## SABnzbd Unreachable by Radarr / Sonarr

### Symptoms
- Radarr/Sonarr report SABnzbd as unreachable
- Web UI works locally but API calls fail
- SAB logs show binding to IPv6 (`::`) only

### Root Cause
The LinuxServer.io SABnzbd image forces IPv6 binding during initialization, overriding user configuration in some cases.

### Fix
- Switched from `linuxserver/sabnzbd` to:
- Confirmed binding to IPv4:

### Lesson
Container init behavior matters. If a service ignores its own config, try a different image before wasting time debugging the app itself.

---

## Radarr Root Folder Exists but Cannot Be Selected

### Symptoms
- `/media/Movies` exists inside the container
- Radarr UI cannot browse or select the folder
- Imports fail with root folder errors

### Root Cause
Radarr UI file browser intermittently fails with certain bind-mounted paths.

### Fix
Created a symlink inside the container:
/movies -> /media/Movies

Radarr was able to use the symlink without issue.

### Lesson
If the *arr UI refuses a valid path, try a symlink before assuming permissions or mount issues.

---

## Jellyfin Admin Login Lockout

### Symptoms
- Admin login fails
- Password reset attempts ineffective
- Config directory partially corrupted

### Root Cause
Repeated password reset attempts left Jellyfin config in an inconsistent state.

### Fix
- Stopped Jellyfin container
- Fully wiped config, cache, and transcode directories
- Recreated Jellyfin container cleanly

### Lesson
For stateless apps like Jellyfin, a full reset is often faster and safer than trying to repair corrupted auth state.

---

## Snapshot Overconfidence (Proxmox)

### Symptoms
- Assumed a “golden snapshot” existed
- Snapshot had already been deleted
- No quick rollback available

### Root Cause
Snapshots were treated as backups.

### Fix
- Accepted loss
- Introduced Restic-based backups for app data

### Lesson
Snapshots are **not backups**. Anything important should exist outside Proxmox’s snapshot system.

---

## Docker Volumes vs Bind Mounts

### Lesson
Bind mounts:
- Make backups trivial
- Make restores predictable
- Make documentation clearer

Opaque Docker volumes hide critical state and complicate disaster recovery.

---

## Final Takeaway

This stack is stable *because* of the mistakes made while building it. Every fix here exists to prevent repeating those mistakes during the next rebuild.
