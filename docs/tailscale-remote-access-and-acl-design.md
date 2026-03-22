# Tailscale Remote Access and ACL Design

This page documents how Tailscale is used in the homelab for remote access, service exposure, and access control.

## Summary

Tailscale is used as the practical remote-access layer for selected homelab services, especially user-facing services hosted on `seranogenomics`.

The current design is intentionally narrow. Admin access remains broad, but member and shared-user access is restricted to media-related services only. In practice, Tailscale is primarily used to let trusted personal devices, invited members, and limited remote family users reach Jellyfin and Seerr without exposing those services directly to the public internet.

A key part of the design is cost and plan simplicity: instead of adding every remote media user as a full Tailscale user, `seranogenomics` is shared outward to selected remote users so the setup can stay within the free-tier user limits.

## Main Goal

Document:

- how Tailscale fits into the homelab
- which devices are currently on the tailnet
- which services are intended to be reachable over Tailscale
- how access is restricted
- what the intended operating model is for admin, member, and shared access
- how MagicDNS currently fits into the design
- why sharing `seranogenomics` is preferred over adding every remote user as a full tailnet user

## Environment

### Tailnet Role in the Homelab

Tailscale serves as the low-friction remote-access layer for the homelab, especially for user-facing services that should be reachable remotely without direct public exposure.

In practice, Tailscale is mainly used for:

- Jellyfin
- Seerr
- remote access from personal devices such as phone, BOOX, and TV clients
- limited family access to media services

### Primary Tailscale-Exposed Server

#### `seranogenomics`
**Role:** primary Tailscale-exposed homelab server  
**Current state:** connected  
**Tailscale IPv4:** `100.66.168.39`  
**Tag:** `tag:media`

This is the main service host behind the current Tailscale access-control design.

## Why `seranogenomics` Is Shared Out

The current setup uses Tailscale sharing for remote media users instead of adding each person as a full Tailscale user.

### Reason

The main reason is to stay within the free-tier user limit while still allowing remote family/media access.

### Practical Effect

Instead of making each remote family user a full tailnet member, `seranogenomics` is shared outward to selected external users so they can reach only the intended media services.

This keeps the design:

- cheaper
- simpler
- more limited in scope
- better aligned with the actual use case

## Current Access Control Model

The current access model is tag-based and intentionally narrow for non-admin users.

### Admin Access

`autogroup:admin` can access:

- all users and devices
- all ports and protocols

**Intent:** admin/owner can access anything.

### Member Self-Access

`autogroup:member` can access:

- `autogroup:self`
- all ports and protocols

**Intent:** members can access their own devices.

### Member Access to Media Services

`autogroup:member` can access:

- `tag:media`
- ports `8096` and `5055`

This clearly maps to:

- Jellyfin on `8096`
- Seerr on `5055`

**Intent:** members can only access Jellyfin and Seerr on the media server.

### Shared Access to Media Services

`autogroup:shared` can access:

- `tag:media`
- ports `8096` and `5055`

This means shared external users are also intentionally limited to the same two media-service ports rather than receiving broad access to the homelab.

**Intent:** shared external users can only access Jellyfin and Seerr on the media server.

### Tag Ownership

The current tag model shows:

- `tag:media`
- owned by `autogroup:admin`

This means admin users control assignment and usage of the media-service tag.

## Tailscale SSH

The current JSON policy also includes a Tailscale SSH section.

### SSH Rule

Members are allowed to SSH into their own devices in `check` mode as:

- `autogroup:nonroot`
- `root`

### Intended Scope

This applies only to:

- `autogroup:member` -> `autogroup:self`

So Tailscale SSH is not being used as a general-purpose admin path into the homelab server fleet. It is scoped to users accessing their own devices.

## Current Operating Model

The current Tailscale design appears to follow this operating model:

- `seranogenomics` acts as the primary remote-access service host
- Tailscale is used as the preferred remote path for selected user-facing services
- admin users retain full access
- ordinary member users are restricted to self-access plus approved media-service ports
- shared users are also restricted to approved media-service ports
- the shared-user model is mainly for remote family/media access
- sharing `seranogenomics` is used as a practical workaround so the setup can stay on the free plan
- access is intentionally narrower than a flat “everyone can reach everything” tailnet model

This is effectively a mixed model that supports both:

- full tailnet membership for trusted users/devices
- narrower shared access for remote family/media use cases

### MagicDNS

MagicDNS is enabled.

This allows tailnet devices to be addressed by name instead of only by Tailscale IP.

## What Worked

The current setup appears to be working well in several important ways:

- `seranogenomics` is online and reachable on Tailscale
- a tag-based ACL model is in place instead of broad unrestricted access
- members and shared users are both limited to the intended media-service ports
- admin access remains broad enough for management tasks
- multiple device types are successfully enrolled in the tailnet
- MagicDNS is enabled, making the naming model cleaner
- remote family/media access can be supported without exposing the whole homelab
- sharing the server outward is an effective workaround for staying within the free-tier user limits
- the tailnet supports practical real-world remote access for selected services

## What Was Confusing or Needed Cleanup

A few parts of the design are still worth documenting as caveats.

## Lessons Learned

- Tailscale is most useful when it is treated as an intentional access layer, not just “VPN on everything”
- tag-based ACLs are much cleaner than broad unrestricted member access
- restricting member and shared access to only the required service ports makes the design easier to reason about
- media-service access is a good candidate for separate access policy from full admin access
- shared family/media access can be handled cleanly without granting broad homelab visibility
- sharing a server outward can be a practical way to stay within free-tier limits when the use case is narrow
- MagicDNS makes the naming model cleaner without requiring extra public exposure
- not every core infrastructure host needs to be a Tailscale node if the real use case is service access through one primary host
- reconstructing from the current working state is often easier than trying to remember setup history from scratch

## Follow-Up Items

- document the share recipients more explicitly if that becomes useful
- decide whether Tailscale HTTPS certificates are worth enabling later for any user-facing services

## Commands Worth Preserving

### Current State on `seranogenomics`

```bash
tailscale status
tailscale ip -4
tailscale ip -6
tailscale version
tailscale debug prefs
sudo systemctl status tailscaled --no-pager
````

## Status

In progress, but already accurate enough to document the current Tailscale design, DNS state, ACL model, SSH scope, and practical usage pattern.
