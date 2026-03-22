# Remote LUKS Unlock, Tailscale Hardening, and NAS/Server Reliability

This page documents a major homelab hardening and reliability phase focused on remote reboot recovery, encrypted boot workflows, Tailscale access control, mount resiliency, and reducing manual post-boot intervention.

## Summary

The goal of this phase was to make the homelab more resilient and more remotely manageable after reboots and power events.

The work centered on four major areas:

- enabling reliable remote unlock for LUKS-encrypted systems
- tightening Tailscale access so shared users could only reach specific media services
- preventing mount and networking failures from cascading into Docker and service startup problems
- reducing unnecessary GUI, sleep, and timeout behavior on infrastructure hosts

## Main Goal

Make the homelab resilient and remotely manageable after reboots and power events by:

- enabling reliable remote reboots for both the NAS and main server despite LUKS encryption
- hardening Tailscale access so friends and family could only reach Jellyfin and Seerr
- reducing boot and mount failures that could break Docker or YAMS
- removing unnecessary GUI components and preventing sleep or timeout behavior
- creating simple one-click Windows scripts to unlock systems remotely

## Environment

### Main Server
- Hostname: `seranogenomics`
- Platform: Lenovo ThinkCentre
- Role: main Linux server running Docker and YAMS
- State at start: LUKS-encrypted root, LightDM GUI present, occasional sleep/timeout and reboot issues

### NAS
- Hostname: `nas`
- Platform: Ubuntu 24.04
- Storage layout:
  - LUKS
  - encrypted ZFS pool: `mediapool`
  - unencrypted ZFS storage: `tank/archive`
- State at start: required local console steps after reboot, including dm-crypt passphrase entry and manual ZFS key loading

### Tailscale
- State at start:
  - overly permissive default policy
  - confusion around device sharing vs full membership
  - need to restrict shared users to only media services

### Mounting and Dependencies
- `seranogenomics` mounted NAS storage at `/mnt/nas-archive` via CIFS
- mount failures could cascade into failed systemd units and break Docker startup

### Reliability Context
- no UPS in place at the time
- occasional apartment power outages made remote recovery more important

## Starting Problems

At the beginning of this work, the environment had several operational weak points:

- LUKS encryption blocked normal SSH access after reboot until a passphrase was entered locally
- Docker startup could fail because of dependency failures tied to networking or NAS mounts
- Tailscale access was broader than intended
- the main server still had GUI and sleep-related behavior that did not fit a headless server role
- the NAS required too many manual steps after reboot
- remote management after power events was not clean enough

## Key Changes Made

## 1. Remote LUKS Unlock for NAS and Main Server

Installed and configured `dropbear-initramfs` so both systems could be unlocked remotely during early boot.

### Changes
- installed `dropbear-initramfs`
- added Windows SSH public key to `/etc/dropbear/initramfs/authorized_keys`
- configured preboot SSH on port `2222`
- ensured initramfs networking used DHCP

### Result
Both machines could be reached during preboot, and `cryptroot-unlock` could be used remotely to continue boot.

## 2. Windows Unlock Tooling

Created Windows-side scripts to make remote unlock easier and more repeatable.

### Changes
Created a PowerShell workflow that could:
- wait for dropbear on port `2222`
- run `cryptroot-unlock`
- wait for normal SSH to return

Also created click-to-run launcher files:
- `unlock-nas.cmd`
- `unlock-server.cmd`

### Result
Remote unlock became close to a one-click process instead of a fragile manual sequence.

## 3. Tailscale ACL Hardening

Reworked Tailscale policy to avoid broad access for shared users.

### Changes
- replaced permissive access with tag-based access control
- created `tag:media`
- added `tagOwners`
- restricted `autogroup:member` and `autogroup:shared` access to:
  - `tag:media:8096` for Jellyfin
  - `tag:media:5055` for Seerr

### Result
Friends and family could access the intended media services without being able to reach the rest of the homelab.

## 4. Hardware Transcoding Readiness

Confirmed Jellyfin hardware transcoding support on the main server.

### Checks
- verified `/dev/dri` existed
- confirmed Jellyfin ffmpeg had `vaapi` and `qsv` support available

### Result
The environment was confirmed to be capable of Intel iGPU-based hardware transcoding.

## 5. Preventing Sleep and Timeout Behavior

Disabled sleep-related behavior on infrastructure systems that should remain headless and always available.

### Changes
Masked:
- `sleep.target`
- `suspend.target`
- `hibernate.target`
- `hybrid-sleep.target`

Also adjusted logind behavior to ignore idle and suspend actions.

### Result
The NAS and main server were less likely to suspend or enter unwanted low-power states.

## 6. Main Server Networking Cleanup

Simplified the main server’s network stack to reduce boot problems.

### Changes
- disabled NetworkManager
- kept `systemd-networkd`
- disabled and masked `systemd-networkd-wait-online`

### Result
This removed a major source of boot-time dependency issues.

## 7. Cockpit for Headless VM Management

Moved toward a cleaner headless-server workflow.

### Changes
- installed `cockpit`
- installed `cockpit-machines`
- disabled LightDM on the main server

### Result
The main server became more appropriate for headless administration while still keeping browser-based VM management available.

## 8. CIFS Mount Hardening on Main Server

Made the NAS mount non-fatal so mount failures would not break boot or Docker startup.

### Changes
Updated `/etc/fstab` for `/mnt/nas-archive` to use:
- `nofail`
- `_netdev`
- `x-systemd.automount`
- `x-systemd.idle-timeout=60`

### Result
The mount became more resilient, and NAS availability no longer had to determine whether the main server booted cleanly.

## 9. NAS ZFS Auto Key Load and Mount

Reduced manual post-unlock steps on the NAS.

