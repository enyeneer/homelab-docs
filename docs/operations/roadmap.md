# Roadmap

This page tracks the major next steps for the homelab, from immediate cleanup items to longer-term infrastructure improvements.

## Near Term

### SG Preboot / Remote Unlock Reliability
Finish stabilizing and proving the remote unlock workflow for `seranogenomics` during initramfs/preboot.

**Goals:**
- make preboot networking more predictable
- fully validate remote unlock after reboot and after outage
- reduce or eliminate manual recovery steps
- document a clean recovery procedure

### NAS Power-Loss / Boot Reliability
Continue investigating why `nas` does not always recover cleanly after power loss.

**Goals:**
- narrow down BIOS / CMOS / hardware persistence issues
- reduce dependence on PSU drain/manual intervention
- improve confidence in unattended outage recovery
- document the final behavior clearly once understood

### Legacy DHCP / DNS Cleanup
Keep removing stale `192.168.7.x` and `192.168.8.x` artifacts that still show up in DHCP, DNS, and client behavior.

**Goals:**
- confirm old mappings and host overrides are gone
- verify old aliases stay removed
- reduce troubleshooting confusion
- keep the 5.x LAN layout clean and consistent

### Topology Diagram
Create a clean, sanitized topology diagram for the repo.

**Goals:**
- show core network flow clearly
- include router, switch, main server, NAS, VPN, and DNS layers
- improve readability for both personal docs and portfolio use

### Documentation Cleanup
Keep turning rough operational history into clearer public-facing documentation.

**Goals:**
- tighten the highest-value writeups
- keep README and index pages current
- make service, incident, and deployment docs easier to navigate
- ensure public docs stay accurate without exposing sensitive details

## Medium Term

### OpenVPN to WireGuard Migration
Evaluate and eventually migrate remote access from OpenVPN to WireGuard.

**Goals:**
- simplify remote-access configuration
- improve performance and ease of use
- preserve reliable LAN access for remote clients
- document migration steps and tradeoffs clearly

### Better Runbooks
Expand recovery and maintenance documentation into more repeatable procedures.

**Goals:**
- outage recovery steps
- remote unlock recovery procedure
- NAS recovery procedure
- VPN troubleshooting workflow
- service restart/recovery notes

### Standardized Maintenance Cadence
Keep core systems on a more consistent maintenance rhythm.

**Priority Systems:**
- OPNsense
- `seranogenomics`
- `nas`

**Goals:**
- reduce drift
- make maintenance more predictable
- improve rollback/recovery confidence
- validate critical services after updates

### Monitoring and Alerting Improvements
Build on the existing SMART, ZFS, mdraid, and UPS alerting foundation.

**Goals:**
- improve visibility into failures earlier
- make service and infrastructure issues easier to catch
- keep alerting practical and low-noise
- expand monitoring only where it meaningfully improves operations

## Longer Term

### NAS Hardware / Storage Refresh
Plan for eventual NAS hardware and storage upgrades.

**Goals:**
- reduce risk around aging hardware
- improve storage flexibility
- document the upgrade path before changes are made
- keep the storage layout clean and supportable

### Better Network Closet Layout
Improve the physical organization and airflow of the network/storage area.

**Goals:**
- cleaner cable and device layout
- better airflow and thermal behavior
- more stable long-term placement for UPS and network gear
- less friction during maintenance

### Additional Self-Hosted Services
Continue evaluating new services only where they are useful and low-risk.

**Current examples already added:**
- BOOX reading stack (`Komga`, `Suwayomi`, `Calibre-Web`)
- `Karakeep`

**Possible future additions:**
- Unpackerr
- Joplin
- Profilarr
- Shelfmark
- camera integration

**Goals:**
- add services intentionally, not just for novelty
- keep stacks understandable and maintainable
- document new deployments cleanly

### Media / Remote Access Experience
Continue refining the practical access model for family, remote users, and personal devices.

**Goals:**
- keep Tailscale/Jellyfin/Seerr access simple
- improve consistency across devices where worthwhile
- avoid risky or brittle UI hacks
- document the operating model clearly for future reference

## Guiding Principles

This homelab is being built with a few consistent priorities:

- prefer low-risk, high-ROI changes
- improve reliability before adding complexity
- keep the environment understandable
- document failures and recovery behavior honestly
- avoid exposing secrets or sensitive internal details in public docs
- use the lab as both a practical tool and a professional portfolio

## Current Overall Direction

The current direction is to:

- stabilize recovery behavior
- clean up remaining legacy networking artifacts
- keep remote access intentional and well-documented
- make operations more repeatable
- continue turning real troubleshooting history into strong public documentation

## Status

This roadmap is a living document and should be updated as priorities change.
