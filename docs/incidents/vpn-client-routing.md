# VPN Client Routing and Internet Access

This page documents a routing/firewall issue affecting remote VPN clients, the behavior that was observed, and the current direction for cleanup.

## Summary

Remote VPN clients were able to connect to the homelab and reach LAN resources, but internet access did not initially work as expected. The issue required additional routing/firewall cleanup to restore proper behavior.

## Environment

- VPN solution: OpenVPN
- VPN subnet: `192.168.6.0/24`
- LAN subnet: `192.168.5.0/24`
- Router/firewall: OPNsense (`texhnolyze.home.arpa`)
- DNS path: clients -> AdGuard Home on `192.168.5.1:53` -> upstream
- Main goal: allow remote clients to reach LAN resources reliably and, when intended, access internet through the VPN

## Original Symptom

The VPN client could successfully connect and access LAN resources, but internet connectivity from the VPN client was not working correctly.

In other words:
- LAN access: working
- remote client internet access through VPN: not working correctly

## Why This Mattered

This issue mattered for a few reasons:

- remote access was only partially functional
- behavior was confusing and inconsistent from the client side
- it made the VPN setup harder to trust as a general remote-access solution
- it highlighted the need for cleaner documentation of firewall/routing behavior

## Likely Cause

The problem was tied to how traffic from the VPN subnet was being routed and allowed through firewall/NAT policy.

The core lesson was that VPN clients should not be treated as if they are automatically equivalent to ordinary LAN clients. Even though they can access LAN resources, they are still arriving from a separate subnet and may require explicit handling depending on the desired behavior.

## Current State

The issue has been improved through configuration changes, and VPN clients can now reach the internet. However, the setup still needs cleaner documentation and validation so the final behavior is easy to understand and maintain.

## Lessons Learned

- VPN-connected clients are not the same thing as native LAN clients
- separate subnets often need explicit routing/firewall consideration
- “it connects” does not necessarily mean the VPN is fully working as intended
- documenting the intended traffic flow is just as important as making it work

## Follow-up Work

- document the final working firewall/routing behavior
- confirm expected DNS behavior for VPN clients
- verify client behavior on multiple devices
- simplify the setup where possible before a future migration
- use this writeup as reference material for eventual OpenVPN-to-WireGuard migration

## Status

In progress