### Changes
Because the vendor `zfs-load-key.service` was masked, a custom unit was created:
- `zfs-load-keys-and-mount.service`

This unit:
- loaded the key for `mediapool`
- mounted ZFS datasets automatically

### Result
After LUKS unlock, the NAS could bring up encrypted datasets and `/archive` without requiring manual ZFS commands.

## 10. Key Material Backup

Created and copied off an encrypted bundle of important NAS recovery material.

### Changes
- created encrypted archive: `nas-key-material_2026-03-01.tar.gz.gpg`
- copied it off the NAS to Windows
- set strict permissions on the on-host copy

### Result
Recovery material was no longer only stored on the NAS itself.

## What Worked

The following improvements were successful:

- remote preboot unlock worked on both systems
- Windows unlock scripts made the process much easier
- Tailscale ACLs correctly restricted shared access to Jellyfin and Seerr only
- Jellyfin hardware transcoding support was confirmed
- NAS mount behavior on the main server became more resilient
- NAS ZFS key loading and mounting worked automatically after LUKS unlock
- the main server ran stably in a more headless-server style
- encrypted key backup material was successfully created and copied off-box

## What Broke or Regressed

A few problems came up during the work:

### Docker Startup Dependency Failures
`docker.service` initially failed because of dependency issues involving:
- `systemd-networkd-wait-online.service`
- `mnt-nas-archive.mount`

### NAS Export Lock Error
The NAS initially reported:

```text
failed to lock /etc/exports.d/zfs.exports.lock: No such file or directory
````

This was fixed by creating `/etc/exports.d`.

### PowerShell Script Issues

The unlock script initially had:

* variable parsing problems involving `:`
* post-boot SSH banner exchange timeout issues

These were fixed by:

* using safer PowerShell variable syntax
* waiting for fully usable SSH
* running commands in a cleaner single-session flow

### SCP Permission Issue

Copying recovery material directly from `/root` failed because of permissions, so a temporary copy to `/home/zach` was used for transfer.

## Ending State

By the end of this phase:

### Main Server: `seranogenomics`

* headless-oriented setup
* LightDM disabled
* stable networking with `systemd-networkd`
* Cockpit available for VM management
* CIFS mount at `/mnt/nas-archive` hardened with automount and `nofail`
* remote LUKS unlock via dropbear on port `2222` working

### NAS: `nas`

* remote LUKS unlock working
* ZFS key auto-load and mount working via custom systemd unit
* `/archive` and encrypted datasets mounting automatically after boot
* encrypted recovery bundle created and copied off-box

### Tailscale

* ACLs hardened using `tag:media`
* shared users restricted to Jellyfin and Seerr only
* sharing model better understood

## Lessons Learned

* remote unlock is one of the highest-value improvements for encrypted homelab systems
* mount failures should not be allowed to cascade into unrelated service failures
* a headless server should behave like a headless server, not like a desktop left half-enabled
* early-boot recovery and post-boot service dependency behavior are both critical
* Tailscale sharing and ACLs can be safely constrained without sacrificing expected media access
* recovery material should be backed up off-box as early as possible

## Follow-Up Items

* purge GUI packages from the NAS once fully comfortable headless
* consider adding a UPS and planning for power-flap scenarios
* optionally create LUKS header backups using `cryptsetup luksHeaderBackup`
* optionally refine Windows unlock scripts with better logging and profiles
* optionally finalize power-button behavior on the main server using logind settings

## Commands Worth Preserving

### Preboot Unlock from Windows

```powershell
ssh -p 2222 root@192.168.8.2
cryptroot-unlock
```

### Dropbear Initramfs Setup

```bash
sudo apt install -y dropbear-initramfs
sudo mkdir -p /etc/dropbear/initramfs
sudo nano /etc/dropbear/initramfs/authorized_keys
echo 'DROPBEAR_OPTIONS="-p 2222"' | sudo tee /etc/dropbear/initramfs/dropbear.conf
sudo sed -i 's/^#\?IP=.*/IP=dhcp/' /etc/initramfs-tools/initramfs.conf
sudo update-initramfs -u
```

### Resilient CIFS Automount

```fstab
//192.168.8.2/archive  /mnt/nas-archive  cifs  credentials=/root/.nas-archive,uid=1000,gid=1000,dir_mode=0775,file_mode=0664,iocharset=utf8,rw,vers=3.0,_netdev,nofail,x-systemd.automount,x-systemd.idle-timeout=60  0  0
```

### Disable Sleep Targets

```bash
sudo systemctl mask --now sleep.target suspend.target hibernate.target hybrid-sleep.target
```

### NAS ZFS Key Auto-Load Unit

`/etc/systemd/system/zfs-load-keys-and-mount.service`

```ini
[Unit]
Description=Load ZFS keys and mount ZFS datasets
After=zfs-import-cache.service zfs-import-scan.service local-fs.target cryptsetup.target
Wants=zfs-import-cache.service

[Service]
Type=oneshot
ExecStart=/bin/sh -lc '/usr/sbin/zfs load-key mediapool || true'
ExecStart=/usr/sbin/zfs mount -a

[Install]
WantedBy=multi-user.target
```

### Cockpit

* URL: `https://192.168.7.5:9090`

```bash
sudo apt install -y cockpit cockpit-machines
sudo systemctl enable --now cockpit.socket
```

### Key Backup Bundle

```bash
sudo tar -czf nas-key-material_$(date +%F).tar.gz \
  /etc/zfs/keys /etc/crypttab /etc/fstab /etc/dropbear/initramfs/authorized_keys
gpg --symmetric --cipher-algo AES256 nas-key-material_$(date +%F).tar.gz
```

## Status

Completed, with some optional cleanup and resilience improvements still remaining.

````
