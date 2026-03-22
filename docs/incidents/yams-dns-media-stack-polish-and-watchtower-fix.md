# YAMS, DNS, Media Stack Polish, and Watchtower Fix

This page documents a homelab cleanup and optimization phase focused on DNS sanity, media-stack quality control, subtitle automation, TV client experience, improved search, and restoring a safe automatic update path for containers.

## Summary

The goal of this phase was to make the post-migration homelab feel less messy and more polished, especially around DNS behavior and the self-hosted media stack.

The work centered on:

- cleaning up obvious DNS misconfiguration in OPNsense, AdGuard Home, and Unbound
- improving request and media-quality behavior for Radarr and Sonarr
- making subtitle automation more reliable through Bazarr
- evaluating better Jellyfin TV clients
- improving Jellyfin search quality with Meilisearch
- adding lightweight request notifications through Seerr
- restoring Watchtower to a sane scheduled-update workflow

## Main Goal

Clean up and optimize the homelab after the major network migration, especially by:

- verifying and fixing DNS weirdness
- improving the YAMS and Jellyfin stack
- improving request, search, and subtitle workflows
- evaluating a better TV client experience
- restoring a trustworthy automatic update path for containers

## Environment

### Core Network
- OPNsense / `texhnolyze.home.arpa` at `192.168.5.1`
- AdGuard Home + Unbound in place for DNS
- OpenVPN working well enough for LAN and internet access
- legacy `192.168.7.x` / `192.168.8.x` cruft still affecting some DHCP and DNS behavior

### Main Server
- Hostname: `seranogenomics`
- IP: `192.168.5.7`
- Role: main Docker/YAMS host

### NAS
- Hostname: `nas`
- IP: `192.168.5.8`

### Media Stack
- Jellyfin
- Jellyseerr
- Radarr
- Sonarr
- Bazarr
- Prowlarr
- qBittorrent
- Gluetun

### Starting Pain Points
- DNS configuration looked suspicious and inconsistent
- remote users were triggering bad media grabs
- subtitle handling was too manual
- Jellyfin TV UX felt too plain
- search quality in Jellyfin and Seerr was mediocre
- Watchtower auto-updates were not clearly working

## Starting State

At the beginning of this phase:

- the homelab was already largely functional on the new `192.168.5.x` LAN
- OpenVPN was working well enough for both LAN and internet access
- the YAMS stack was generally running
- AdGuard Home + Unbound were in place, but DNS design and configuration were still messy
- stale legacy 7.x and 8.x entries still influenced parts of DNS and DHCP
- the media stack worked, but quality control and automation still felt hacky
- Watchtower existed, but automatic updates were not clearly healthy

## Problems Addressed

This work covered several connected cleanup and quality-of-life problems:

- stale and suspicious DNS settings in OPNsense and local DNS components
- awkward router hostname access from VPN clients
- poor or undesirable remote media grabs
- subtitle automation/provider issues
- weak Jellyfin TV client experience
- mediocre search behavior in Jellyfin
- lack of lightweight notifications for Seerr request flow
- broken Watchtower auto-update behavior due to Docker API mismatch

## Changes Made

## 1. Cleaned Up OPNsense, AdGuard, and Unbound DNS Behavior

Removed the most obviously broken or stale DNS behavior.

### Changes
- removed stale system DNS entries such as:
  - old `192.168.6.1`
  - old `192.168.7.1`
  - old `192.168.8.1`
- removed broken Unbound query-forwarding and self-forwarding behavior
- set AdGuard Home as the primary DNS service in OPNsense
- corrected stale AdGuard DNS rewrites for local hostnames

### Result
DNS behavior became much cleaner and less self-referential without requiring a full redesign.

## 2. Accepted Direct IP Access for Router over VPN

Looked at hostname access for the router over OpenVPN and chose the simpler path.

### Issue
`texhnolyze.home.arpa` over OpenVPN was not worth fully solving.

### Result
Direct IP access to `192.168.5.1` was accepted as the practical solution for router access over VPN.

## 3. Installed and Configured Recyclarr

Added real quality-profile automation for Radarr and Sonarr.

