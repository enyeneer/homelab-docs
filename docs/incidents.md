# Incidents and Fixes

This page tracks notable issues, outage tests, recovery findings, and troubleshooting writeups from the homelab.

## Incident Writeups

- [Gluetun / Mullvad Migration: OpenVPN to WireGuard](gluetun-mullvad-wireguard-migration.md)
- [Remote LUKS Unlock, Tailscale Hardening, and NAS/Server Reliability](remote-unlock-tailscale-hardening-and-reliability.md)
- [NAS Alerting, Storage Cleanup, and UPS Prep](nas-alerting-storage-cleanup-and-ups-prep.md)
- [UPS Arrival, Switch Migration, OPNsense Fixes, YAMS Recovery, and NUT Start](ups-switch-migration-opnsense-fixes-yams-recovery-nut-start.md)
- [2.5GbE Upgrade, 5.x LAN Migration, and OpenVPN Fix](2-5gbe-upgrade-5x-lan-migration-and-openvpn-fix.md)
- [YAMS, DNS, Media Stack Polish, and Watchtower Fix](yams-dns-media-stack-polish-and-watchtower-fix.md)
- [UPS / Power Outage Recovery Test](power-outage-test.md)

## Current Notable Issues

### VPN Client Internet Access / Routing
Remote VPN clients were able to reach LAN resources, but internet access behavior required additional routing/firewall work. This remains an important reference point for future VPN cleanup and the eventual move to WireGuard.

**Status:** In progress

### NAS Power-Loss Boot Behavior
The NAS has shown unreliable recovery behavior after power loss and may require full power drain/manual intervention before booting normally again.

**Status:** Open

### Preboot / Dropbear Remote Unlock Networking
The main server can stall during remote unlock in initramfs, and preboot networking has not always been reliably reachable.

**Status:** Open

## Why This Page Exists

This page is meant to capture:
- what happened
- what was learned
- what still needs to be fixed

Over time, major incidents should be broken out into their own dedicated writeups under `docs/`.
