# 2.5GbE Upgrade, 5.x LAN Migration, and OpenVPN Fix

This page documents a major homelab networking phase focused on upgrading the core path to 2.5GbE, moving key systems off legacy 192.168.7.x and 192.168.8.x addressing onto the primary 192.168.5.x LAN, validating real throughput gains, and fixing OpenVPN full-tunnel internet access after the migration.

## Summary

The goal of this phase was to modernize the core network path where it mattered most, reduce dependence on stale legacy subnets, and keep the environment functional while several infrastructure changes were happening at once.

The work centered on:

- upgrading key devices to 2.5GbE
- moving `seranogenomics`, `nas`, and Home Assistant onto the `192.168.5.x` LAN
- reassigning OPNsense LAN to the new 2.5GbE interface
- fixing storage mount and YAMS recovery after the addressing changes
- cleaning up stale DHCP and DNS behavior from the old networks
- restoring OpenVPN internet-through-tunnel behavior for phone clients
- improving closet layout and airflow around the network gear

## Main Goal

Upgrade the core homelab networking path to 2.5GbE, consolidate key systems onto the `192.168.5.x` LAN, verify that the upgrade delivered real throughput improvements, and fix enough routing, VPN, DHCP, and service recovery behavior to keep the environment stable.

## Environment

### Router / Firewall
- Platform: OPNsense
- WAN upstream: Xfinity gateway on `10.0.0.8/24`
- State at start: still carrying legacy/stale 7.x and 8.x baggage from earlier transitional networking

### Main Server
- Hostname: `seranogenomics`
- Role: primary Linux host for Docker/YAMS
- State at start: still tied to older addressing assumptions and dependent on NAS share availability for clean service recovery

### NAS
- Hostname: `nas`
- Role: primary storage system
- State at start: still associated with old 8.x-era behavior, with BIOS/CMOS sensitivity and somewhat fragile boot/recovery behavior

### Home Assistant
- State at start: still on old `192.168.7.4`

### VPN
- OpenVPN subnet: `192.168.6.0/24`
- State at start: VPN clients could reach LAN resources, but full-tunnel internet access from phone was broken by the end of the migration work until additional NAT fixes were made

## Starting State

At the beginning of this phase:

- OPNsense was still running behind an Xfinity gateway on `10.0.0.8/24`
- legacy 7.x and 8.x addressing still existed in various places
- `seranogenomics` and `nas` were not yet cleanly normalized onto `192.168.5.x`
- OPNsense LAN was still bottlenecked at 1GbE
- preboot/dropbear behavior on the main server was already somewhat fragile
- YAMS and Jellyfin recovery after reboot could be inconsistent if the NAS share was not ready
- OpenVPN from phone could reach LAN but not the internet through the tunnel by the end of the migration until fixed

## Problems Addressed

This work touched several connected problems:

- lack of a 2.5GbE path for the core LAN
- need to renumber key systems off old 7.x and 8.x networks
- stale DHCP and DNS records continuing to pollute local name resolution
- broken YAMS recovery when the NAS share was not mounted
- Home Assistant still living on old addressing
- OpenVPN full-tunnel internet not working correctly
- ghost 7.1 and 8.1 alias behavior still appearing on OPNsense LAN
- physical closet layout and airflow needing cleanup

## Changes Made

## 1. Installed 2.5GbE Hardware on the Core Path

Added 2.5GbE capability where it mattered most.

### Changes
Installed 2.5GbE NICs in:

- `seranogenomics`
- `nas`
- OPNsense router

Installed a TRENDnet `TEG-S381` 2.5G switch for the upgraded local path.

### Result
The key devices were now physically capable of multi-gig local traffic instead of being held back by 1GbE LAN bottlenecks.

## 2. Reassigned OPNsense LAN to the New 2.5GbE NIC

Reworked interface usage on OPNsense so the upgraded LAN path was actually used.

### Changes
- kept WAN on the Intel X540 10G NIC: `ix1`
- moved LAN to the new 2.5G NIC: `re0`