### Changes
- added a Recyclarr container to YAMS
- generated and synced:
  - Radarr: `HD Bluray + WEB`
  - Sonarr: `WEB-1080p`
- enabled **Bad Dual Groups**
- removed old manual release-profile hacks such as:
  - `YTS`
  - `QxR`

### Result
Radarr and Sonarr profile handling became much cleaner, more reproducible, and less dependent on manual hacks.

## 4. Repaired Bazarr Subtitle Automation

Confirmed existing Bazarr integrations and improved provider behavior.

### Changes
- confirmed Sonarr and Radarr integrations were already live
- repaired subtitle provider configuration
- standardized around plain English subtitles rather than hearing-impaired or SDH captions

### Result
Subtitle automation became much more viable and less annoying to manage manually.

## 5. Evaluated TV Client Options

Compared alternative Jellyfin TV client approaches.

### Kodi
Kodi was tested and rejected.

### Why It Was Rejected
- did not provide the expected out-of-box “Netflix-like” experience
- required more effort than it was worth for the resulting gains

### Moonfin
Moonfin was installed and tested on Android TV.

### Result
Moonfin was clearly preferred over the stock Jellyfin TV app.

## 6. Improved Jellyfin Search with Meilisearch

Added a dedicated Meilisearch instance for Jellyfin.

### Changes
- installed and configured a dedicated Meilisearch instance
- set up the Jellyfin Meilisearch plugin
- verified service health and improved search behavior

### Result
Jellyfin search improved noticeably and felt more capable than the default setup.

## 7. Added ntfy Notifications to Seerr

Set up lightweight notifications for request workflow visibility.

### Changes
- configured `ntfy` notifications in Seerr for pending requests

### Result
Pending requests could now generate simple phone notifications without adding a heavier notification stack.

## 8. Fixed Watchtower and Restored Scheduled Updates

Watchtower was initially broken because of a Docker API mismatch.

### Observed Failure
Watchtower was failing with:

```text
client version 1.25 is too old. Minimum supported API version is 1.44
```

### Changes
- resolved the Docker API mismatch
- scheduled weekly Sunday 4 AM update checks
- removed rolling restart because of the `qbittorrent` / `gluetun` dependency warning

### Result
Watchtower returned to a sane, scheduled maintenance role instead of being broken or overly aggressive.

## What Worked

The following improvements were successful:

- DNS cleanup reduced obvious misconfiguration without requiring a full redesign
- Recyclarr synced successfully and created:
  - 37 Radarr custom formats
  - 34 Sonarr custom formats
  - updated profiles and quality definitions
- removing old manual release-profile hacks simplified the stack in a good way
- Bazarr providers were repaired and subtitle automation became much more usable
- Moonfin on Android TV was clearly better than the stock Jellyfin TV app
- Jellyfin Meilisearch improved search noticeably
- Seerr `ntfy` notifications worked well as a lightweight request-notification path
- Watchtower was fixed and scheduled for weekly Sunday 4 AM update checks

## What Did Not Work

A few areas were intentionally left unresolved or were rejected after testing.

### Router Hostname over VPN
`texhnolyze.home.arpa` over OpenVPN was not worth fully solving, and direct IP access remained the practical answer.

### Kodi
Kodi did not provide the expected out-of-box experience and was abandoned.

### Moonfin Extras / Featurettes
Moonfin on Android TV still did not reliably expose extras and featurettes the same way Jellyfin desktop or web did.

### File Transformation / Moonfin Web UI Injection
This was intentionally skipped because it looked risky and potentially unstable.

### Watchtower Rolling Restart
Rolling restart caused an incompatibility warning because `qbittorrent` depends on `gluetun`, so it was not kept.

## Validation

### Container State Check

```bash
docker ps --format 'table {{.Names}}\t{{.Image}}\t{{.Status}}\t{{.Ports}}'
```

Used to confirm active container state after cleanup and changes.

### Recyclarr Validation

```bash
docker exec recyclarr recyclarr config list templates
docker exec recyclarr recyclarr config create -t hd-bluray-web
docker exec recyclarr recyclarr config create -t web-1080p
docker exec recyclarr recyclarr sync --log info
```

Also verified key config details:

