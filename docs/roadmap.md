# Roadmap

This page tracks the major next steps for the homelab, from immediate cleanup items to longer-term infrastructure improvements.

## Near Term

### VPN Client Internet Access Cleanup
Finish documenting and validating the current VPN behavior so remote clients have predictable access to both LAN resources and internet connectivity.

**Goals:**
- confirm final routing/firewall behavior
- document the working configuration clearly
- reduce confusion before any future VPN migration

### Legacy DHCP/DNS Cleanup
Remove or verify the removal of stale legacy mappings and old network artifacts from previous subnet layouts.

**Goals:**
- confirm old `192.168.7.x` and `192.168.8.x` references are gone
- verify old aliases like `192.168.7.1` and `192.168.8.1` stay removed
- simplify ongoing troubleshooting

### Preboot / Dropbear Networking Cleanup
Improve the reliability of remote unlock behavior during initramfs/preboot.

**Goals:**
- make remote unlock networking more predictable
- reduce manual recovery steps
- document a clean recovery procedure

### Documentation Improvements
Continue turning operational notes into cleaner public-facing documentation.

**Goals:**
- add a topology diagram
- improve service dependency documentation
- split larger topics into dedicated writeups where useful

## Medium Term

### OpenVPN to WireGuard Migration
Evaluate and eventually migrate from OpenVPN to WireGuard for remote access.

**Goals:**
- simplify VPN configuration
- improve performance and ease of use
- preserve reliable LAN access for remote clients

### Standardized Maintenance Cadence
Establish a more consistent update/maintenance rhythm for core systems.

**Targets:**
- OPNsense
- NAS
- `seranogenomics`

**Goals:**
- reduce drift
- make maintenance more predictable
- improve rollback/recovery confidence

### Better Runbooks
Expand recovery and maintenance documentation into clearer repeatable procedures.

**Goals:**
- outage recovery steps
- NAS recovery procedure
- VPN troubleshooting workflow
- service restart/recovery notes

## Longer Term

### Better Monitoring and Alerting
Improve visibility into system health, failures, and service status.

**Goals:**
- identify failures earlier
- make troubleshooting faster
- improve confidence in unattended operation

### NAS Hardware / Storage Expansion
Plan for future NAS hardware refreshes and additional storage capacity as needed.

**Goals:**
- improve long-term storage flexibility
- reduce risk around aging hardware
- document upgrade path before changes are made

### Camera Integration
Add and document the Hikvision camera setup as part of the broader homelab/home infrastructure.

**Goals:**
- get the device online reliably
- document network/discovery/setup steps
- fold it into the overall service inventory

### Additional Self-Hosted Services
Evaluate and document additional services as the lab grows.

**Possible future additions:**
- Unpackerr
- Joplin
- Profilarr
- Shelfmark

## Guiding Principles

This homelab is being built with a few priorities in mind:

- prefer low-risk, high-ROI changes
- improve reliability before adding complexity
- document changes clearly
- avoid exposing sensitive information in public documentation
- use the lab as both a practical tool and a professional portfolio

## Status

This roadmap is a living document and will be updated as priorities change.
