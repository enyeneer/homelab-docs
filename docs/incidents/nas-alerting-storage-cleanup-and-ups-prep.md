# NAS Alerting, Storage Cleanup, and UPS Prep

This page documents a homelab reliability phase focused on getting real storage alerting in place, cleaning up failing legacy drives and pools, and preparing the environment for better power protection and recovery.

## Summary

The goal of this phase was to make the NAS actually capable of warning when something was failing, while also cleaning up old risky storage and laying the groundwork for UPS-backed reliability.

The work focused on:

- setting up outbound email alerting through Gmail relay
- enabling SMART monitoring and scheduled disk self-tests
- enabling ZFS event email alerts through ZED
- enabling regular ZFS scrub scheduling
- confirming mdraid monitoring
- identifying and removing failing legacy drives and an old experimental ZFS pool
- preparing for later UPS and NUT integration

## Main Goal

Set up reliable "something is dying" alerting for the NAS through:

- SMART monitoring
- ZFS event notifications
- mdraid monitoring
- email delivery to a real inbox

At the same time, remove unsafe old storage components and improve the overall reliability posture before adding UPS support.

## Environment

### Main Server
- Hostname: `seranogenomics`
- Role: Docker/YAMS host for media and related services
- State at the time: no UPS, some intermittent app/network weirdness, Docker stack active

### NAS
- Hostname: `nas`
- Platform: Ubuntu 24.04
- Storage layout:
  - OS on mdraid RAID1 SSDs under LUKS/LVM
  - ZFS `tank` as the real storage pool
  - legacy experimental `mediapool` made from older miscellaneous drives
- State at the time: some old disks were failing, and outbound alerting was not fully in place

### Monitoring / Alerting Goal
The environment needed a reliable way to surface disk, array, and filesystem problems before they turned into silent data loss or confusing downtime.

## Starting State

At the start of this phase:

- outbound email alerting was not fully configured
- SMART monitoring was not yet fully set up to notify a real inbox
- ZFS event notifications were not fully wired up
- monthly ZFS scrubs were present but not enabled
- the NAS still had an old degraded/corrupted pool (`mediapool`) built from junk drives
- one of the old drives was critically failing
- UPS support had not yet been implemented
- a brief attempt to use Uptime Kuma for service monitoring did not become a stable long-term solution

## Problems Addressed

This work covered several separate but related reliability problems:

- no dependable storage alert path to a real inbox
- need for scheduled SMART self-tests
- need for ZFS event email alerting
- need for regular ZFS scrub scheduling
- critically failing HDD in the old storage pile
- degraded/corrupted legacy ZFS pool
- unnecessary temperature and reliability risk from leaving junk drives installed
- desire to prepare for a future UPS/NUT setup

## Changes Made

## 1. Postfix Gmail Relay and Root Mail Routing

Configured local mail delivery so storage alerts would reach a real inbox.

### Changes
- installed and configured Postfix
- configured relay through Gmail SMTP using port `587` and STARTTLS
- used a Gmail app password for relay authentication
- set `/etc/aliases` so mail for `root` forwarded to a personal Gmail inbox
- confirmed test delivery worked

### Result
The NAS could now send real alert emails instead of leaving notifications trapped locally.

## 2. SMART Monitoring and Scheduled Disk Self-Tests

Configured SMART monitoring to notify `root`, which then forwarded to Gmail.

### Changes
- enabled `smartd`
- configured scheduled SMART tests:
  - daily short tests
  - weekly long tests
- used a `smartd.conf` line that both monitored health and ran scheduled tests

### Result
Disk problems could now generate actionable email alerts, and routine self-tests were scheduled automatically.

## 3. ZFS ZED Email Alerts

Enabled ZFS event delivery through `zfs-zed`.

### Changes
Configured:

- `ZED_EMAIL_ADDR="root"`
- `ZED_NOTIFY_VERBOSE=1`

Also ensured `zfs-zed` was installed and running.

### Result
ZFS-related events could now send mail through the same root-forwarding path.

## 4. Monthly ZFS Scrub Scheduling

Enabled the existing monthly scrub timer for the real pool.

### Changes
- enabled `zfs-scrub-monthly@tank.timer`

### Result
Regular scrub scheduling was now active instead of merely available.

## 5. mdraid Monitoring Confirmation

Verified mdraid alerting and monitoring state for the OS RAID1 arrays.

### Changes
- confirmed `/etc/mdadm/mdadm.conf` contained array definitions
- confirmed `MAILADDR root` was present
- confirmed `mdmonitor.service` was active

### Result
mdraid monitoring was confirmed to be wired into the same root mail alert path.

## 6. Storage Cleanup and Legacy Pool Removal

Removed failing and unnecessary legacy storage components.

### Changes
- identified critically failing disk `/dev/sdg`
- confirmed the old `mediapool` was degraded/corrupted and no longer worth keeping
- destroyed `mediapool`
- physically removed failing and unused old drives
- verified `tank` remained healthy and was the only remaining ZFS pool