```bash
grep -nE 'base_url|api_key|delete_old_custom_formats|Bad Dual Groups' /opt/yams/config/recyclarr/configs/*.yml
```

### Meilisearch Health Check

```bash
curl http://192.168.5.7:7700/health
```

Expected result:

```json
{"status":"available"}
```

### Watchtower Diagnostics

```bash
docker logs --tail 100 watchtower
docker version --format '{{.Server.APIVersion}}'
```

Healthy final behavior included output like:

```text
Watchtower 1.7.1
Checking all containers (except explicitly disabled with label)
Scheduling first run: 2026-03-15 04:00:00 -0400 EDT
```

### YAMS Recovery / General Service Refresh

```bash
yams restart
```

Used as the standard restart/revalidation path after stack-side changes.

## Ending State

By the end of this phase:

### Core Homelab
- the homelab remained functional on the new `192.168.5.x` layout
- DNS was cleaner and no longer had the most obviously broken self-referential behavior
- `HomeVPN` interface remained disabled, but VPN still worked through the existing OpenVPN rules path

### Media Stack
- Recyclarr was installed and actively managing Radarr and Sonarr profiles/custom formats
- old `YTS` and `QxR` release-profile hacks were removed
- Bazarr was usable for automated subtitle handling with repaired providers
- Moonfin was the preferred Android TV client over stock Jellyfin
- Jellyfin search was improved with a dedicated Meilisearch backend
- Seerr had `ntfy` notifications for pending requests

### Updates and Maintenance
- Watchtower was fixed
- weekly Sunday 4 AM auto-updates were configured
- the media stack felt much more polished and much less hacky than at the start

## Lessons Learned

- DNS cleanup often produces major clarity gains even before a full redesign
- old release-profile hacks become technical debt once better automation exists
- quality control for media grabs is worth systematizing instead of tweaking ad hoc
- subtitle automation is only as good as its providers and target language strategy
- the best TV client is not necessarily the most famous or most customizable one
- low-risk improvements like Meilisearch and `ntfy` can noticeably improve daily usability
- automatic updates should be scheduled conservatively and should respect service dependencies
- not every annoying hostname issue is worth solving if a direct IP is cleaner

## Follow-Up Items

- observe real-world Recyclarr behavior before adding custom movie-size overrides or reintroducing favorite release-group preferences
- observe Bazarr subtitle behavior over time and confirm providers remain stable
- possibly improve Jellyfin curation and discovery in low-risk ways
- continue monitoring Moonfin as the preferred TV client and decide later whether to roll it out more broadly
- keep using direct IP for router access over VPN unless hostname access becomes worth revisiting
- confirm Watchtower actually performs updates successfully on its next scheduled Sunday 4 AM run
- continue long-term cleanup of stale DHCP, DNS, and router cruft only if it causes real issues
- eventually revisit older background items such as:
  - preboot/dropbear networking cleanup
  - stale legacy mailserver mapping
  - NAS CMOS/BIOS persistence behavior

## Commands Worth Preserving

### Check Active Containers

```bash
docker ps --format 'table {{.Names}}\t{{.Image}}\t{{.Status}}\t{{.Ports}}'
```

### Recyclarr Setup and Sync

```bash
docker exec recyclarr recyclarr config list templates
docker exec recyclarr recyclarr config create -t hd-bluray-web
docker exec recyclarr recyclarr config create -t web-1080p
docker exec recyclarr recyclarr sync --log info
```

### Recyclarr Config Verification

```bash
grep -nE 'base_url|api_key|delete_old_custom_formats|Bad Dual Groups' /opt/yams/config/recyclarr/configs/*.yml
```

### Meilisearch Health Check

```bash
curl http://192.168.5.7:7700/health
```

### Watchtower Debug

```bash
docker logs --tail 100 watchtower
docker version --format '{{.Server.APIVersion}}'
```

### Watchtower Final Environment Idea

```text
WATCHTOWER_CLEANUP=true
WATCHTOWER_SCHEDULE=0 0 4 * * 0
DOCKER_API_VERSION=1.54
TZ=${TZ}
```

### Standard YAMS Restart Flow

```bash
yams restart
```

## Status

Completed as a cleanup and optimization phase, with monitoring and low-risk polish follow-up still remaining.
