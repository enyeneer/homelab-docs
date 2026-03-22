# Maintenance and Updates

This page documents the general maintenance approach for the homelab, including update priorities, validation steps, and operational notes.

## Purpose

The goal of maintenance is to keep core infrastructure reliable without making unnecessary or risky changes.

This homelab is maintained with a bias toward:
- low-risk, high-ROI improvements
- clear rollback/recovery awareness
- validating critical services after changes
- avoiding unnecessary churn

## Core Systems

The highest-priority systems for maintenance are:

- OPNsense
- `seranogenomics`
- `nas`

These systems have the biggest impact on network access, storage, service availability, and recovery behavior.

## General Maintenance Approach

### 1. Check for Available Updates
Review available updates for core infrastructure and identify whether the change is:
- security-related
- bug-fix related
- feature-related
- optional / low priority

### 2. Prefer Low-Risk Maintenance Windows
Apply updates during a low-risk period when:
- downtime is acceptable
- recovery can be monitored
- there is time to validate services after reboot if needed

### 3. Validate Core Functionality After Changes
After updates, confirm that critical systems and services still work as expected.

## Post-Update Validation Checklist

### Network
- OPNsense is reachable
- LAN clients have network access
- internet connectivity is working
- VPN access behaves as expected

### DNS
- clients can resolve normal internet domains
- clients can resolve expected internal resources
- AdGuard Home is working
- Unbound is working

### Core Hosts
- `seranogenomics` is online
- `nas` is online
- Home Assistant is reachable
- key Dockerized services are reachable

### Reliability
- systems that rebooted returned cleanly
- no manual intervention was unexpectedly required
- no known recurring failure points were triggered

## Known Risk Areas

### Main Server Remote Unlock
The main server may stall during initramfs/dropbear remote unlock, so reboot-related maintenance should take this risk into account.

### NAS Post-Outage / Boot Behavior
The NAS has shown unreliable behavior after power-loss scenarios, so any maintenance involving power cycling or restart should be watched carefully.

### VPN / Routing Behavior
VPN behavior should be validated after network-related changes so routing, firewall, and internet access remain predictable.

## Maintenance Cadence

A regular monthly review/update cadence is the long-term goal for:
- OPNsense
- `nas`
- `seranogenomics`

The purpose is to:
- reduce drift
- avoid large batches of deferred updates
- keep recovery confidence higher
- make troubleshooting easier by reducing unknowns

## What to Record After Maintenance

After meaningful changes, record:
- what was updated
- whether a reboot was required
- whether recovery was clean
- whether any services needed manual intervention
- any new issues or odd behavior

## Future Improvements

This page may eventually include:
- a more detailed monthly checklist
- rollback notes for major systems
- update order recommendations
- links to service-specific maintenance procedures
