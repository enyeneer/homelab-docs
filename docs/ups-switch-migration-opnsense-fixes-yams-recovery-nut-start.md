# UPS Arrival, Switch Migration, OPNsense Fixes, YAMS Recovery, and NUT Start

This page documents a major homelab transition phase focused on introducing a new unmanaged switch, restoring connectivity after subnet and gateway issues, recovering the YAMS stack, and beginning UPS/NUT integration.

## Summary

The goal of this phase was to integrate a newly arrived UPS, move the environment onto a new unmanaged switch, restore broken connectivity to the main server and NAS after the network changes, and begin NUT-based graceful shutdown planning.

The work centered on:

- moving core devices onto a new unmanaged switch
- fixing OPNsense gateway and interface conflicts after consolidation
- restoring cross-subnet communication between the main LAN and server subnets
- recovering the YAMS stack after containers exited during the disruption
- physically placing and powering the UPS correctly
- starting NUT setup on the NAS for future coordinated shutdown behavior

## Main Goal

Get the new UPS integrated, migrate everything onto the new switch, restore SG/NAS and YAMS connectivity after the network changes, and begin the path toward graceful UPS-driven shutdown.

## Environment

### Router / Firewall
- Platform: OPNsense
- State at start: multiple separate LAN-style interfaces for different devices/subnets

### Switch
- Model: NETGEAR GS308
- Role: new unmanaged switch to become the central wired switching point

### Main Server
- Hostname: `seranogenomics`
- IP at the time: `192.168.7.5`
- Role: Docker/YAMS host for Jellyfin and related services

### NAS
- Hostname: `nas`
- IP at the time: `192.168.8.2`
- Role: primary storage system
- Notable detail: using a USB-to-Ethernet adapter

### UPS
- Model: CyberPower CP1500PFCLCD
- State at start: newly arrived, not yet integrated

## Starting State

At the beginning of this phase:

- OPNsense still had a multi-interface “separate LANs per device” layout
- the new GS308 switch had not yet become the main switching point
- the main server was still on `192.168.7.5`
- the NAS was still on `192.168.8.2`
- the UPS had arrived but was not yet fully integrated
- NUT had not yet been configured

## Problems Addressed

This work touched several related problems:

- SG and NAS became unreachable after being moved to the switch
- the consolidated LAN did not initially provide working gateway presence for the 7.x and 8.x networks
- OPNsense had old interface assignments that conflicted with the new layout
- firewall policy was blocking communication between `192.168.5.x` and the 7.x/8.x subnets
- YAMS services became unreachable because multiple containers had exited
- UPS placement and outlet selection needed to be decided safely
- NUT needed to be started on the NAS

## Changes Made

## 1. Moved Core Devices onto the GS308

The new unmanaged switch became the central switching layer, with OPNsense LAN uplinked into the switch.

### Result
This established the intended physical topology, but immediately exposed logical gateway and routing issues for SG and NAS.

## 2. Added OPNsense Virtual IPs on LAN

To preserve gateway reachability for the existing server subnets after consolidation, OPNsense LAN was given alias IPs for those networks.

### Added VIPs on LAN
- `192.168.7.1/24`
- `192.168.8.1/24`

### Result
The 7.x and 8.x devices once again had gateway presence on the consolidated LAN.

## 3. Disabled Old OPNsense Interfaces

Old interface assignments created conflicts and confusion after the switch migration.

### Changes
Disabled:
- `LAN_MainServer` (`em0`)
- dead or missing `LAN_NAS`

### Result
This reduced overlapping gateway/interface behavior and removed stale interface conflicts.

## 4. Fixed Firewall Policy for Cross-Subnet Access

Cross-subnet traffic still failed until LAN firewall policy was corrected.

### Changes
Created alias:

- `SERVERS_NETS`
  - `192.168.7.0/24`
  - `192.168.8.0/24`

Added allow rules:

- `LAN net -> SERVERS_NETS`
- `SERVERS_NETS -> LAN net`

Also removed or disabled the overly broad deny behavior that was blocking access.

### Result
Clients on `192.168.5.x` could again reach SG and NAS on the 7.x and 8.x subnets.

## 5. Recovered YAMS After Container Failures

After the network disruption, YAMS services were unreachable because multiple containers had exited.

### Observed State
- several containers were `Exited (128)`
- Jellyfin was `Exited (137)`

### Changes
Stopped Watchtower from restarting automatically:

