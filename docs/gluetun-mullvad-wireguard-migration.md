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
```

## Investigation

The troubleshooting process focused on:

* checking container state and health output
* reviewing `gluetun` logs
* locating the active YAMS `.env` file
* determining whether the existing Mullvad OpenVPN configuration was still viable
* testing a migration from OpenVPN to WireGuard

## Changes Made

### 1. Updated `/opt/yams/.env`

Added WireGuard-specific values:

* `WIREGUARD_PRIVATE_KEY=...`
* `WIREGUARD_ADDRESSES=10.73.191.199/32`

Old OpenVPN credentials were later removed.

### 2. Updated the `gluetun` Compose Configuration

Changed:

* `VPN_TYPE=openvpn` -> `VPN_TYPE=wireguard`

Replaced OpenVPN credential variables with:

* `WIREGUARD_PRIVATE_KEY`
* `WIREGUARD_ADDRESSES`

### 3. Recreated the Container

Used Docker Compose to force recreation of `gluetun`, then restarted the YAMS stack so dependent services picked up the change.

## Validation

### Tunnel Validation from Inside `gluetun`

The Mullvad tunnel was verified from inside the container:

```bash
docker exec gluetun wget -qO- https://am.i.mullvad.net/connected
```

This returned a successful Mullvad connection response.

### Split-Routing Validation

`yams check-vpn` confirmed the intended behavior:

* host IP remained on the ISP connection
* qBittorrent IP showed the Mullvad VPN IP
* qBittorrent traffic was successfully masked behind the VPN

Result:

```text
Your IPs are different. qBittorrent is masking your IP!
```

## What Worked

* migrating `gluetun` to Mullvad WireGuard restored VPN connectivity
* qBittorrent traffic was successfully routed through the Mullvad tunnel
* host traffic remained on normal ISP routing
* the YAMS stack returned to the intended split-routing behavior

## What Did Not Work

A brief attempt was made to force a specific exit region using:

* `SERVER_COUNTRIES=United States`
* `SERVER_CITIES=New York`

This caused server selection and connectivity problems and broke VPN validation again.

Effects included:

* `gluetun` failing to select or connect to a working server
* `yams check-vpn` failing to retrieve the qBittorrent IP

This was resolved by removing or commenting out those location filters.

## Ending State

By the end of the work:

* `gluetun` was successfully running on Mullvad WireGuard
* qBittorrent traffic was confirmed to be routed through Mullvad
* the host continued to use its normal ISP connection
* functionality was stable
* exit location was not reliably pinned and could vary

## Lessons Learned

* a working Docker stack can still fail if an upstream VPN method stops functioning
* `gluetun` health state and container logs are the fastest place to confirm the failure mode
* WireGuard was a cleaner and more reliable path here than continuing to fight the broken OpenVPN setup
* location pinning can introduce extra failure points if the selector values do not match what the provider or container expects
* validating from both inside the VPN container and from the application side is important

## Follow-Up Items

* optionally re-implement location pinning using `SERVER_HOSTNAMES` from the Mullvad WireGuard configuration bundle instead of city/country filters
* optionally verify tighter permissions on `/opt/yams/.env` for secret hygiene
* port forwarding was intentionally deferred

## Commands Worth Preserving

```bash
# Inspect failing container state
docker ps -a --filter name=gluetun
docker logs --tail=200 gluetun
docker inspect gluetun --format '{{json .State.Health}}'

# Edit YAMS environment
ls -la /opt/yams
nano /opt/yams/.env

# Redeploy gluetun
cd /opt/yams
docker compose up -d --force-recreate gluetun
docker logs --tail=80 gluetun

# Validate Mullvad tunnel from inside gluetun
docker exec gluetun wget -qO- https://am.i.mullvad.net/connected

# Restart full stack and validate VPN masking
yams restart
yams check-vpn

# Cleanup orphaned containers
docker compose up -d --remove-orphans
```

## Status

Resolved, with optional cleanup and improvement items remaining.
