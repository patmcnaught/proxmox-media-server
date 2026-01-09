Media Automation & Transcoding Design
Overview

This document explains the intentional design decisions behind media automation, storage layout, and Plex transcoding for this Proxmox-based media server.

The goal is to balance:

Performance

Reliability

Storage efficiency

Operational simplicity

Storage Architecture
Component	Path	Storage Type	Rationale
OS / Docker	/	Local disk	Isolation from media workloads
Plex metadata	/mnt/plexdata	SSD	Fast DB + thumbnail access
Plex transcodes	/mnt/plextranscode	SSD	High I/O, avoids NAS thrash
Downloads	/mnt/downloads	NAS (NFS)	Large, temporary data
Movies	/mnt/media/Movies	NAS (NFS)	Long-term storage
TV Shows	/mnt/media/TV	NAS (NFS)	Long-term storage

SSD is reserved for latency-sensitive workloads, while the NAS handles capacity.

Hardlinks vs Atomic Moves

Hardlinks are intentionally disabled.

Although downloads and media live on the same NAS volume, Docker exposes them as separate mounts, making hardlinks unreliable.

Instead:

Atomic moves are enabled

Files are renamed rather than copied

No duplicate storage is created

This ensures predictable imports without filesystem edge cases.

SABnzbd Configuration

Incomplete downloads: /mnt/downloads/incomplete

Completed downloads: /mnt/downloads/complete

Key settings:

Pause downloading during post-processing: Enabled

Direct unpack: Disabled

Folder rename: Enabled

This avoids race conditions during Radarr/Sonarr imports.

Radarr & Sonarr Media Management

Key settings:

Use Hardlinks instead of Copy: Disabled

Atomic Moves: Enabled

Analyze video files: Disabled

Rescan media folder after refresh: Disabled

These settings minimize I/O and prevent unnecessary rescans.

Quality Profile Philosophy

The system prioritizes consistent quality over maximum bitrate.

Preferred formats:

WEBDL 1080p

Bluray 1080p

Selective 2160p with size limits

De-prioritized or disabled:

REMUX

HDTV

CAM / TS

This reduces storage growth while maintaining high visual quality.

Plex Transcoding Strategy

Plex is configured to use:

Dedicated SSD transcode directory: /mnt/plextranscode

Hardware-accelerated transcoding (GPU)

Limited concurrent transcodes (2–3)

Transcode throttling enabled

This prevents:

Network I/O saturation

NAS performance degradation

SSD exhaustion

Observability

The stack includes:

Tautulli for playback and transcode visibility

Prometheus + Grafana for disk and container monitoring

Alerts for disk usage and transcode activity

This enables proactive detection of issues.

Design Tradeoffs
Decision	Tradeoff
NAS downloads	Slower unpack vs SSD safety
No hardlinks	More storage used vs reliability
Transcode caps	Fewer concurrent streams vs stability

All tradeoffs are intentional and documented.

Future Enhancements

Automated cleanup policies for failed downloads

Per-user Plex quality enforcement

Storage tiering based on access frequency
