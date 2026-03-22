# Homelab Documentation

This repository documents my homelab architecture, services, troubleshooting, and operational notes. The lab is focused on practical networking, self-hosting, Linux administration, and reliability.

## Topology

> Topology diagram placeholder. A cleaned-up visual diagram will be added later.

<!-- Uncomment once you add the image -->
![Homelab topology](images/topology.png)

## Environment Summary

Current environment includes:
- OPNsense router/firewall
- Main server: `seranogenomics`
- NAS: `nas`
- LAN: `192.168.5.0/24`
- OpenVPN subnet: `192.168.6.0/24`
- 2.5 Gbps internet service
- 2.5 GbE switching
- Wi-Fi provided by the ISP modem/router
- AdGuard Home + Unbound for DNS
- Docker/YAMS media stack
- Home Assistant
- UPS/NUT shutdown testing and recovery notes

## Goals

- build practical NetOps, sysadmin, and self-hosting skills
- improve reliability and recovery workflows
- document troubleshooting and decisions clearly
- maintain a public portfolio of real homelab work

## Documentation

- [Network Topology](docs/network-topology.md)
- [Services](docs/services.md)
- [Incidents and Fixes](docs/incidents.md)
- [Roadmap](docs/roadmap.md)
- [Runbooks](docs/runbooks.md)

## Current Focus

- VPN client internet access and routing behavior
- cleanup of stale DHCP/DNS mappings
- preboot/dropbear remote unlock networking
- NAS boot and power-loss behavior investigation
- eventual OpenVPN to WireGuard migration

## Why This Repo Exists

This repo serves two purposes:
1. personal infrastructure documentation for operating and maintaining the lab
2. a public showcase of practical networking, systems administration, and troubleshooting work

## Notes

All public documentation is sanitized. Sensitive information such as credentials, private keys, tokens, and other secrets is excluded from this repository.
