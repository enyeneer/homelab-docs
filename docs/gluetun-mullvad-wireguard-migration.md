# Gluetun / Mullvad Migration: OpenVPN to WireGuard

This page documents a service-impacting VPN issue in the YAMS stack and the steps taken to restore VPN-protected torrent traffic by migrating `gluetun` from Mullvad OpenVPN to Mullvad WireGuard.

## Summary

`gluetun` became unhealthy and Mullvad OpenVPN connectivity stopped working, which broke VPN-backed traffic flow for qBittorrent in the YAMS stack.

The issue was resolved by migrating the `gluetun` container from an OpenVPN-based Mullvad configuration to a WireGuard-based Mullvad configuration.

## Environment

- Host: `seranogenomics`
- Stack location: `/opt/yams`
- Platform: Docker Compose + Portainer
- VPN container: `gluetun`
- Torrent client: qBittorrent
- VPN provider: Mullvad

## Goal

Restore working VPN connectivity for `gluetun` and confirm that qBittorrent traffic is routed through Mullvad while the host system continues using normal ISP routing.

## Starting State

At the start of the issue:

- the YAMS stack was running under `/opt/yams`
- `gluetun` was configured to use Mullvad OpenVPN
- qBittorrent was expected to route traffic through `gluetun`
- `gluetun` showed as **unhealthy** in Portainer
- VPN connectivity was failing
- `yams check-vpn` was intermittently unable to retrieve the qBittorrent VPN IP

## Symptoms

### Container Health
- `gluetun` reported as unhealthy
- healthcheck output included:
  - `500 ... healthcheck did not run yet`

### OpenVPN Failure Pattern
Observed failures included:

- TLS handshake failures
- TLS key negotiation failures
- refused UDP connection attempts

Representative error pattern:

```text
TLS Error: TLS key negotiation failed to occur within 60 seconds
TLS Error: TLS handshake failed
read UDPv4 [ECONNREFUSED]: Connection refused