### Result
The LAN side of OPNsense was now aligned with the upgraded 2.5GbE design.

## 3. Renumbered Key Hosts onto the 192.168.5.x LAN

Moved the most important systems off legacy addressing.

### Changes
Migrated:

- `seranogenomics` -> `192.168.5.7`
- `nas` -> `192.168.5.8`
- Home Assistant -> `192.168.5.14`

Gaming PC remained:

- `192.168.5.15`

### Result
The main server, NAS, and Home Assistant were all brought onto the intended primary LAN.

## 4. Updated CIFS Mounting and YAMS Recovery on the Main Server

The main server still depended on the NAS share path, so the mount target needed to follow the renumbering.

### Changes
Updated the SG CIFS mount from the old NAS path to:

- `//192.168.5.8/archive`

Validated the mount and used `yams restart` once the NAS share was available.

### Result
YAMS and Jellyfin recovered successfully once the share path matched the new NAS address and the mount was active.

## 5. Updated NUT for the New 5.x Layout

Because NAS and SG changed addresses, NUT also needed cleanup.

### Changes
Updated NUT so:

- NAS NUT server listened on `192.168.5.8`
- SG NUT client pointed to `ups@192.168.5.8`

### Result
UPS monitoring and shutdown communication stayed aligned with the new addressing plan.

## 6. Cleaned DHCP and Static Mapping Behavior on LAN

Began cleaning up DHCP and DNS fallout from the earlier mixed-subnet environment.

### Changes
- cleaned LAN DHCP so clients now receive only `192.168.5.1` as DNS
- added new OPNsense LAN static mappings for:
  - `seranogenomics`
  - `nas`
  - `homeassistant`

### Result
The intended LAN address plan became much clearer, even though stale legacy references still remained in some parts of the config and client devices.

## 7. Fixed OpenVPN Full-Tunnel Internet Access

VPN clients could still reach LAN resources, but internet-through-VPN was not working until NAT was corrected.

### Changes
- switched outbound NAT to **Hybrid**
- added a WAN outbound NAT rule for source `192.168.6.0/24`
- recreated the Xfinity UDP `1194` port forward to `10.0.0.8`
- restarted or revalidated OpenVPN service behavior

### Result
Phone clients could again use OpenVPN for both LAN access and internet access through the tunnel.

## 8. Removed Ghost Legacy LAN Aliases from OPNsense

Old 7.x and 8.x gateway behavior still lingered on the LAN interface even after cleanup attempts.

### Changes
Removed live aliases from `re0` using shell commands:

- `192.168.7.1`
- `192.168.8.1`

### Result
The active runtime ghost aliases were removed from LAN, though stale legacy config artifacts still needed more cleanup.

## 9. Improved Physical Closet Layout and Airflow

Networking changes were also used as an opportunity to improve physical organization.

### Changes
- cleaned closet layout
- physically separated gear for better airflow

### Result
The network area ended the phase in a cleaner and more supportable physical state.

## What Worked

The following improvements were successful:

- UPS/NUT shutdown testing worked, with SG and NAS shutting down after a little over five minutes on battery
- SG to NAS `iperf3` throughput reached about **2.35 Gbit/s**
- gaming PC internet speed reached about **2370 Mbps down / 2360 Mbps up**
- OPNsense LAN successfully moved onto the new 2.5GbE path
- `seranogenomics` and `nas` both moved successfully onto `192.168.5.x`
- Home Assistant was moved successfully to `192.168.5.14`
- YAMS recovered once the NAS share was mounted and restarted cleanly
- OpenVPN phone access regained both LAN access and internet-through-tunnel behavior
- the physical closet layout improved

## What Broke or Regressed

A few important problems surfaced during the work:

### NAS CMOS / Battery Behavior
After installing a new CMOS battery, the NAS failed to boot and only recovered after reinstalling the old battery.