### Result
The NAS was left with a much cleaner and safer storage layout centered only around the real pool.

## 7. UPS Planning and Reliability Prep

Began planning for power protection and coordinated shutdown behavior.

### Direction
The intended follow-up was:

- NAS as the UPS/NUT "brain" over USB
- main server as NUT client
- optional future OPNsense integration

### Result
UPS integration was not yet completed in this phase, but the design direction was established.

## What Worked

The following improvements were successful:

- Postfix relay worked and delivered test emails
- SMART alerts worked in a real, useful way
- a critically failing old drive was correctly surfaced
- ZFS ZED was configured and running
- monthly scrub scheduling for `tank` was enabled
- mdraid monitoring was confirmed
- legacy junk storage was removed without affecting the real pool
- drive temperatures improved after cleanup, settling around the mid-30°C range
- the NAS ended the phase in a cleaner and more supportable state

## What Did Not Work

A few things were attempted or observed but did not become clean wins:

### Uptime Kuma
A trial of Uptime Kuma for service monitoring ran into UI and socket instability.

Possible contributing factors included:

- cross-subnet access path issues
- intermittent routing path weirdness
- general instability during testing

This was ultimately abandoned.

### Intermittent Jellyfin Disconnect Reports
There were user-reported Jellyfin disconnects after some monitoring/network tinkering.

The root cause was not fully confirmed. Possibilities included:

- client path differences
- subnet/routing path issues
- container restarts
- Docker daemon restarts or blips

### Space Pressure on `/archive`
Even after cleanup, `/archive` remained heavily used at roughly 94%, which is operationally uncomfortable and needs follow-up cleanup.

## Ending State

By the end of this phase:

### NAS: `nas`
- `tank` remained ONLINE
- `tank` was the only remaining ZFS pool
- old `mediapool` was removed
- failing/unused junk drives were removed
- SMART monitoring was configured and scheduled
- ZFS ZED email alerting was configured
- mdraid monitoring was confirmed
- monthly scrub scheduling was enabled
- `/archive` remained heavily utilized and still needed cleanup

### Main Server: `seranogenomics`
- Postfix relay was also configured and tested
- Docker stack remained running
- Uptime Kuma was dropped as a monitoring path

### Reliability Posture
- the environment was in a much better position to generate meaningful disk/storage failure alerts
- UPS planning was ready for the next phase

## Lessons Learned

- alerting only matters if it reaches a real inbox
- SMART, ZFS, and mdraid should all be treated as first-class failure signals
- old junk drives and abandoned experimental pools create unnecessary operational risk
- removing bad hardware can improve both reliability and temperature behavior
- monitoring tools are only worth keeping if the access path is stable and trustworthy
- storage capacity pressure is its own reliability issue and should not be ignored
- getting alerting in place before a UPS event or real disk failure is high-value preventative work

## Follow-Up Items

- configure NUT when the UPS arrives
- use the NAS as the primary UPS controller over USB
- add the main server as a NUT client
- optionally integrate OPNsense later
- free up roughly 1 to 1.5 TB on `/archive`
- if Jellyfin instability continues, verify:
  - Docker daemon restart history
  - Watchtower activity
  - container restart counts
  - whether clients are using a stable access path such as LAN or Tailscale consistently

## Commands and Config Snippets Worth Preserving

### SMART Failure Example

```text
SMART overall-health self-assessment test result: FAILED!
Drive failure expected in less than 24 hours. SAVE ALL DATA.
Reallocated_Sector_Ct ... FAILING_NOW 2063
Current_Pending_Sector 9
Offline_Uncorrectable 5
```

### Healthy `tank` Status

```bash
zpool status tank
# tank ONLINE, mirror of:
# ata-WDC_WD140EDGZ-11B1PA0_7LGHJZWK
# ata-WDC_WD140EDGZ-11B1PA0_7LGHMK5K
```

### Space Breakdown

```bash
sudo du -xhd 1 /archive | sort -h
# /archive/movies ~5.8T
# /archive/tv     ~6.0T
# /archive/downloads ~48G
# /archive/.Trash-1000 ~3.1G
```

### `smartd.conf` Active Line

```conf
DEVICESCAN -a -o on -S on -n standby,q -s (S/../.././02|L/../../6/03) -m root -M exec /usr/share/smartmontools/smartd-runner
```

### ZED Configuration

```conf
ZED_EMAIL_ADDR="root"
ZED_NOTIFY_VERBOSE=1
```

### Enable Monthly Scrub

```bash
sudo systemctl enable --now zfs-scrub-monthly@tank.timer
systemctl list-timers | grep -i zfs
```

### mdraid Monitoring Checks

```bash
systemctl status mdmonitor --no-pager
cat /proc/mdstat
sudo mdadm --detail /dev/md/bootraid1
sudo mdadm --detail /dev/md/osdriveraid1
```

### Postfix Relay Credential Location

```bash
sudo cat /etc/postfix/sasl_passwd
# contains Gmail app password; keep permissions locked down
```

## Status

Completed, with UPS integration and storage cleanup follow-up still remaining.