```bash
docker stop watchtower
docker update --restart=no watchtower
````

Then restarted the full stack cleanly:

```bash
yams restart
```

### Result

YAMS services came back online and expected ports were listening again.

## 6. UPS Placement and Outlet Decisions

The UPS needed to be installed in a way that was both safe and practical.

### Decisions

* UPS should remain upright because of side venting
* critical devices should use **battery + surge** outlets
* temporary placement on a cutting board was acceptable for the moment
* longer-term plan was a better cart or shelf arrangement

### Result

The UPS was physically deployed in a workable initial state.

## 7. Began NUT Setup on NAS

The UPS USB connection was placed on the NAS so the NAS could become the primary NUT controller.

### Changes

Confirmed UPS detection via:

```bash
lsusb | egrep -i 'cyber|power|ups'
```

Installed:

```bash
sudo apt-get install -y nut nut-client nut-server
```

### Result

UPS hardware was confirmed visible to the NAS, and the software foundation for NUT was installed.

## What Worked

The following improvements were successful:

* the GS308 became the central switch
* OPNsense VIPs restored gateway presence for the 7.x and 8.x networks
* disabling stale interfaces removed interface conflicts
* firewall alias and allow rules restored 5.x to 7.x/8.x communication
* PC clients on `192.168.5.x` could again ping and SSH to both SG and NAS
* `yams restart` restored the media stack and related services
* Watchtower was stopped from interfering further
* the UPS was detected by the NAS over USB
* the NAS mount on SG was confirmed active

## What Broke or Regressed

A few failures showed up during the transition:

### Loss of Reachability After Switch Migration

SG and NAS were initially unreachable because:

* there was no effective gateway presence for the 7.x and 8.x networks on the consolidated LAN
* old OPNsense interface assignments were still conflicting
* firewall policy did not yet allow the needed cross-subnet access

### YAMS Service Outage

YAMS became unreachable during the disruption because multiple containers exited.

Likely contributing factors included:

* network disruption during the migration
* service restarts during gateway/interface changes
* possible Watchtower churn during an already unstable moment

No clear OOM evidence was found afterward.

## Validation

### Connectivity Validation from LAN Client

```powershell
ping 192.168.7.5
ping 192.168.8.2
Test-NetConnection 192.168.7.5 -Port 22
Test-NetConnection 192.168.8.2 -Port 22
```

Successful results confirmed working reachability from the main LAN to both hosts.

### Service Validation on Main Server

```bash
sudo ss -ltnp | egrep ':(8096|8989|7878|6767|8686|5055|9696|9000)\b'
```

This verified that key YAMS services were listening again after recovery.

### Mount Validation

```bash
findmnt /mnt/nas-archive
ls -la /mnt/nas-archive | head
```

This confirmed the NAS-backed mount was active on the main server.

### UPS Detection Validation

```bash
lsusb | egrep -i 'cyber|power|ups'
```

The UPS appeared as a CyberPower device, confirming USB detection on the NAS.

## Ending State

By the end of this phase:

### Network

* the GS308 was the central switch
* OPNsense LAN had VIPs for `192.168.7.1/24` and `192.168.8.1/24`
* old OPNsense interfaces had been disabled
* firewall rules allowed traffic between `192.168.5.x` and the 7.x/8.x server networks

### Services

* `yams restart` restored the full stack
* key ports were listening again
* Jellyfin and the related service set were reachable
* Watchtower was stopped and set not to restart automatically

### UPS

* the UPS was powered on
* critical devices were placed on battery-backed outlets
* UPS USB was connected to the NAS

### NUT

* packages were installed on the NAS
* the UPS was detected
* full shutdown-chain configuration still remained to be completed

## Lessons Learned

* physical switch migration can expose hidden routing and firewall assumptions immediately
* preserving gateway presence is critical when temporarily keeping legacy subnets alive
* stale interface assignments in OPNsense can create confusing duplicate-gateway behavior
* cross-subnet firewall policy must be explicit when consolidating networks
* service outages during network migration can be amplified by background automation like Watchtower
* UPS integration is not just hardware placement; shutdown orchestration matters just as much

## Follow-Up Items

* finish NUT configuration with the NAS as `netserver`
* configure the main server as a NUT client
* implement the intended 300-second shutdown timer behavior
* perform a controlled power-loss test by unplugging the UPS from the wall
* clean up remaining unused OPNsense interfaces and old rules
* eventually renumber SG and NAS onto `192.168.5.x`
* later remove the 7.1/8.1 VIPs and the temporary cross-subnet policy
* investigate intermittent `gluetun` unhealthy behavior separately

## Commands Worth Preserving

### Main Server Network and Service Checks

```bash
ip -br a
ip r
sudo ss -ltnp | egrep ':(8096|8989|7878|6767|8686|5055|9696|9000)\b'
```

### YAMS Recovery

```bash
yams restart
```

### Disable Watchtower Restart Behavior

```bash
docker stop watchtower
docker update --restart=no watchtower
```

### NAS Mount Validation on Main Server

```bash
findmnt /mnt/nas-archive
ls -la /mnt/nas-archive | head
```

### Connectivity Tests from Windows

```powershell
ping 192.168.7.5
ping 192.168.8.2
Test-NetConnection 192.168.7.5 -Port 22
Test-NetConnection 192.168.8.2 -Port 22
```

### UPS Detection and NUT Install on NAS

```bash
lsusb | egrep -i 'cyber|power|ups'
sudo apt-get install -y nut nut-client nut-server
```

### Example UPS USB ID

```text
Bus 001 Device 002: ID 0764:0601 Cyber Power System, Inc. PR1500LCDRT2U UPS
```

## Status

Completed as a transition and recovery phase, with NUT configuration and final renumbering work still remaining.