### Main Server Preboot Fragility
SG preboot/dropbear remote unlock behavior became more fragile during NIC and IP migration.

### Ghost OPNsense Alias Behavior
OPNsense temporarily still had old subnet IPs `192.168.7.1` and `192.168.8.1` bound to LAN even after VIP cleanup.

### Stale DNS / DHCP Pollution
Name resolution for `nas` and other hosts remained polluted by stale legacy DHCP/static records.

### Windows Hosts File Pollution
The gaming PC had a stale Windows hosts entry forcing:

- `nas.home.arpa`
- `nas`

to `192.168.8.2`

### Home Assistant Static Config
Home Assistant did not pick up its new reservation automatically because it was still statically configured inside the guest.

### Legacy DHCP Cleanup Incomplete
Old 7.x and 8.x mappings still existed in config and were not fully removable through the normal GUI.

## Validation

### 2.5GbE Link Validation

Checked that the upgraded interfaces negotiated as intended.

```bash
# Verify SG 2.5GbE link
sudo ethtool enp3s0

# Verify NAS 2.5GbE link
sudo ethtool enp7s0
```

### Local Throughput Validation

Used `iperf3` between SG and NAS to verify the local path.

```bash
# On NAS:
iperf3 -s

# On SG:
iperf3 -c 192.168.5.8
```

Observed result:

```text
~2.35 Gbit/s
```

### Internet Throughput Validation

Gaming PC speed tests showed approximately:

```text
~2370 Mbps down / ~2360 Mbps up
```

This confirmed that the WAN/LAN path upgrade was working at the expected level.

### CIFS Mount and YAMS Recovery Validation

```bash
sudo sed -i 's#//192\.168\.8\.2/archive#//192.168.5.8/archive#' /etc/fstab
sudo umount /mnt/nas-archive 2>/dev/null || true
sudo mount -a
mount | grep nas-archive

yams restart
docker ps -a --format 'table {{.Names}}\t{{.Status}}\t{{.Ports}}'
```

This confirmed that the corrected NAS path allowed the share to mount and the YAMS stack to recover.

### NUT Validation

```bash
sudo sed -i 's/ups@192\.168\.8\.2/ups@192.168.5.8/' /etc/nut/upsmon.conf
sudo systemctl restart nut-monitor.service
upsc ups@192.168.5.8
```

Also verified NAS listen configuration:

```text
LISTEN 127.0.0.1 3493
LISTEN 192.168.5.8 3493
```

### OPNsense Interface and Alias Validation

```bash
ifconfig re0 -alias 192.168.7.1
ifconfig re0 -alias 192.168.8.1
ifconfig re0

ifconfig re0
ifconfig ix1
ifconfig ix0
```

These checks confirmed the intended interface truth and removal of ghost aliases.

### OpenVPN Diagnostics

```bash
sockstat -4 -l | grep 1194
tcpdump -ni ix1 udp port 1194
configctl openvpn restart
```

These checks helped verify that the service and port-forward path were functioning again.

### Home Assistant VM Checks

```bash
virsh list --all
virsh domiflist homeassistant
virsh reboot homeassistant
```

Also preserved the bridged NIC MAC:

```text
52:54:00:35:d5:2d
```

## Ending State

By the end of this phase:

### Router / Network
- OPNsense WAN remained on `ix1` (10G)
- OPNsense LAN was moved to `re0` (2.5G)
- ghost live 7.x and 8.x aliases were removed from LAN
- some stale legacy DHCP and DNS records still needed cleanup

### Main Server
- `seranogenomics` was running at `192.168.5.7`
- CIFS mount was updated to the new NAS address
- YAMS and Jellyfin were back up and functional

### NAS
- `nas` was running at `192.168.5.8`
- NUT server was updated for the new LAN layout
- BIOS / CMOS persistence remained an unresolved future issue

### Home Assistant
- Home Assistant was fixed on `192.168.5.14`

