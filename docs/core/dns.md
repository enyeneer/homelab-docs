# DNS

This page documents the current DNS design in the homelab, including the request path, key components, and operational notes.

## Overview

The homelab uses a layered DNS setup built around AdGuard Home and Unbound.

### Current DNS Path
- clients -> AdGuard Home on `192.168.5.1:53`
- AdGuard Home -> upstream resolver path
- Unbound runs on port `5353`

## Goals of the DNS Design

- provide DNS filtering for client devices
- keep local control over name resolution
- support reliable DNS behavior for LAN clients
- make DNS troubleshooting easier by documenting the chain clearly

## Components

### AdGuard Home
**Role:** Primary DNS entry point for clients  
**Address/Port:** `192.168.5.1:53`

**Notes:**
- client devices send DNS requests here first
- used for filtering and DNS policy control
- acts as the front end of the DNS design

### Unbound
**Role:** Recursive/upstream DNS resolver  
**Port:** `5353`

**Notes:**
- sits behind AdGuard Home in the resolver path
- important dependency for successful DNS resolution
- part of the current local DNS architecture

## Client Behavior

### LAN Clients
LAN clients are expected to use AdGuard Home as their primary DNS resolver.

### VPN Clients
VPN clients should be validated separately to confirm:
- they receive the intended DNS settings
- they can resolve both internet domains and expected internal resources
- their DNS behavior matches the intended VPN design

## Why This Matters

DNS is one of the most important shared services in the homelab. When it breaks, many other services can appear broken even if they are technically still online.

Documenting DNS clearly helps with:
- faster troubleshooting
- understanding service dependencies
- validating changes after updates or network work
- avoiding confusion when testing VPN and LAN connectivity

## Common Failure Points

- clients cannot reach `192.168.5.1`
- AdGuard Home is not listening as expected
- Unbound is unavailable or misconfigured
- upstream resolution is failing
- VPN clients are not using the expected DNS path

## DNS Troubleshooting Checklist

1. Confirm client has general network connectivity
2. Confirm client is using the expected DNS server
3. Confirm AdGuard Home is reachable on `192.168.5.1:53`
4. Confirm Unbound is available on `5353`
5. Test resolution from a known-good LAN client
6. Compare LAN client behavior with VPN client behavior if relevant
7. Check whether the failure affects all clients or only one segment/device

## Current Priorities

- keep DNS behavior clearly documented
- validate expected DNS behavior for VPN clients
- reduce confusion during troubleshooting by making the resolver path explicit

## Future Improvements

This page may eventually include:
- example queries/tests
- local hostname resolution notes
- failure examples and recovery steps
- diagrams showing DNS flow through the network
