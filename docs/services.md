# Services

This page documents the major services in the homelab, what they do, where they live, and any important operational notes.

## Networking

### OPNsense
**Role:** Primary router/firewall for the homelab  
**Host:** `texhnolyze.home.arpa`  
**Notes:**
- Handles LAN routing and firewall policy
- Provides VPN access into the network
- Core network control point for connectivity and troubleshooting

### OpenVPN
**Role:** Remote access VPN  
**Subnet:** `192.168.6.0/24`  
**Notes:**
- Used for remote access to LAN resources
- Current VPN solution, with WireGuard as a likely future migration target
- One ongoing focus area is documenting and validating client routing behavior

### AdGuard Home
**Role:** DNS filtering for client devices  
**Host/Path:** `192.168.5.1:53`  
**Notes:**
- Primary DNS entry point for clients
- Used for filtering and DNS control
- Forwards upstream through the configured resolver chain

### Unbound
**Role:** Recursive/upstream DNS resolver  
**Port:** `5353`  
**Notes:**
- Sits behind AdGuard Home in the DNS path
- Important dependency for name resolution behavior across the lab

## Infrastructure

### Main Server
**Hostname:** `seranogenomics`  
**IP:** `192.168.5.7`  
**Role:** Primary Linux host for self-hosted services  
**Notes:**
- Main general-purpose server in the environment
- Runs Dockerized services and supporting workloads
- Key operational focus includes service reliability and recovery behavior

### NAS
**Hostname:** `nas`  
**IP:** `192.168.5.8`  
**Role:** Primary storage system  
**Notes:**
- Stores archive/media/infrastructure-related data
- Current known concern: power-loss / boot reliability behavior
- Important part of recovery planning and outage testing

### Home Assistant VM
**IP:** `192.168.5.14`  
**Role:** Home automation platform  
**Notes:**
- Separate VM in the environment
- Part of the broader self-hosting stack

### Gaming PC
**Hostname:** `ZACH-PC`  
**IP:** `192.168.5.15`  
**Role:** Client endpoint / workstation  
**Notes:**
- Useful as a known LAN client for connectivity testing and validation

### UPS / NUT
**Role:** Power event monitoring and controlled shutdown handling  
**Notes:**
- Used to improve shutdown behavior during outages
- Successfully validated for shutdown flow
- Recovery/startup behavior after outage is still being improved

## Applications

### Docker / Container Stack
**Role:** Application hosting layer  
**Notes:**
- Used to run self-hosted services on the main server
- Central part of the current homelab architecture

### Jellyfin / Media Stack
**Role:** Self-hosted media services  
**Notes:**
- Part of the Docker-based application stack
- Includes tooling around requests, acquisition, subtitles, search, and update management
- One of the most visible user-facing service groups in the lab

### Reading Stack
**Role:** Self-hosted manga/comics/books platform  
**Components:**
- `Komga`
- `Suwayomi`
- `Calibre-Web`

**Notes:**
- Runs as a standalone Docker Compose stack under `/opt/reading`
- Uses NAS-backed storage
- Supports BOOX workflow through Mihon and KOReader

### Karakeep
**Role:** Self-hosted bookmark manager  
**Notes:**
- Runs as a standalone Docker Compose stack on `seranogenomics`
- Intended to replace large browser bookmark/folder sprawl
- Later extended with OpenAI-based auto-tagging

### Home Assistant
**Role:** Home automation  
**Notes:**
- Included as a core self-hosted application
- Runs separately from the media stack
- Important for demonstrating service diversity in the homelab

## Current Operational Priorities

- improve VPN routing and client internet behavior documentation
- clean up stale DHCP/DNS legacy mappings
- improve preboot/dropbear remote unlock networking
- continue NAS power-loss / boot behavior investigation
- document service dependencies and recovery procedures more clearly

## Future Improvements

Each service entry will eventually include:
- backup and restore notes
- dependency mapping
- failure points
- maintenance/update workflow
- monitoring/alerting notes where applicable