### VPN
- phone clients could again connect through OpenVPN
- VPN clients could access LAN resources
- VPN clients could again reach the internet through the tunnel after Hybrid NAT and WAN outbound NAT were added

### Performance
- SG to NAS local throughput was confirmed at about **2.35 Gbit/s**
- wired gaming PC internet throughput was confirmed at about **2.3 Gbit/s both directions**

## Lessons Learned

- renumbering hosts and upgrading interfaces at the same time is powerful but increases the number of failure modes
- stale DHCP, DNS, and host overrides can linger long after the “real” network has changed
- runtime interface truth on OPNsense does not always match what the GUI seems to imply
- a VPN that reaches LAN resources is not necessarily fully working unless internet-through-tunnel behavior is also validated
- service recovery often depends on storage readiness more than the application stack itself
- local throughput tests and real-world speed tests are both worth doing after a network upgrade
- hardware quirks like CMOS battery behavior can derail an otherwise clean infrastructure change

## Follow-Up Items

- clean up stale legacy DHCP mappings for old `192.168.7.x` and `192.168.8.x` records
- decide whether to add proper Unbound or AdGuard host overrides for key local hostnames
- verify ghost old aliases do not return after reboot
- investigate and clean up SG preboot/dropbear remote unlock networking on the new 5.x layout
- revisit NAS BIOS / CMOS persistence later with a separate battery test
- optionally add `noserverino` to the SG CIFS mount to silence the inode warning
- investigate whether the old `mailserver` VM still exists under a different name or was removed

## Commands Worth Preserving

### Verify 2.5GbE Link State

```bash
sudo ethtool enp3s0
sudo ethtool enp7s0
```

### SG to NAS Throughput Test

```bash
# On NAS:
iperf3 -s

# On SG:
iperf3 -c 192.168.5.8
```

### Fix SG CIFS Mount After NAS Move

```bash
sudo sed -i 's#//192\.168\.8\.2/archive#//192.168.5.8/archive#' /etc/fstab
sudo umount /mnt/nas-archive 2>/dev/null || true
sudo mount -a
mount | grep nas-archive
```

### Update NUT Client on SG

```bash
sudo sed -i 's/ups@192\.168\.8\.2/ups@192.168.5.8/' /etc/nut/upsmon.conf
sudo systemctl restart nut-monitor.service
upsc ups@192.168.5.8
```

### NAS NUT Listen Configuration

```text
LISTEN 127.0.0.1 3493
LISTEN 192.168.5.8 3493
```

### Remove Ghost Old Subnet Aliases from OPNsense LAN

```bash
ifconfig re0 -alias 192.168.7.1
ifconfig re0 -alias 192.168.8.1
ifconfig re0
```

### Check OPNsense Interface Truth

```bash
ifconfig re0
ifconfig ix1
ifconfig ix0
```

### OpenVPN Diagnostics

```bash
sockstat -4 -l | grep 1194
tcpdump -ni ix1 udp port 1194
configctl openvpn restart
```

### YAMS Recovery Pattern After Reboot

```bash
mount | grep nas-archive
ls /mnt/nas-archive
yams restart
docker ps -a --format 'table {{.Names}}\t{{.Status}}\t{{.Ports}}'
```

### Home Assistant VM Checks

```bash
virsh list --all
virsh domiflist homeassistant
virsh reboot homeassistant
```

### New Static Mappings Added

```text
seranogenomics -> 192.168.5.7
nas            -> 192.168.5.8
homeassistant  -> 192.168.5.14
```

### Important Observations

```text
- New CMOS battery caused NAS boot failure / CPU red LED
- Old battery restored normal boot
- Windows hosts file on gaming PC had stale entry:
  192.168.8.2 nas.home.arpa nas
- CIFS warning suggested optional 'noserverino' mount option
- VPN internet fix required WAN outbound NAT rule for source 192.168.6.0/24 in Hybrid NAT mode
```

## Status

Completed as a major networking and renumbering phase, with legacy cleanup and preboot reliability follow-up still remaining.
