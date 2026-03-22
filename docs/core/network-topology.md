# Network Topology

## Overview

The homelab is built around an OPNsense firewall/router, a 2.5 GbE switching layer, a primary Linux server, and a NAS. Remote access is currently handled through OpenVPN.

## High-Level Layout

- Internet service: 2.5 Gbps
- ISP modem/router provides Wi-Fi
- OPNsense handles routing/firewall duties for the lab
- 2.5 GbE switch connects core wired devices
- LAN subnet: `192.168.5.0/24`
- OpenVPN subnet: `192.168.6.0/24`

## Key Hosts

- router/firewall: `texhnolyze.home.arpa`
- main server: `seranogenomics` (`192.168.5.7`)
- NAS: `nas` (`192.168.5.8`)
- Home Assistant VM: `192.168.5.14`
- gaming PC / `ZACH-PC`: `192.168.5.15`

## Core Network Flow

- client devices use AdGuard Home on `192.168.5.1:53` for DNS
- AdGuard forwards upstream through the configured resolver path
- Unbound operates on port `5353`
- VPN clients connect on `192.168.6.0/24` and access LAN resources through OPNsense

## Notes

This page will eventually include:
- a visual topology diagram
- VLAN/subnet details if the network becomes more segmented
- service dependency mapping
- recovery notes for critical infrastructure components
